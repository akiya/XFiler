# JavaScript マクロ ガイド

XFiler では JavaScript (Jint エンジン) を使用して、ファイラーの動作を自動化したり、独自の機能を追加したりすることができます。

## 概要
- マクロファイル（`.js`）は、実行ファイルと同じディレクトリにある `macro` フォルダに配置します。
- XFiler の起動時に、`macro` フォルダ内のすべての `.js` ファイルが読み込まれます。
- マクロ内では、グローバルオブジェクト `XFiler` を通じて各機能にアクセスできます。

## マクロの登録
`XFiler.RegisterCommand` を使用して、アプリケーションにマクロを登録します。登録されたマクロは、メニューやショートカット（`ExecuteMacro` アクション）から実行できます。

```javascript
XFiler.RegisterCommand("hello_macro", "挨拶を表示", function() {
    XFiler.ShowMessage("こんにちは、XFilerマクロの世界へ！", "挨拶");
});
```

## セキュリティとファイルアクセス制限
XFiler のマクロは、誤操作や悪意のあるスクリプトからシステムを保護するため、書き込み操作に制限が設けられています。

### 作業ディレクトリ制限 (サンドボックス)
デフォルトでは、マクロは**現在アクティブなペインおよびパッシブなペインが表示しているディレクトリ**（作業ディレクトリ）とその配下に対してのみ、ファイルの作成・修正・削除が許可されます。  
それ以外の場所（ルートディレクトリや別ドライブなど）へ直接書き込みを行おうとすると、セキュリティ例外が発生しマクロが中断されます。

### 外部パスの許可
作業ディレクトリ外の特定のパスへアクセスする必要がある場合は、`XFiler.AllowExternalPath(path)` を使用して一時的にアクセスを許可することができます。  
設定で「確認ダイアログを出す」が有効な場合、この関数を呼んだタイミングでユーザーに許可を求めるダイアログが表示されます。

### 制限の無効化
設定ウィンドウの「マクロ」タブから「マクロからの書き込み制限をしない」を有効にすることで、このセキュリティ制限を完全に無効化できます。  
ただし、PC上のすべての(Windowsによって許可されている)ファイルに対して書き込みが可能になるため、使用には十分注意してください。

----

## API リファレンス

### 制御・登録
- **`XFiler.RegisterCommand(id: string, displayName: string, action: function): boolean`**
  - マクロをコマンドとして登録します。`id` は一意な識別子、`displayName` はメニュー等に表示される名称です。既に同じIDが登録されている場合は `false` を返します。
- **`XFiler.Log(message: string): boolean`**
  - ファイラーのログエリアにメッセージを出力します。成功すれば `true` を返します。
- **`XFiler.AllowExternalPath(path: string): boolean`**
  - 指定したパスを作業ディレクトリ外への書き込み許可リストに追加します。
  - 許可を求める確認ダイアログが表示される場合があります。拒否された場合は例外をスローしてマクロを即座に停止します。
  - この許可はマクロの1回の実行の間のみ有効です。

### 情報取得
- **`XFiler.GetActivePath(): string`**
  - アクティブなペインの現在のディレクトリパスを返します。
- **`XFiler.GetPassivePath(): string`**
  - 反対側のペイン（パッシブペイン）の現在のディレクトリパスを返します。
- **`XFiler.GetMarkedItems(isPassive: boolean = false): string[]`**
  - 指定したペインでマークされているアイテムの**相対パス**一覧を返します。
- **`XFiler.GetSelectedItems(isPassive: boolean = false): string[]`**
  - 指定したペインでマークされているアイテムがあればその一覧を、なければ現在カーソルがあるアイテムの相対パスを配列で返します。
- **`XFiler.GetFileSize(path: string, isPassive: boolean = false): number`**
  - 指定されたファイルのサイズ（バイト単位）を返します。ファイルが存在しない、またはディレクトリの場合は `-1` を返します。
- **`XFiler.GetAllItems(isPassive: boolean = false): string[]`**
  - 指定したペインにある（表示されている）すべてのアイテム（親ディレクトリ `..` を除く）の**相対パス**一覧を返します。
- **`XFiler.MarkItem(path: string, mark: boolean = true): boolean`**
  - 指定されたパスのアイテムをマーク、またはマーク解除します。パスは相対パスまたはフルパスです。成功すれば `true` を返します。
- **`XFiler.ClearMark(): boolean`**
  - アクティブペインのすべてのアイテムのマークを解除します。成功すれば `true` を返します。
- **`XFiler.Exists(path: string, isPassive: boolean = false): boolean`**
  - 指定されたパス（ファイルまたはディレクトリ）が存在するか確認します。`isPassive` が `true` なら反対側のペインを基準にします。
- **`XFiler.IsDirectory(path: string, isPassive: boolean = false): boolean`**
  - 指定されたパスがディレクトリであるか確認します。
- **`XFiler.GetFiles(path: string, isPassive: boolean = false): string[]`**
  - 指定されたディレクトリ内のファイル名（名前のみ）の一覧を返します。
- **`XFiler.GetDirectories(path: string, isPassive: boolean = false): string[]`**
  - 指定されたディレクトリ内のディレクトリ名（名前のみ）の一覧を返します。
- **`XFiler.GetItems(path: string, isPassive: boolean = false): string[]`**
  - 指定されたディレクトリ内のすべてのアイテム（ファイルおよびディレクトリ。名前のみ）の一覧を返します。
- **`XFiler.GetAttributes(path: string, isPassive: boolean = false): object | null`**
  - 指定されたパス（ファイルまたはディレクトリ）の属性をオブジェクト形式で返します。見つからない場合は `null` を返します。
  - 返されるオブジェクトの構造： `{ readOnly: bool, hidden: bool, system: bool, archive: bool }`

### ダイアログ
- **`XFiler.ShowMessage(message: string, title?: string): boolean`**
  - メッセージボックスを表示します。成功すれば `true` を返します。
- **`XFiler.Confirm(message: string, title?: string): boolean`**
  - 「はい」「いいえ」の確認ダイアログを表示します。「はい」なら `true` を返します。
- **`XFiler.Prompt(message: string, title?: string, defaultText?: string): string | null`**
  - 文字列入力ダイアログを表示します。キャンセルされた場合は `null` を返します。

### ファイル操作 (単体)
マクロ内でのパス指定には以下のルールが適用されます：
- **相対パスの場合**:
    - **操作元（srcなど）**: **アクティブなペイン**のパスが基準となります。
    - **操作先（destなど）**: **反対側のペイン（パッシブペイン）**のパスが基準となります。
- **フルパスの場合**: 指定されたパスがそのまま使用されます。

このルールにより、`XFiler.CopyFile("test.txt", "test.txt")` と記述するだけで、アクティブペインのファイルを反対側のペインへ同名でコピーできます。同一ペイン内で操作したい場合は、`XFiler.GetActivePath()` を使用してフルパスを組み立ててください。

※ コピーや移動時、同名ファイルがある場合は XFiler 標準の上書き確認ダイアログが表示されます。

- **`XFiler.CopyFile(src: string, dest: string): boolean`**
  - ファイルをコピーします。
- **`XFiler.MoveFile(src: string, dest: string): boolean`**
  - ファイルを移動します。
- **`XFiler.DeleteFile(path: string, isPassive: boolean = false): boolean`**
  - ファイルを削除します（設定によりごみ箱へ送られます）。
- **`XFiler.CopyDirectory(src: string, dest: string): boolean`**
  - ディレクトリを再帰的にコピーします（システム機能を使用）。
- **`XFiler.MoveDirectory(src: string, dest: string): boolean`**
  - ディレクトリを移動します。
- **`XFiler.DeleteDirectory(path: string, isPassive: boolean = false): boolean`**
  - ディレクトリを削除します。
- **`XFiler.Rename(path: string, newName: string, isPassive: boolean = false): boolean`**
  - ファイル名またはディレクトリ名を変更します。
- **`XFiler.SetAttributes(path: string, attr: object, isPassive: boolean = false): boolean`**
  - ファイルの属性を設定します。`attr` には `XFiler.GetAttributes` で取得した形式のオブジェクトを渡します。
- **`XFiler.CreateDirectory(path: string, isPassive: boolean = false): boolean`**
  - 新しいディレクトリを作成します。既に存在する場合は何もしませんが `true` を返します。
- **`XFiler.CreateFile(path: string, isPassive: boolean = false): boolean`**
  - 新しい空のファイルを作成します。
- **`XFiler.CreateShortcut(src: string, dest: string): boolean`**
  - `src`（リンク先）へのショートカットを `dest`（作成場所）に作成します。
  - ルール：`src` はアクティブ基準、`dest` はパッシブ基準です。
  - `dest` が `.lnk` で終わらない場合は自動的に付与されます。

### その他
- **`XFiler.ChangeDirectory(path: string): boolean`**
  - アクティブペインのディレクトリを指定したパスに変更します。成功すれば `true` を返します。
- **`XFiler.ChangePassiveDirectory(path: string): boolean`**
  - 反対側（パッシブ）ペインのディレクトリを指定したパスに変更します。成功すれば `true` を返します。
- **`XFiler.Refresh(): boolean`**
  - ペインの表示内容を最新の状態に更新します。成功すれば `true` を返します。

### パス操作ユーティリティ (`path` オブジェクト)
Node.js の `path` モジュールに準拠したパス操作関数をグローバルオブジェクト `path` として提供しています。

- **`path.join(...paths: string[]): string`**
  - 複数のパスセグメントを結合して一つのパスを作成します。
- **`path.basename(p: string): string`**
  - パスの最後にある名前（ファイル名など）を返します。
- **`path.basenameWithoutExtension(p: string): string`**
  - パスの最後にある名前から、拡張子を除いた部分のみを返します。
- **`path.extname(p: string): string`**
  - パスの拡張子（`.txt` など）を返します。
- **`path.dirname(p: string): string`**
  - パスのディレクトリ部分を返します。

---

## サンプル: 選択したファイルを反対側にバックアップ
```javascript
XFiler.RegisterCommand("backup_to_passive", "反対側へ.bakとしてコピー", function() {
    var selected = XFiler.GetSelectedItems(); // アクティブペインからの相対パス取得

    var count = 0;
    for (var i = 0; i < selected.length; i++) {
        var src = selected[i];         // 例: "memo.txt"
        var dest = src + ".bak";       // 例: "memo.txt.bak"
        
        // 相対パスを渡すと、srcはアクティブ、destはパッシブペインを基準にします。
        // 結果として、反対側のペインに .bak 付きでコピーされます。
        if (XFiler.CopyFile(src, dest)) {
            count++;
        }
    }
    
    XFiler.Log(count + "件を反対側へバックアップしました。");
    XFiler.Refresh();
});
```
