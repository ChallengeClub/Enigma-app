# Enigma I 互換 HTML アプリ 実装仕様書

## 1. 目的

本仕様書は、Cloudflare Pages で静的配信可能な Enigma I 互換の暗号変換器 HTML アプリを実装するためのものです。

対象は、歴史的な軍用 Enigma I の 3 ロータ構成を基本とします。

最初の実装範囲は以下です。

- 3ロータ構成
- ロータ I, II, III, IV, V
- リフレクタ UKW B, UKW C
- ETW はストレート配線
- リング設定 Ringstellung
- 初期ロータ位置
- プラグボード Steckerbrett
- double stepping を含むロータ進行

M3/M4、ロータ VI–VIII、Beta/Gamma、薄型リフレクタは将来拡張対象とし、本仕様の初期版には含めません。

---

## 2. 配線仕様

### 2.1 アルファベット

```text
ABCDEFGHIJKLMNOPQRSTUVWXYZ
```

文字は A=0, B=1, ..., Z=25 として扱います。

### 2.2 ETW

```text
ETW = ABCDEFGHIJKLMNOPQRSTUVWXYZ
```

本アプリでは ETW を恒等写像として扱います。

### 2.3 ロータ配線

| Rotor | Wiring | Notch |
|---|---|---|
| I | EKMFLGDQVZNTOWYHXUSPAIBRCJ | Q |
| II | AJDKSIRUXBLHWTMCQGZNPYFVOE | E |
| III | BDFHJLCPRTXVZNYEIWGAKMUSQO | V |
| IV | ESOVPZJAYQUIRHXLNFTGKDCMWB | J |
| V | VZBRGITYUPSDNHLXAWMJQOFECK | Z |

### 2.4 リフレクタ配線

| Reflector | Wiring |
|---|---|
| UKW B | YRUHQSLDPXNGOKMIEBFZCWVJAT |
| UKW C | FVPJIAOYEDRZXWGCTKUQSBNMHL |

---

## 3. 用語

### 3.1 ロータ順

画面表示上は以下の順で指定します。

```text
Left rotor / Middle rotor / Right rotor
```

信号は以下の順で流れます。

```text
Keyboard
→ Plugboard
→ Right rotor forward
→ Middle rotor forward
→ Left rotor forward
→ Reflector
→ Left rotor reverse
→ Middle rotor reverse
→ Right rotor reverse
→ Plugboard
→ Lamp/output
```

### 3.2 Rotor position

ロータ窓に表示される現在位置です。

例：

```text
AAA
```

左・中・右の順で表示します。

### 3.3 Ringstellung

リング設定です。

既定値は AAA とします。

リング設定は、ロータの内部配線と外側の文字リングの相対位置をずらします。

実装上は、ロータ変換時に以下のように offset を扱います。

```text
offset = rotorPosition - ringSetting
```

---

## 4. ロータ進行仕様

### 4.1 基本

Enigma は文字を変換する前にロータが進みます。

- 右ロータは毎文字 1 ステップ進む
- 中ロータは右ロータのノッチにより進む
- 左ロータは中ロータのノッチにより進む
- 中ロータは double stepping により、連続して進む場合がある

### 4.2 推奨実装ロジック

文字変換前に、以下の順で進行判定します。

```javascript
const middleAtNotch = isAtNotch(middleRotor, middlePos);
const rightAtNotch = isAtNotch(rightRotor, rightPos);

if (middleAtNotch) {
  leftPos = step(leftPos);
  middlePos = step(middlePos);
}

if (rightAtNotch) {
  middlePos = step(middlePos);
}

rightPos = step(rightPos);
```

この処理により、右ロータは常に進み、中ロータが自身のノッチ位置にある場合に左ロータを進めつつ自分も進むため、double stepping を再現できます。

### 4.3 Notch の扱い

Notch は、キー押下前のロータ窓位置で判定します。

ロータごとのノッチ：

```text
I   Q
II  E
III V
IV  J
V   Z
```

---

## 5. 文字変換仕様

### 5.1 入力文字

- A-Z は変換対象
- a-z は大文字化して変換
- 空白、改行、句読点、数字、記号は変換せずそのまま出力
- 非変換文字ではロータを進めない

### 5.2 プラグボード

入力例：

```text
AV BS CG DL FU HZ IN KM OW RX
```

仕様：

- 2文字1組
- 空白区切り
- 同じ文字を複数ペアに使ってはいけない
- A-A のような自己接続は禁止
- 不正な入力はエラー表示し、変換を実行しない
- 空欄の場合は恒等写像

### 5.3 ロータ順方向変換

入力 index を `x`、ロータ位置を `pos`、リング設定を `ring` とします。

```text
shifted = (x + pos - ring + 26) % 26
wired = wiring[shifted]
result = (wired - pos + ring + 26) % 26
```

### 5.4 ロータ逆方向変換

逆方向では、配線の逆写像を使います。

```text
shifted = (x + pos - ring + 26) % 26
wired = inverseWiring[shifted]
result = (wired - pos + ring + 26) % 26
```

### 5.5 リフレクタ変換

リフレクタは位置・リング設定を持たず、単純な置換として扱います。

---

## 6. UI 仕様

### 6.1 画面構成

以下のセクションを持ちます。

1. タイトル
2. Enigma 設定
3. プラグボード設定
4. 入力
5. 操作ボタン
6. 出力
7. 変換ログ

### 6.2 Enigma 設定

#### ロータ選択

- Left rotor: I〜V
- Middle rotor: I〜V
- Right rotor: I〜V

推奨初期値：

```text
Left = I
Middle = II
Right = III
```

同一ロータの重複選択は、実機運用としては不自然なため警告を出します。

実装方針はどちらでも可：

- 変換を禁止する
- 警告のみ出して変換は許可する

推奨は「変換禁止」です。

#### リフレクタ選択

- UKW B
- UKW C

初期値：

```text
UKW B
```

#### リング設定

- Left ring: A〜Z
- Middle ring: A〜Z
- Right ring: A〜Z

初期値：

```text
AAA
```

#### 初期ロータ位置

- Left position: A〜Z
- Middle position: A〜Z
- Right position: A〜Z

初期値：

```text
AAA
```

### 6.3 入力欄

複数行テキストエリア。

### 6.4 操作ボタン

- Encrypt / Decrypt
- Reset positions
- Clear

Enigma では暗号化と復号が同じ操作なので、ボタン名は `Encrypt / Decrypt` とします。

### 6.5 出力表示

- 変換結果
- 最終ロータ位置
- 変換ログ

### 6.6 変換ログ

ログには最低限以下を表示します。

| # | Input | Position before | Output | Position after |
|---|---|---|---|---|

将来拡張として、各ロータ通過後の値も表示可能です。

---

## 7. データ構造案

```javascript
const ALPHABET = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";

const ROTORS = {
  I:   { wiring: "EKMFLGDQVZNTOWYHXUSPAIBRCJ", notch: "Q" },
  II:  { wiring: "AJDKSIRUXBLHWTMCQGZNPYFVOE", notch: "E" },
  III: { wiring: "BDFHJLCPRTXVZNYEIWGAKMUSQO", notch: "V" },
  IV:  { wiring: "ESOVPZJAYQUIRHXLNFTGKDCMWB", notch: "J" },
  V:   { wiring: "VZBRGITYUPSDNHLXAWMJQOFECK", notch: "Z" }
};

const REFLECTORS = {
  B: "YRUHQSLDPXNGOKMIEBFZCWVJAT",
  C: "FVPJIAOYEDRZXWGCTKUQSBNMHL"
};
```

---

## 8. 主要関数案

### 8.1 文字変換ユーティリティ

```javascript
function charToIndex(ch)
function indexToChar(i)
function mod26(n)
```

### 8.2 配線変換

```javascript
function buildInverseWiring(wiring)
function encodeRotorForward(x, rotor, pos, ring)
function encodeRotorReverse(x, rotor, pos, ring)
function encodeReflector(x, reflector)
```

### 8.3 プラグボード

```javascript
function parsePlugboard(text)
function applyPlugboard(x, plugboardMap)
```

### 8.4 ロータ進行

```javascript
function isAtNotch(rotorName, pos)
function stepPos(pos)
function stepRotors(state)
```

### 8.5 1文字変換

```javascript
function encodeChar(ch, state)
```

処理順：

1. 非A-Zならそのまま返す
2. ロータを進める
3. plugboard
4. right rotor forward
5. middle rotor forward
6. left rotor forward
7. reflector
8. left rotor reverse
9. middle rotor reverse
10. right rotor reverse
11. plugboard
12. 出力

### 8.6 全文変換

```javascript
function encodeText(text, initialState)
```

---

## 9. 受け入れ条件

### 9.1 基本動作

- index.html 単体で動く
- Cloudflare Pages に配置して動く
- 外部ライブラリ不要
- スマートフォンでも最低限操作できる

### 9.2 暗号仕様

- ロータ I〜V が選択できる
- UKW B/C が選択できる
- リング設定が反映される
- 初期位置が反映される
- プラグボードが反映される
- double stepping が実装される
- 暗号化と復号が同一処理になる

### 9.3 エラー処理

以下の場合、変換せずにエラーを表示します。

- ロータ重複
- プラグボードの同一文字重複
- プラグボードの自己接続
- プラグボードに英字以外が含まれる
- プラグボードのペアが2文字でない

---

## 10. テスト観点

### 10.1 可逆性テスト

同じ設定・同じ初期位置で、暗号文を再入力すると平文に戻ること。

例：

1. 初期設定で `HELLOWORLD` を変換
2. 得られた暗号文を、同じ初期設定で再変換
3. `HELLOWORLD` に戻ること

### 10.2 ロータ進行テスト

- A-Z文字ではロータが進む
- 空白・記号ではロータが進まない
- 右ロータが毎回進む
- ノッチ位置で中ロータが進む
- double stepping が起きる

### 10.3 プラグボードテスト

- `AB` 設定時、A と B が入れ替わる
- `AB AC` はエラー
- `AA` はエラー

---

## 11. 将来拡張

以下は将来対応とします。

- ロータ VI, VII, VIII
- M3互換プロファイル
- M4互換プロファイル
- Beta / Gamma ロータ
- UKW B thin / C thin
- 表示用の信号経路アニメーション
- URLクエリによる設定共有
- 既知テストベクトル集の追加
- PCB実機との相互通信テスト用モード

---

## 12. 実装方針メモ

本アプリは、暗号強度よりも歴史的互換性と教育的理解を優先します。

特に重要なのは以下です。

1. 配線表を正しく実装する
2. リング設定を正しく反映する
3. ロータ進行を文字変換前に行う
4. double stepping を正しく扱う
5. 同一設定で暗号化・復号が同じ操作になることを保つ

以上を満たせば、Paper Enigma や他の Enigma I 互換シミュレータとの相互検証が可能になります。
