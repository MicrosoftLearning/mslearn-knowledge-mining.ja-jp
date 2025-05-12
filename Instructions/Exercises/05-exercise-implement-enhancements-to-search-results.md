---
lab:
  title: 検索結果に拡張機能を実装する
---

# 検索結果に拡張機能を実装する

旅行予約アプリで使用される既存の検索サービスがあります。 検索結果の関連性が、予約の数に影響を与えていることに気付きました。 最近、ポルトガルのホテルも追加したので、サポート言語としてポルトガル語を提供したいと考えています。

この演習では、検索結果の関連性を向上させるためにスコアリング プロファイルを追加します。 その後、Azure AI サービスを使用して、すべてのホテルにポルトガル語の説明を追加します。

> **注** この演習を完了するには、Microsoft Azure サブスクリプションが必要です。 アカウントを取得済みでない場合は、[https://azure.com/free](https://azure.com/free?azure-portal=true) から無料評価版にサインアップできます。

## Azure リソースを作成する

Azure AI Search サービスを作成し、サンプル ホテル データをインポートします。

1. [Azure portal](https://portal.azure.com/learn.docs.microsoft.com?azure-portal=true) にサインインします。
1. **+ リソースの作成**を選択します。
1. 「**search**」を検索し、**[Azure AI Search]** を選びます。
1. **［作成］** を選択します
1. [リソース グループ] で **[新規作成]** を選び、**learn-advanced-search** という名前を付けます。
1. **[サービス名]** に「**advanced-search-service-12345**」と入力します。 名前はグローバルに一意である必要があるため、名前の末尾に乱数を追加します。
1. お近くのサポート対象リージョンを選択します。
1. **[価格レベル]** には既定値を使用します。
1. **[Review + create] (確認および作成)** を選択します。
1. **［作成］** を選択します
1. リソースがデプロイされるまで待ってから、**[リソースグループに移動]** を選択します。

### 検索サービスにサンプル データをインポートする

サンプル データをインポートします。

1. **[概要]** ウィンドウで、**[データのインポート]** を選択します。

    ![[データのインポート] メニューを示すスクリーンショット。](../media/05-media/import-data-new.png)
1. **[データのインポート]** ペインの **[データ ソース]** ドロップダウンで **[サンプル]** を選びます。
1. **[hotels-sample]** を選択します。

1. **[コグニティブ スキルの追加 (省略可能)]** タブで **[AI サービスのアタッチ]** を展開し、**[新しい AI サービス を作成する]** を選びます。

    ![Azure AI サービスの選択、追加を示すスクリーンショット。](../media/05-media/add-cognitive-services-new.png)

### 翻訳をサポートする Azure AI サービスを作成する

1. 新しいタブで Azure portal にサインインします。
1. **[リソース グループ]** で、**[learn-advanced-search]** を選択します。
1. **[リージョン]** で、検索サービスに選択したのと同じリージョンを選択します。
1. **[名前]** に「**learn-cognitive-translator-12345**」または任意の名前を入力します。 名前はグローバルに一意である必要があるため、名前の末尾に乱数を追加します。
1. **[価格レベル]** で **[Standard S0]** を選択します。
1. **[このボックスをオンにすることにより、以下のすべてのご契約条件を読み、同意したものとみなされます]** を選択します。
1. **[Review + create] (確認および作成)** を選択します。
1. **［作成］** を選択します
1. リソースが作成されたら、タブを閉じます。

### 翻訳エンリッチメントを追加する

1. **[コグニティブ スキルの追加 (省略可能)]** タブで、[最新の情報に更新] を選択します。
1. 新しいサービスの **learn-cognitive-translator-12345** を選びます。
1. **[エンリッチメントの追加]** セクションを展開します。
    ![ポルトガル語翻訳の追加を示すスクリーンショット。](../media/05-media/add-translation-enrichment-new.png)
1. **[テキストの翻訳]** を選択し、**[ターゲット言語]** を **[ポルトガル語]** に変更してから、**[フィールド名]** を「**Description_pt**」に変更します。
1. **[次へ: 対象インデックスをカスタマイズします]** を選択します。

### 翻訳されたテキストを格納するようにフィールドを変更する

1. **[対象インデックスをカスタマイズします]** タブで、フィールド リストの一番下までスクロールし、**Description_pt** フィールドの **[アナライザー]** を **[ポルトガル語 (ポルトガル) - Microsoft]** に変更します。
1. **[次へ: インデクサーの作成]** を選択します。
1. **[Submit] (送信)** をクリックします。

    インデックスが作成され、インデクサーが実行され、サンプル ホテル データを含む 50 のドキュメントがインポートされます。
1. **[概要]** ペインで **[インデックス]** を選択し、**[hotels-sample-index]** を選択します。
1. **[検索]** を選択すると、インデックス内のすべてのドキュメントの JSON が表示されます。
1. 結果で **Description_pt** を検索します (これには **Ctrl + F** キーを使用できます)。これは英語の説明のポルトガル語翻訳ではなく、次のように表示されることに注意してください。

    ```json
    "Description_pt": "45",
    ```

Azure portal では、ドキュメント内の最初のフィールドを翻訳する必要があると想定しています。 そのため、現在は翻訳スキルを使用して `HotelId` を翻訳しています。

### ドキュメント内の正しいフィールドを翻訳するようにスキルセットを更新する

1. ページの上部にある検索サービス **advanced-search-service-12345 |Indexes** リンクを選びます。
1. 左ペインにある [検索管理] で **[スキルセット]** を選び、次に **hotels-sample-skillset** を選びます。
1. JSON ドキュメントを編集し、9 行目を次に変更します。

    ```json
    "context": "/document/Description",
    ```

1. 11 行目の既定の開始言語を英語に変更します。

    ```json
    "defaultFromLanguageCode": "en",
    ```

1. 15 行目のソース フィールドを次に変更します。

    ```json
    "source": "/document/Description",
    ```

1. **[保存]** を選択します。
1. ページの上部にある検索サービス **advanced-search-service-12345 | Skillsets** リンクを選びます。
1. **[概要]** ペインで **[インデクサー]** を選択し、**[hotels-sample-indexer]** を選択します。
1. **[JSON の編集]** を選択します。
1. 20 行目のソース フィールド名を次に変更します。

    ```json
    "sourceFieldName": "/document/Description/Description_pt",
    ```

1. **[保存]** を選択します。
1. **[リセット]** を選び、次に **[はい]** を選びます。
1. **[実行]** を選び、次に **[はい]** を選びます。

### 更新したインデックスをテストする

1. ページの上部にある検索サービス **advanced-search-service-12345 | Indexers** リンクを選びます。
1. **[概要]** ペインで **[インデックス]** を選択し、**[hotels-sample-index]** を選択します。
1. **[検索]** を選択すると、インデックス内のすべてのドキュメントの JSON が表示されます。
1. 結果で「**Description_pt**」と検索し、ポルトガル語の説明が表示されていることを確認します。

    ```json
    "Description_pt": "O maior resort durante todo o ano da área oferecendo mais de tudo para suas férias – pelo melhor valor!  O que você pode desfrutar enquanto estiver no resort, além das praias de areia de 1,5 km do lago? Confira nossas atividades com certeza para excitar tanto os jovens quanto os jovens hóspedes do coração. Temos tudo, incluindo ser chamado de \"Propriedade do Ano\" e um \"Top Ten Resort\" pelas principais publicações.",
    ```

1. 次に湖を望むホテルを検索します。 まず、`HotelName`、`Description`、`Category`、`Tags` のみを返す単純な検索を使います。 **[クエリ文字列]** に、次の検索を入力します。

    `lake + view&$select=HotelName,Description,Category,Tags&$count=true`

    結果を調べて、`lake` と `view` の検索語句が一致するフィールドを探してみてください。 このホテルとその位置に注目してください。

    ```json
    {
      "@search.score": 0.9433406,
      "HotelName": "Lady Of The Lake B & B",
      "Description": "Nature is Home on the beach.  Save up to 30 percent. Valid Now through the end of the year. Restrictions and blackout may apply.",
      "Category": "Luxury",
      "Tags": [
        "laundry service",
        "concierge",
        "view"
      ]
    },
    ```

このホテルは、`HotelName` フィールドの lake と、`Tags` フィールドの view の用語が一致しています。 ホテル名の `Description` フィールドの用語のマッチ率を上げたいと思います。 このホテルが結果の最後になるのが理想的です。

## スコアリング プロファイルを追加して検索結果を改善する

1. **[スコアリング プロファイル]** タブを選択します。
1. **[+ スコアリング プロファイルの追加]** を選択します。
1. **[プロファイル名]** に「**boost-description-categories**」と入力します。
1. **[重み]** で、次のフィールドと重みを追加します。

    ![スコアリング プロファイルに追加されている重みのスクリーンショット。](../media/05-media/add-weights-new.png)
1. **[フィールド名]** の **[Description]** を選択します。
1. **[重み]** に「**5**」と入力します。
1. **[フィールド名]** の **[Category]** を選択します。
1. **[重み]** に「**3**」と入力します。
1. **[フィールド名]** の **[Tags]** を選択します。
1. **[重み]** に「**2**」と入力します。
1. **[保存]** を選択します。
1. 上部にある**保存**を選択します。

### 更新したインデックスをテストする

1. **[hotels-sample-index]** ページの **[検索エクスプローラー]** タブに戻ります。
1. **[クエリ文字列]** に、前と同じ検索を入力します。

    `lake + view&$select=HotelName,Description,Category,Tags&$count=true`

    検索結果を調べます。

    ```json
    {
      "@search.score": 3.5707965,
      "HotelName": "Lady Of The Lake B & B",
      "Description": "Nature is Home on the beach.  Save up to 30 percent. Valid Now through the end of the year. Restrictions and blackout may apply.",
      "Category": "Luxury",
      "Tags": [
        "laundry service",
        "concierge",
        "view"
      ]
    }
    ```

    検索スコアが **0.9433406** から **3.5707965** に増加しています。 しかし、他のどのホテルも、それより高い計算スコアになっています。 このホテルが、結果の最後になりました。

## クリーンアップ

これで演習が完了したので、不要になったすべてのリソースを削除します。

1. Azure portal で、**[リソース グループ]** を選択します。
1. 不要なリソース グループを選び、**[リソース グループの削除]** を選びます。
