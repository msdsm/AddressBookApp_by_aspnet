# ASP.NETで住所録アプリ開発
## ソース
- Udemy:【入門者向け】ASP.NET MVCでWebアプリ開発のノウハウを学ぼう！

## 概要
- ブログアプリとtodoアプリはコードファーストで開発したが、今回はデータベースファーストで作成する
    - コードファースト:C#のmodelクラスから自動でデータベースのテーブル作成した
    - データベースファースト:データベースからモデルクラスのコード自動作成
- 以下の2つの入力制限を行う
    - 列挙型をもとにしたドロップダウンリスト
    - 正規表現を使用した入力チェック
## 作業進捗
### DB作成
- サーバーエクスプローラーから新しくDB作成
- SQLクエリ開く
- CREATE TABLEで２つ作る
```sql
CREATE TABLE [dbo].[Groups] (
  [Id] INT IDENTITY (1, 1) NOT NULL,
  [Name] NVARCHAR (200) NOT NULL,
  CONSTRAINT [PK_dbo.Groups] PRIMARY KEY CLUSTERED ([Id] ASC),
  CONSTRAINT [AK_dbo.GroupName] UNIQUE([Name])
);

CREATE TABLE [dbo].[Addresses] (
  [Id] INT IDENTITY (1, 1) NOT NULL,
  [Name] NVARCHAR (100) NOT NULL,
  [Kana] NVARCHAR (200) NOT NULL,
  [ZipCode] NVARCHAR (7) NULL,
  [Prefecture] NVARCHAR (10) NULL,
  [StreetAddress] NVARCHAR (600) NULL,
  [Telephone] NVARCHAR (11) NULL,
  [Mail] NVARCHAR (128) NULL,
  [Group_Id] INT NULL,
  CONSTRAINT [PK_dbo.Addresses] PRIMARY KEY CLUSTERED ([Id] ASC),
  CONSTRAINT [FK_dbo.Addresses.Group_Id] FOREIGN KEY ([Group_Id]) REFERENCES [dbo].[Groups] ([Id]) ON DELETE CASCADE
);
```
- 左上のexecute(デバッグと同じマーク)を実行
- table作られていることを確認
### プロジェクト作成
- 空、MVCで作成
- EntityFrameworkをNugetパッケージからインストール
### model追加
- Modelsを右クリックして追加->新しい項目->すべてのテンプレートを表示
- 左の項目からデータを選択して、ADO.NET Entity Data Modelを選択
- `AddressBookModel`として作成
- データベースからEF designerを選択
- データベースを先ほど作成しtAddressBookInfoを選択
- モデルに含めるデータベースオブジェクトでテーブルにチェック
- 生成されたオブジェクトの名前を複数化にチェック
- modelsにAddressBookModel.edmxファイルが生成される
- AddressBookMode.edmxを展開してAddressBookModel.ttの中にAddress.csとGroup.csがある
    - これらがテーブルから自動生成されたPOCOになる
    - これらのモデルクラスは編集してはいけない
- 各プロパティにディスプレイネームを指定したい場合はモデルクラスを編集してはいけないのでメタクラスを作る
    - `AddressMetadata.cs`作成(Models右クリック->追加->クラス)
        - namespaceが一致していればpartial classで定義できる
    - `GroupMetadata.cs`作成して同じことする
- EntityFrameworkとEntityFramework.jaのversionが一致するようにEntityFrameworkのバージョン落とす
    - 6.2.0でそろえた
    - この後コントローラー生成するときにエラー出て解消できなかった
    - あきらめてEntityFramework.jaをアンインストールしてEntityFrameworkだけで進めた

### Controllerの作成
- Controllerを作成する前に一度ソリューションのビルドを行う
- address,groupに対してコントローラー作成
- ビュー修正

### 入力制限の実装
- enum型で都道府県管理
    - model,Controller,viewをそれぞれ編集
- 正規表現で入力制限
    - AddressMetadataにRegularExpressionアノテーションつける
        - メールアドレスについては正規表現でチェックすると複雑な式になる
        - そのためDataType.EmailAddressを使う
- 動作確認した

### 検索機能の実装
- SerchViewModel作成
    - カナ検索のため
- Controller修正
    - Searchメソッド実装
- view作成
- RouteConfigと共通レイアウト編集
    - IndexではなくSearchに変更
- 動作確認のために疑似個人情報作成サイトからcsvファイル作成
- Visual Studioでファイル->新規プロジェクトの作成->asp.netコマンドコンソールを選択
- ソリューションを作成するのではなくaddressbookappを選択
- nugetからentityframeworkとcsvhelperインストール
- Program.csでcsvをデータベースに登録する処理実装
- デバッグから実行してデータベース確認
### 動作確認
- 検索できた