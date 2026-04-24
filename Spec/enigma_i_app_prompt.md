# Gemini Canvas / Codex 向け作成依頼プロンプト

以下の仕様書に従って、Cloudflare Pages で静的配信できる 1 ファイル構成の HTML アプリを作成してください。

## 目的

歴史的な Enigma I 互換の暗号変換器を、ブラウザだけで動作する HTML アプリとして実装します。

対象はまず Enigma I の 3 ロータ構成です。

- ロータ：I, II, III, IV, V
- リフレクタ：UKW B, UKW C
- 入力盤：ETW = ABCDEFGHIJKLMNOPQRSTUVWXYZ
- プラグボード対応
- リング設定 Ringstellung 対応
- 初期表示位置 Grundstellung / rotor positions 対応
- Enigma のロータ進行、特に double stepping を正しく実装
- 暗号化と復号は同一処理で行う

## 実装条件

- 単一の `index.html` として完結させる
- 外部ライブラリを使わない
- Cloudflare Pages にそのまま配置して動くこと
- JavaScript / CSS / HTML を同一ファイル内に含める
- UI はシンプルでよいが、学習・展示用途として分かりやすくする
- 入力文字は A-Z のみ処理し、空白・改行・記号は表示上は保持する
- 内部処理では英字を大文字化する
- 変換ごとにロータ位置が進む
- 変換結果、最終ロータ位置、可能なら文字ごとの変換ログを表示する

## 重要な互換性要件

Enigma I の既知配線をそのまま使用してください。

ロータ配線：

- I:   EKMFLGDQVZNTOWYHXUSPAIBRCJ, notch Q
- II:  AJDKSIRUXBLHWTMCQGZNPYFVOE, notch E
- III: BDFHJLCPRTXVZNYEIWGAKMUSQO, notch V
- IV:  ESOVPZJAYQUIRHXLNFTGKDCMWB, notch J
- V:   VZBRGITYUPSDNHLXAWMJQOFECK, notch Z

リフレクタ配線：

- UKW B: YRUHQSLDPXNGOKMIEBFZCWVJAT
- UKW C: FVPJIAOYEDRZXWGCTKUQSBNMHL

ETW:

- ABCDEFGHIJKLMNOPQRSTUVWXYZ

## UI 要件

画面には以下を含めてください。

1. ロータ選択
   - 左ロータ、中ロータ、右ロータ
   - I〜Vから選択
   - 同じロータを重複選択しないようにする、または警告を出す

2. リフレクタ選択
   - UKW B
   - UKW C

3. リング設定
   - 左・中・右それぞれ A〜Z
   - 既定値 AAA

4. 初期ロータ位置
   - 左・中・右それぞれ A〜Z
   - 既定値 AAA

5. プラグボード設定
   - 例: `AV BS CG DL FU HZ IN KM OW RX`
   - 同一文字の重複や自己接続を検出してエラー表示
   - 空欄ならプラグなし

6. 入力テキスト
   - 複数行入力可

7. 実行ボタン
   - Encrypt / Decrypt
   - Reset positions
   - Clear

8. 出力
   - 変換結果
   - 最終ロータ位置
   - 文字ごとのログ表
     - 入力文字
     - 変換前ロータ位置
     - 出力文字
     - 変換後ロータ位置

## 実装上の注意

- Enigma はキーを押す前にロータが進む
- 右ロータは毎文字進む
- 中ロータは右ロータのノッチにより進む
- 左ロータは中ロータのノッチにより進む
- double stepping を正しく再現すること
- ロータ配列は、画面上は「左・中・右」だが、信号は右→中→左→リフレクタ→左→中→右の順に通る
- リング設定はロータ内部配線と表示位置の相対ずれとして処理する
- 変換処理は可逆であり、同じ設定・同じ初期位置で暗号文を再入力すると平文に戻る

## 受け入れ条件

- `index.html` だけでブラウザ動作する
- Cloudflare Pages で静的配信できる
- ロータI–V、UKW B/C、リング設定、初期位置、プラグボードが動作する
- 同一設定・同一初期位置で暗号化結果を再入力すると元に戻る
- ダブルステップが実装されている
- 設定エラー時に分かりやすく警告を出す
- コード内に主要関数のコメントを入れる

## 実装後に説明してほしいこと

実装後、以下を簡潔に説明してください。

- 追加したファイル
- 主要な関数
- Enigma の変換順序
- double stepping の実装方針
- 既知の制約
