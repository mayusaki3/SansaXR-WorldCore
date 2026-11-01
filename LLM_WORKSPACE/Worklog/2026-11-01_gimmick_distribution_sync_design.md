目次 > SansaXR-WorldCore > LLM_WORKSPACE > Worklog > ギミック配布・同期 詳細設計案

# ギミック配布・同期 詳細設計案

- 対象: mayusaki3/SansaXR-WorldCore / develop
- 日付: 2026-11-01
- 文書状態: Worklog 設計案。正式仕様・API仕様・テスト仕様として未承認
- 文書規格: HLDocS v0.6.1-preview。正規仕様への昇格時に構造・ID・Traceabilityを整備する

## 1. 決定済みの方針
- 観客側Bは複数インスタンス。出演側Aも複数候補だが、同時に有効な中継元は1つ。
- 選択したBはAへ観客投影する双方向接続、それ以外のBはAの出演者投影を受信する配信方式。全Bのリアクションを集約してAへ返す。
- ギミック配布はC案: 基本パッケージの事前配布＋ライブ中の動的差分更新。
- Aが演出を制御し、Bは座席・観客・物理状態を保持したままローカルでギミックを実行する。
- Sandboxのテスト利用とLiveの利用実績を分離する。権利・収益配分はSansaSphereとの連携事項。

## 2. 論理コンポーネント（提案）
- EventCoordinator: イベントID、共通時刻、ActiveSourceId、SourceEpoch、演出世代切替。
- GimmickPackageRegistry: 不変パッケージのメタデータ、ハッシュ、依存アセット、署名・権限の検証窓口。
- GimmickDistributionManager: Bごとの不足パッケージ検出、差分配布、準備完了管理。
- LiveBridge: AからBへの出演者投影・ギミックイベント配送、選択BからAへの観客投影、全Bの反応集約。
- GimmickEndpoint(B): ローカルターゲット対応付け、コマンド検証、スケジュール実行、ACK・状態報告。
- StateReconciler: 再接続・世代切替時のスナップショットとイベント再同期。
- CapabilityProvider: 実行機能の抽象インターフェース。動画・音声配信やエンジン固有ギミック実装は外部。

## 3. データ契約（提案）
- GimmickPackage: PackageId、Version、ContentHash、Manifest、Dependencies、RequiredCapabilities、ExecutionPermissions、DefaultState。
- GimmickBinding: AbstractTargetId、BInstanceId、ConcreteTargetIds、BindingVersion、MissingTargetPolicy。
- GimmickState: EventId、BInstanceId、PackageGeneration、SourceEpoch、EventClockPosition、StateRevision、SnapshotHash。
- GimmickEvent: EventId、SourceEpoch、PackageGeneration、CommandId、ExecutionTime、TargetBinding、Operation、Parameters、RequiredCapability。
- DistributionSession: BInstanceId、TargetGeneration、ManifestHash、MissingArtifacts、Readiness、Deadline。
- ActivationPlan: TargetGeneration、ActivationTime、RequiredReadinessPolicy、FallbackPolicy、ActivationId。
- GimmickExecutionReceipt: CommandId、BInstanceId、Status、StateRevision、AppliedTime、ErrorCode。
- PackageGenerationはイベント内の有効化世代。ContentHashは内容同一性。SourceEpochはAの中継元切替世代。互いに別の概念とする。

## 4. 事前配布手順（提案）
1. Aがパッケージを登録し、ハッシュ・依存関係・権利・必要機能を検証する。
2. WorldCoreは対象Bの必要なパッケージを計算し、ハッシュ一致済みのアセットを再利用する。
3. Bは不足アセットを権限付きで取得し、署名／ハッシュ・依存関係・機能・ローカルバインディングを検証する。
4. Bは未有効化の準備領域へ配置し、Ready/Rejectedと理由を報告する。
5. Coordinatorが同期時刻と世代を指定して有効化する。
6. Bは有効化した世代と状態リビジョンをACKする。

## 5. 動的差分更新手順（提案）
1. Aが新しい不変パッケージ世代を提出する。既存世代は変更しない。
2. WorldCoreは差分と必要機能・権利・互換性を検証し、対象Bへ不足分のみ配布する。
3. Bは旧世代を稼働させたまま新世代を準備する。
4. CoordinatorがRequiredReadinessPolicyに従い、同期切替時刻を決定する。
5. 全Bの準備を待つか、未準備Bを旧世代に留めるか、対象から除外するかはイベントポリシーで決定する。既定値は未決定。
6. 有効化コマンドはActivationIdと世代番号により冪等化し、切替後の旧世代コマンドを適切に拒否する。
7. 失敗時は旧世代を保持し、ロールバック・再同期の判断を明示する。

## 6. 実行イベントと障害対策（提案）
- イベントは共通イベント時刻でスケジュールする。CommandIdで重複を排除する。
- Bは発行元の認可、SourceEpoch、PackageGeneration、TargetBinding、Capability、ExecutionTimeを検証する。
- 状態設定型（照明色等）と副作用型（生成、課金、発射等）を区別し、後者の重複実行を永続的な実行記録で防ぐ。
- 再接続時は検証済みスナップショットを適用し、必要なイベントのみ追跡する。単発演出の無条件リプレイは禁止。
- A1からA2への切替ではSourceEpochを増加させ、旧Aからの遅延メッセージを拒否する。
- 配布・演出状態はイベント単位、B単位で監査可能にする。高頻度の個人行動履歴は収集しない。

## 7. セキュリティと分離
- AがBの内部データや実行メモリを直接書き換えることは許可しない。
- 初期PoCでは宣言的なギミック定義と制限付きコマンドのみを配布する。
- 任意スクリプトの配布・実行は、実行サンドボックス、リソース上限、能力制限、署名、更新互換性が定義されるまで保留する。
- 有料/Privateアセットの配布は、入場権限とは別にAssetFetch権限を検証する。
- 配信Bと接続Bは同じA→Bギミック同期を利用できる。B→A観客投影は接続Bに限定し、その他は匿名・集約リアクションのみ。

## 8. WorldCore単独で可能な検証
- GSYNC-001: 同一ハッシュのパッケージを再配布しない。
- GSYNC-002: ハッシュ不一致／依存不足を検出し、有効化しない。
- GSYNC-003: 異なるB会場構造へ論理ターゲットを対応付ける。
- GSYNC-004: 再接続でスナップショットと不足イベントを復元し、副作用を重複実行しない。
- GSYNC-005: SourceEpochが古いAからのコマンドを拒否する。
- GSYNC-006: Capability未許可のギミックを拒否する。
- GSYNC-007: 接続Bと配信Bの演出時刻を揃え、Aには選択Bの観客投影のみ返す。
- GSYNC-008: 差分配布中も旧世代を稼働させる。
- GSYNC-009: 一部Bが未準備の場合、未決定のReadinessPolicyに従う動作を検証する。
- GSYNC-010: ActivationIdの再送で二重有効化しない。
- GSYNC-011: A切替とパッケージ切替が競合しても不正な世代混在を許さない。
- GSYNC-012: 有料/Private依存アセットを無権限Bに配布しない。
- 単体テストは番号付き、実装時はカバレッジ100%を目標とする。テスト仕様の確定前にコードを作らない。

## 9. 未決定・次の意思決定
- 動的差分更新のReadinessPolicy: 全B準備完了必須か、部分有効化を許すか。
- 切替時刻の遅延許容、遅延演出のスキップ/補正、ライブ時計の同期方式。
- ギミックパッケージ形式、サイズ上限、バージョン互換性、Bの機能不足時の代替演出。
- スクリプト実行の有無と隔離方式。
- Bが異なるパッケージ世代を持つ場合の相互作用ルール。

## 10. 他リポジトリとの調整
- SansaSphere: コンテンツ権利・アセット取得権限・利益配分・利用状況。
- SansaVRM: ギミック表現とアセット依存関係の記述形式。
- SansaXR: Unity/Godot/O3DEの実ギミックランタイムと同期表示。
- ProjectSansa: 横断契約、用語、来歴。
- WorldCoreは外部連携をモック化して状態遷移・配布計画・冪等性を単独検証する。

---
目次 > SansaXR-WorldCore > LLM_WORKSPACE > Worklog > ギミック配布・同期 詳細設計案
