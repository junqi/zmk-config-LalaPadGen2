# macOS ディクテーション用 Function Key の選定

## 背景

macOS のディクテーション (音声入力) は標準で Globe (🌐) キー / `fn` 二度押し、または専用マイクキーで起動する。カスタムキーボード (ZMK など) からこれを再現する場合:

- **🌐 (Globe) キー**は Apple Vendor HID Page を使っており、非 Apple 製キーボードでは送出できない
- **HID Consumer の `AC Voice Command` (0xCF)** は macOS の対応が不安定で、バージョン/設定によって発火しない
- **専用マイクキー (F5)** は古い Mac / Touch ID キーボードでデフォルト割当があるが、これも本体特有挙動で再現性が低い

そのため、最も堅い運用は:

1. macOS の **System Settings → Keyboard → Dictation → Shortcut** に任意の Function Key を割り当てる
2. カスタムキーボード側でその F-key を送る (ZMK なら `&kp F<N>`)

MacBook 本体キーボードでも完結させたいので **F1–F12 から選定**。開発で日常的に使うアプリと衝突しないキーを選ぶのが目的。

## アプリごとの F-key 使用 (デフォルト, macOS)

| F-key | Chrome | iTerm2 / Ghostty | Slack | Xcode | Android Studio | macOS |
|---|---|---|---|---|---|---|
| F1 | - | (透過) | - | - | Quick Doc? | - |
| F2 | - | (透過) | - | - | Next Error | - |
| F3 | Find next | (透過) | - | - | (keymap 依存) | - |
| F4 | - | (透過) | - | - | Jump to Source | - |
| F5 | Reload | (透過) | - | - | Copy | - |
| F6 | パネル切替 | (透過) | パネル切替 | Step Over | Move | - |
| F7 | Caret browsing | (透過) | - | Step Into | Step Into | - |
| F8 | - | (透過) | - | Step Out | Step Over | - |
| F9 | - | (透過) | - | - | **Resume Program** | - |
| **F10** | - | (透過) | - | - | - | - |
| F11 | Fullscreen | (透過) | - | - | Toggle Bookmark | Show Desktop |
| F12 | DevTools | (透過) | - | - | Last Tool Window | - |

備考:

- iTerm2 / Ghostty は F-key を裸で消費せず TUI に透過するだけ。macOS のシステムショートカット (Dictation) は最優先で割り込むので、ターミナルが前面でも問題なく発火する
- MacBook 本体キーボードでは F-key のメディア機能 (音量等) が優先のため、生の F-key を送るには `fn` + F-key、または System Settings の "Use F1, F2, etc. keys as standard function keys" を ON にする
- Xcode と Android Studio が **F6 / F7 / F8 をデバッガで占有**、Android Studio はさらに **F9 (Resume Program)** を使うため、F6–F9 帯は避けたい

## 結論

**F10 を Dictation ショートカットに割り当てる。**

- Chrome / iTerm2 / Ghostty / Slack / Xcode / Android Studio いずれもデフォルトで未バインド
- macOS のシステム機能はミュートで、Control Center で代替可能
- F9 は Android Studio の Resume Program と被るため不可、F12 は Chrome DevTools と被るため不可、F11 は macOS Show Desktop と被るため不可、という消去法でも残る

## 設定手順 (要約)

1. ZMK キーマップでマイク用に割り当てたいキーを `F10` にする
2. macOS: System Settings → Keyboard → Dictation を ON にする
3. 同画面の "Shortcut" → "Customize..." で F10 を押して登録
4. 必要に応じて System Settings → Keyboard → "Use F1, F2, etc. keys as standard function keys" を ON にする (本体キーボードからも `fn` 無しで発火させたい場合)
