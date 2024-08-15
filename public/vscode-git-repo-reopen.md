---
title: VSCode で Git リポジトリが表示されない場合の対処
tags:
  - Git
  - VSCode
private: false
updated_at: '2024-08-15T16:10:22+09:00'
id: c51ac7bdc6f1db239aa9
organization_url_name: null
slide: false
ignorePublish: false
---

## リポジトリが表示されない

VSCode で Git 管理されているフォルダやファイルを開いているのに、ソース管理タブでリポジトリが表示されず時間を使ってしまったのでメモを残します。

以下の例では `qiita-contents` フォルダを開いているのに、ソース管理タブに表示されていません。

![explorer.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/d29dd7cd-f7c3-5583-c176-18bca8e2539d.png)
![git.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/a1b28796-d13f-9e9d-41f1-eb3b702661a6.png)

## 対処方法

`Ctrl + Shift + P` でコマンドパレットを呼び出し、`Reopen Closed Repositories` を選択すれば OK です。

![reopen1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/bac38d00-56b6-ce43-87ec-d7e1f26b9116.png)
![reopen2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/6fbc2e4d-5a51-a869-40d1-ed3740f1c252.png)

## コメント

VSCode が Git のリポジトリを認識しないと思っていましたが、どうやら邪魔になったとき「リポジトリを閉じる」をクリックしてしまったようです。
