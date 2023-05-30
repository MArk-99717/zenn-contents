---
title: "「APIによるコンテンツなどの配信連携」第３回～Next.jsとNotionAPIの連携方法について～"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Nextjs","Notion","NotionAPI"]
published: true
---
こんにちは。株式会社トッカシステムズの[@mark_99](https://zenn.dev/mark_99)です。
今回の内容は
社内研修の一環としてテーマ「APIによるコンテンツ等の配信連携」についての技術発表に取り組んだ内容を紹介していきます。
連載記事の第三回目の記事となります。
#### 前回までの記事
**[第一回]**
**(https://zenn.dev/toccaduong/articles/d0e9d77e59d496)**
**[第二回]**
**(https://zenn.dev/mmorichika/articles/mmorichika-notionapi)**

前記事にもありますが
社内のナレッジ/タスク管理ツールとして最近人気の「Notion」が導入されたこと、
前述の連載のテーマとしてAPIを利用した配信連携の技術ナレッジを用いることに関するから、
今回のNotionAPIを使った技術を調べてみようという事で、チーム内の連載記事のお題が決まりました。

### 今回の目標
-----
Next.jsの構築とNotionaAPIの構築
ホームページにNotionから取得した内容を画面表示するまでの手順を実習しながら紹介していきます。

### 事前に用意が必要なツールに関して
-----
- VisualStudioCode 
　[導入の際参考にした記事](https://zenn.dev/protoout/articles/39-visual-studio-code)
- git 
　[導入の際参考にした記事](https://zenn.dev/longbridge/articles/5d11bd51665dac)
- Notion
[（第二回で解説していますので、ご参考にどうぞ）](https://zenn.dev/mmorichika/articles/mmorichika-notionapi)

### Next.jsの導入
-----
Next.jsの導入については下記の記事を参考に環境構築について学びました。
[参考までに](https://zenn.dev/knagano/articles/zenn-article-1)
※こちらの環境構築時のポート番号が重複してしまう場合は、導入時のサーバーを終了してからこの後の作業に取り掛かるか、ポート番号をずらして作業するようにする。

### 参考にした動画はこちらです
-----
https://www.youtube.com/watch?v=My9rXHRn8zc

### next.jsにgit cloneする
-----
まず初めにnext.jsのディレクトリにgitからテンプレートをクローンしてくる作業を行う。
```js
$ cd nextkjs/
(作っていない場合は > $ mkdir nextjs/)
```
ターミナルより下記コマンドの入力を行い、Gitのテンプレートのクローンを行う.
```js
$ git clone https://github.com/Shin-sibainu/notion-blog-template.git　notion-blog-ts
```

### /node_modulesのインストール
-----
ターミナルで下記コマンドの入力を行い、/node_modulesのインストールを行う。
```js
$ npm I
```
![image](/images/zenn-article-01/blog01.png)
（ワークスペース内の.gitignoreから/node_modulesが無視されていることを確認できる）

### ローカルでサーバーを立ち上げる
-----
下記のコマンドの入力でローカルでサーバーを立ち上げる準備をする。
```js
$ npm run dev
```
下記表示がされポート3000でサーバーの立ち上げができていることを確認する
```js
> nextjs@0.1.0 dev
> next dev
ready - started server on 0.0.0.0:3000, url: http://localhost:3000
Browserslist: caniuse-lite is outdated. Please run:
npx browserslist@latest --update-db
```
ブラウザの検索欄にlocalhost:3000を入力して、下記の画面が出ることを確認する。
(この時点ではpostsの値が無いため)
![image](/images/zenn-article-01/blog02.png)

### Notionの設定
-----
前回の記事で行ったインテグレーション時に取得できるシークレットコードを控えておく。
**※このシークレットコードは公開するとブログの記事などを外部から編集されてしまう恐れがある為、第三者に見せないようにする**
![image](/images/zenn-article-01/blog03.png)
![image](/images/zenn-article-01/blog04.png)
### VSCで連携作業
-----
・VSCに戻り、ワークスペース内に新規ファイル「.env」を作成し、下記の記述を行う。
```js
NOTION_TOKEN=“[シークレットコード]"
NOTION_DATABASE_ID=“[データベースのID]“
```
各入力値については下記のものとする。
シークレットコード：インテグレーションで登録を行ったもの
データベースのID：Notionのデータベース画面の[リンクをコピー]したものから下記の[データベースのID]部分の英数字の羅列
```
"www.notion.so/[データベースのID]?v=[～]=4"
```

### Notion.jsの作成
-----
**1行目から追記**
```js:Notion.js
//notion Clientを追加
import {Client} from "@notionhq/client";
  const notion = new Client({
  auth: process.env.NOTION_TOKEN,
});
  //getDatabaseを追加
export const getDatabase = async (databaseId) => {
  const response = await notion.databases.query({
    database_id: databaseId,
  });
  return response.results;
};
```

### Index.jsの作成
-----
**90行目から追記**
```js:Index.js
//ISRを追加
export const getStaticProps = async () => {
  const database = await getDatabase(databaseId);
    return {
    props: {
      posts: database,
    },
    revalidate: 1,
  };
};
```

### 実際に動かしてみる
-----
実際にNotionのメモ画面よりテンプレートに基づいた記事の作成ができることを確認する。

![image](/images/zenn-article-01/blog05.png)

Notionのメモ画面で入力した内容が連携先のページのRead postから確認できます。
![image](/images/zenn-article-01/blog06.png)
![image](/images/zenn-article-01/blog07.png)
![image](/images/zenn-article-01/blog08.png)

### 最後に
-----
今回の記事ではNotionを利用して部録の記事の作成管理が簡単にできる機能の実装を行いました。
上記のものは単純な紐づけの部分となっていますが、Notionのメモ機能のカスタマイズ（APIのドキュメントの設定含め）を使えるという事で、
こちらの記事でも拡張性の高さを見ることができたのではないでしょうか。（まだまだ氷山の一角という所感です。）

次回の記事はこちらです。
**[第四回]**
**(https://zenn.dev/knagano/articles/zenn-article-10)**
