[README.md](https://github.com/user-attachments/files/28863332/README.md)
# BrowserBackup# BrowserBackup — ブラウザ設定バックアップ・リストアツール

軽量・軽快・ランタイム不要の単一バイナリ (約 1.6MB)。 — v1.0


## 対応ブラウザ

Google Chrome / Microsoft Edge / Brave / Mozilla Firefox
（全プロファイル対応。Firefox はリストア時にプロファイル名が異なる場合、既定プロファイルへ自動振替）

## バックアップ対象

| 項目 | Chromium系 | Firefox |
|---|---|---|
| 設定 | Preferences, Secure Preferences, Local State | prefs.js, user.js, search.json.mozlz4 ほか |
| ブックマーク | Bookmarks | places.sqlite |
| パスワード | Login Data | logins.json + key4.db |
| 履歴 | History, Favicons | places.sqlite, favicons.sqlite |
| Cookie | Network\Cookies | cookies.sqlite |
| 自動入力 | Web Data | formhistory.sqlite |
| 拡張機能 | Extensions フォルダ | extensions フォルダ + extensions.json |

Cache / Code Cache / GPUCache / Media Cache / Service Worker / IndexedDB /
LocalStorage / Crashpad / Logs 等は除外（フォルダ名比較のみの高速フィルタ）。

## 使い方

`BrowserBackup.exe` を起動 → ブラウザを選択 → [バックアップ] で zip 保存先を指定。
[リストア] で zip を選択し、上書き確認後に復元。[詳細設定 ▼] で項目ごとの ON/OFF が可能。
進捗・結果はステータスバーにのみ表示（ファイルログなし）。

保存・参照ダイアログの初期フォルダは **exe を置いているフォルダ** です。

**※ 重要: パスワード等の暗号化キーは Windows DPAPI（ユーザー＋マシン）に紐付くため、別マシンへのリストアではパスワード・Cookie は復号できません（同一マシンの復元用）。**

## バックアップ項目のグループと誤検知について

詳細設定の項目は 2 グループに分かれています。

- **通常項目（既定オン）**: 設定 / ブックマーク / 履歴 / 拡張機能
- **機微情報（既定オフ）**: パスワード / Cookie / 自動入力（Web Data）

機微情報グループは、情報窃取型マルウェアの典型的な窃取対象と同じファイル
（`Login Data` / `key4.db` / `Cookies` 等）を扱うため、セキュリティソフトに
**誤検知（exe の削除・処理のブロック）されやすい**項目です。これらを既定でオフにし、
チェックを入れて実行する際は「誤検知に関する注意」ダイアログで除外登録を促します。

通常項目のみ（既定）であれば、実行時の挙動による誤検知リスクは大きく下がります。
ただし **exe ファイル自体**が署名なし・無評価のため、機微情報を扱うコードを含む
バイナリとして静的に検知される可能性は残ります。確実に運用するには、
exe を置いたフォルダをセキュリティソフトの除外（ホワイトリスト）に登録してください。

### Windows Defender に誤検知された場合

`fix-defender.ps1` を**管理者権限**で実行すると、`target-path.txt` に書かれた
フォルダを Defender の除外パスに登録し、隔離を解除して exe を復元します。
他社製セキュリティソフトの場合は、同様に当該フォルダを手動で除外登録してください。

## CLI（動作確認用、`BrowserBackup-test.exe`）

```
BrowserBackup-test.exe --detect
BrowserBackup-test.exe --backup  backup.zip [Chrome,Edge,Brave,Firefox]
BrowserBackup-test.exe --restore backup.zip [Chrome,Edge,Brave,Firefox]
```

## ビルド

```
build.cmd
```

- コンパイラ: [bflat](https://github.com/bflattened/bflat) 8.0.2
  — .NET 8 の Native AOT コンパイラ。lld リンカ内蔵のため **MSVC 不要**。
- 仕様書の Native AOT / InvariantGlobalization / トリミング要件は
  `-Os --no-globalization --no-debug-info` で充足。
- UI は WPF ではなく Win32 API 直接呼び出し（WPF/WinUI 3 は Native AOT 非対応のため）。
  サードパーティライブラリ不使用、System.IO / System.IO.Compression のみ。

## 実測値

- 実行ファイル: 1.6MB（目標 10MB 以下）
- 起動+全ブラウザ検出: 約 26ms
- メモリ使用量: 約 14MB
