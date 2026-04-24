# ENIGMA-APP

Enigma I（3ロータ）互換の暗号変換器を、ブラウザ上で動作するHTMLアプリとして実装したプロジェクトです。静的Webアプリとして構成されています。

---

## 🚀 デモ

https://enigma-app.pages.dev/

---

## 🎯 目的

本プロジェクトは以下を目的としています：

- 歴史的Enigma（Enigma I）の挙動を正確に再現する
- PCBベースの物理Enigma再現機との互換性検証
- 暗号処理の理解・教育用途
- ロータ・リフレクタ・ステッピングの動作確認

---

## 🧠 対応仕様（Enigma I）

### ロータ
- I, II, III, IV, V

### リフレクタ
- UKW B
- UKW C

### 機能
- プラグボード（Steckerbrett）
- リング設定（Ringstellung）
- 初期ロータ位置（Grundstellung）
- ダブルステップ（double stepping）再現
- 可逆暗号（Encrypt = Decrypt）

---

## ⚙️ 使い方

1. ロータとリフレクタを選択
2. リング設定（通常は AAA）
3. 初期ロータ位置を設定（例: AAA）
4. プラグボードを設定（例: `AV BS CG`）
5. テキストを入力
6. 「実行」を押す

---

## 🔐 暗号仕様のポイント

### ロータ進行

- 右ロータ：毎文字回転
- 中ロータ：右ロータのノッチで回転
- 左ロータ：中ロータのノッチで回転
- ダブルステップを正確に再現

### 信号の流れ


Plugboard  
→ Right Rotor  
→ Middle Rotor  
→ Left Rotor  
→ Reflector  
→ Left Rotor（逆）  
→ Middle Rotor（逆）  
→ Right Rotor（逆）  
→ Plugboard  


---

## 📁 ファイル構成


ENIGMA-APP/  
├── index.html # アプリ本体（単一ファイル）  
├── Spec/  
│ ├── enigma_i_app_prompt.md # AI生成用プロンプト  
│ └── enigma_i_app_spec.md # 実装仕様書  
└── README.md  


---

## 🧪 テスト方法

### 可逆性テスト

同じ設定・同じ初期位置で：

1. 平文を暗号化
2. 得られた暗号文を再入力

→ 元の平文に戻ること

---

## ⚠️ 現在の制限

- ロータ VI〜VIII 未対応
- M3 / M4 未対応
- Beta / Gamma 未対応
- リング設定はUI上変更可能だが、PCB実装では固定予定
- 実機完全再現（機械構造）は対象外

---

## 🔧 今後の予定

- [ ] ロータ VI–VIII 対応
- [ ] M3 / M4 モード追加
- [ ] 配線可視化
- [ ] URL共有機能（設定保存）
- [ ] PCB実機との相互通信テスト
- [ ] テストベクトル追加

---

## 🏗️ 開発方針

- 外部ライブラリなし（Pure HTML / JS）
- 単一ファイル構成
- Cloudflare Pagesで即デプロイ可能
- 歴史的互換性を最優先

---

## 📚 参考資料

- Crypto Museum  
- Enigma Rotor Wiring Tables  
- Turing "Treatise on the Enigma"  
- Franklin Heath Paper Enigma  

---

## 👤 作者

https://github.com/HideakiTakechi

---

## 📝 ライセンス

検討中
