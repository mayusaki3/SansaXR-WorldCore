# SansaXR-WorldCore 作業記録 (2026-10-23)

## 状態
- 対象: mayusaki3/SansaXR-WorldCore / develop。HLDocS v0.6.1-preview に準拠して今後ドキュメント化。
- 本書は設計検討の Worklog であり、正式な仕様・テスト仕様ではない。
- ProjectSansa の repository federation / local-first / offline-first を継承。
- SansaXR は Unity / Godot / O3DE で検証するXR実行側、WorldCore はエンジン非依存のRustサーバー・Rust参照クライアントを先行開発。
- SansaVRMはアバター／ワールドオブジェクト記述形式を検討。SansaSphere は identity、権利、監査、来歴、経済、収益配分査定、利用実績評価。責務境界とAPIは今後調整。

## 確認済みの方向性
- 地球複製（緯度経度を基準）の未定義箇所は霧、位置情報付き写真から3D再構築、追加写真による改善。Creator IFは分岐世界。離れた既知エリアは霧ワープゲートで接続する構想。
- セルは動的ストリーミングの検索・配信単位。重複セル、セル横断オブジェクト／ギミックを許す。オブジェクト所有権をセルに固定しない。
- ワールド作品は制作者のSandboxへアップロード・検証し、承認済み差分を稼働中のLive Worldへ公開。公開リビジョンとアセット、権限・必要なランタイム状態を永続化し、再起動後に復元する。
- 公開方式はC: 自領域内の安全な変更は自動公開、共有物・他者領域・高リスク変更は承認。承認は形式／安全性の検証失敗を上書きしない。競合時は再検証。
- Sandbox時に権利・利益配分査定、本番利用はWorldCoreからSansaSphereへ必要最小限の利用実績イベントを送り、Sandbox試験を収益算定に混ぜない。
- ワールド入場の公開/Private/招待/有料/グループ等の条件を組み合わせて制御。Discover（存在発見）、View（外観）、Enter（入場）、Interact（操作）、Asset Fetch（取得）を分ける。
- シームレスワールドのアクセス方式はC: **物理的な境界制御と別インスタンス方式を併用**。通常はドア・境界をサーバー権限で制御、非公開・有料・イベント等は必要に応じ隔離インスタンスに遷移する。入場先の権限を確認してから移送し、失敗時は元の位置/インスタンスに留める。
- アバターの登録・選択・所有者との関連はSansaSphere側。実体アセットは参照型の共通ストレージ候補。描画セッションへの反映はSansaXR。

## アクセス・インスタンス詳細設計（提案、未確定）
- `WorldDefinition`（作品／規則）、`WorldInstance`（稼働状態）、`AccessRegion`（地理空間と境界）、`Portal`（遷移）、`WorldSession`（ユーザー接続）、`AccessPolicy`（組み合わせ条件）、`EntitlementReference`（Sphere由来の利用権）、`WorldRevision`（変更不能な公開スナップショット）を分離。
- インスタンスタイプ例: Shared Live / Private Live / Event / Sandbox。複数インスタンスが同じWorldRevisionを参照できる。Sandboxの試験イベントは別の集計ドメイン。
- Access Policy: Discover/View/Enter/Interact/AssetFetch を個別判定。公開位置の外観データと権限保護対象の内部データを分離。存在秘匿では検索結果・事前配信・エラー応答からの漏洩を防ぐ。
- 境界判定はサーバー権威で衝突形状とユーザーの移動軌跡を評価する。レンダラーの壁表示だけに依存しない。進入拒否時は許可された境界外の最終位置を保持し、取得済みアセットの完全なコピー防止は保証しない。
- インスタンス遷移は `Request -> Authorize -> Reserve -> Preload -> Commit -> Acknowledge` を候補とする。Commit前は元インスタンスで継続、再送・タイムアウトに冪等対応。移送後の復帰地点や再接続方針は要詳細決定。
- オフライン時はポリシーに従う有効な権利キャッシュで既存操作を許容可能だが、新たな有料/Private入場は必要な権限検証が不能なら保留/拒否。失効反映と有効期限は別途協議。
- 使用実績イベントにセッション、世界/インスタンス、リビジョン、利用コンテンツ、匿名化可能な識別子、種別、重複排除IDを含め、必要最小限にする。高頻度の位置データ収集は原則避ける。
- 公開時は永続アセットの存在検証→変更検証/承認→DBトランザクションでリビジョン参照とOutboxを確定→ライブ配信。アセットが有料／Privateの場合は権限付きで配布する。

## 単独で着手可能な検証
1. Rustワークスペース、プロトコル／権限ポリシー／空間セル／参照クライアントを分割。
2. SansaSphereモックで entitlement / group / invite の判定と fail-closed をテスト。
3. WorldDefinition, WorldInstance, AccessRegion, Portal, WorldSession の純粋状態モデルと境界通過テスト。
4. 物理境界による拒否と、インスタンス遷移の失敗・重複・再接続の状態遷移テスト。
5. Sandbox→公開リビジョンの整合性と、公開通知・UsageEventの冪等性テスト。
6. Unity/Godot/O3DEの表示・XR・実機OS対応はSansaXR側で後続検証。

## リポジトリ間要調整
- SansaVRM: World Objectフォーマット、アセット依存性、validator のAPI。
- SansaSphere: 認証トークン/署名、権限／entitlement／招待の失効、査定、収益配分、イベント形式。
- SansaXR: エンジンアダプター、接続API、遷移演出、位置と物理同期、Android/iOS含む利用可能性。
- ProjectSansa: 横断用語、公開境界、共有契約。正式仕様へ昇格時は既存の規約に整合させる。

## 未決定
- 地理空間の座標参照系、垂直座標、セル分割方式、領域衝突判定の精度。
- WorldDefinitionとCreator IFの派生ルール、入場時の料金条件と再入場ルール。
- 世界間・インスタンス間遷移の権限失効時の退去／扱い。
- 収益配分の査定式、支払額、プライバシー規約（SansaSphereで扱う）。
- 永続DB・アセットストレージ選定、ギミック実行モデル、失敗時の補償。
