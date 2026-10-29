# KeyballBLE（Keyball39 / ZMK）作業メモ

このリポジトリは、Shige が使っている Keyball39（左右分割・右トラックボール・Bluetooth）の ZMK 設定。
開発者ではないので、説明は平易な日本語で、変更前に「何がどう変わるか」を一言で伝えてから作業すること。

## 構成
- キーマップ: `config/KeyballBLE.keymap`（Keymap Editor でも編集される。編集後は Editor で開けることを壊さない）
- GitHub: Number867/KeyballBLE（Amasyake1917/KeyballBLE の fork）。gh は ~/.local/bin/gh、`gh repo set-default Number867/KeyballBLE` 済み
  （upstream リモートがあるので、-R を付けないと fork 元を見てしまうことがある）
- ビルド: `build.yaml` と GitHub Actions（workflow は build-user-config@v0.3）。ZMK は `config/west.yml` でコミット 9a8fdd6 に固定
- ボード名は `seeeduino_xiao_ble`（2026-10-29 にこの設定でビルド成功を確認。以前のメモの「xiao_ble / v0.2」は誤り）
- トラックボール設定: `config/boards/shields/Test/KeyballBLE_R.overlay`。automouse-layer = 7、scroll-layers = 8
  （キーマップのレイヤー番号を変えたら、ここも合わせること）
- push 後は Actions のビルド成功まで確認する。失敗したらログを読んで原因を説明する

## 物理配列（各レイヤー 43 箇所、bindings の並び順）
- 1段目: 左5 | 右5（Q W E R T | Y U I O P）
- 2段目: 左5 + 左内側（エンコーダー押し=左クリック） | 右内側 + 右5（A S D F G [LCLK] | [?] H J K L -）
- 3段目: 左5 + 左内側（右クリックボタン） | 右内側 + 右5（Z X C V B [RCLK] | [@] N M , . BSPC）
- 4段目: 左の修飾キー列4（GUI CTRL TAB SHIFT）+ 左親指2 | 右親指2 + 右端1（DEL）
- 左内側の LCLK / RCLK は変更しない。トラックボールはキーマップ外

## 現在のレイヤー方針（2026-10 改訂）
- 0 Base: QWERTY。右内側は ' と /（? は Shift+/、@ は記号レイヤー）。I 長押し = スクロール（レイヤー 8、旧配列から引き継ぎ）
- 1 Num（左親指 Space 長押し）: 左 F1–F12、右テンキー + 記号 / * - + = , .
  左 B 位置 = 今日の日付（マクロ &date_paste: ⌃⌥⌘F18 → 600ms 待つ → ⌘V）。
  Mac 標準の「ショートカット」アプリが ⌃⌥⌘F18 で日付を 20261029 形式でコピーする（両方の Mac に設定が必要）。
  キーボードは日付を知らないので PC 側の設定が必須。前のコピー内容が貼られる時は待ち時間を増やす。
  追加アプリ（Hammerspoon 等）は本人の希望で使わない。Windows は未設定
- 2 Char（右親指 Enter 長押し）: 記号。上段は US 配列の Shift+数字 と同じ並び
- 3 Nav（左親指内側 長押し）: 矢印は右手 I=↑ J K L=← ↓ →（指の位置を変えないこと）。
  左手: ホーム段 A S D F = Ctrl Opt Cmd Shift（押しっぱなしで矢印と組み合わせて選択）、
  下段 Z X C V B = 取消 切取 コピー 貼付 やり直し（Mac の Cmd）。
  上段 Q W E R T = 同アプリ内ウィンドウ切替(⌘`) 閉じる(⌘W) Mission Control(^↑) デスクトップ左/右(^←/^→)、G = アプリのウィンドウ一覧(^↓)。
  アプリ切替は Nav 中に D(Cmd) を押したまま修飾キー列の TAB を連打。
  右: 行頭/行末=Cmd+←/→、単語移動=Opt+←/→、右内側上=Spotlight(⌘Space)、
  スクショ: 右内側下=⌘⇧5（ツール/録画）、,=範囲→クリップボード(⌘⌃⇧4)、.=範囲→ファイル(⌘⇧4)
- System レイヤー W = 画面ロック(^⌘Q)
- 4 NavWin: Windows モード時だけ Nav の一部を Ctrl 系・Home/End に差し替える（conditional: 3 + 5 → 4）
  Q = Alt+Tab（直前のウィンドウへ）、G = Win+D（デスクトップ表示）、
  D = Alt（Mac と同じ「D を押したまま TAB」でアプリ切替になるように。Win キーのままだとタスクビューが開く）
- 5 WinMode: 中身は空のフラグ。System レイヤー左 A 位置の &tog 5 で ON/OFF。電源を切ると OFF（= Mac）に戻る
- 6 System: 左 Space 長押し + 右 Enter 長押しの同時押し（conditional layers）でのみ有効
- 原則: レイヤーキーを押す親指と反対の手で、そのレイヤーの主要キーを打てるようにする

## 守るルール
- BT_CLR / BT_CLR_ALL / bootloader を、Backspace・Delete・Enter など頻繁に押す位置と同じ場所に置かない
- bootloader は左右それぞれに1つずつ必要（片手ずつ書き込むため）
- INT_YEN など JIS 専用キーコードは使わない。OS 側のキーボード種類は US（ANSI）前提
- 入力切替は Ctrl+Space（Mac・Windows とも）。英数/かな専用キーは現状なし
- 文字キーにホールド動作を付けると入力遅延・誤爆が起きやすい。付ける場合は理由を説明してから

## 使用環境
- Mac（個人・副業）と Windows（仕事）の両方で使う。片方でしか動かないショートカットは、その旨を書く

## 要件（2026-10 ヒアリング）
- 主に Mac で使う（Windows は仕事用で次点）
- 効率を上げたい: コピペ・ショートカット操作、英語のメール/Slack、日本語の文章入力
- 慣れる期間があっても効率重視で変えてよい

## 今後の検討（未着手）
- 自動マウスレイヤー: 仕組みはすでに有効（レイヤー 7 Mouse、動かしてから 700ms）。中身が空なので今は何も変わらない。
  クリック/スクロールキーを置くかは本人と相談してから
- 定型文マクロ（メールアドレス・署名など）、プレーンテキスト貼り付け(⌥⇧⌘V)の追加
- IME は Ctrl+Space のトグルで確定（打ちながら切り替える使い方のため専用キーは不要と本人判断）
