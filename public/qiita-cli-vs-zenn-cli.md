---
title: ここがツラいよQiita CLI ～Zenn CLIとの比較～
tags:
  - 'qiita'
  - 'github'
  - 'zenn'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: true
---

## はじめに

私は GitHub を利用して記事を投稿しています。Qiita CLI と Zenn CLI を両方使ってみて感じた Qiita のツラさを書いていきます。

## 画像の貼り付けが大変

Qiita の記事は画像を挿入するのが大変です！GitHub 連携をしている身からすると、Zenn はすべてエディタの中で完結するため理想的です。Qiita の場合はブラウザでの操作が必要で、二重管理のようになってしまうのも気になります。

### Qiita CLI の場合

```md
1. images/<article_title> フォルダに画像を配置する
2. [ファイルのアップロード](https://qiita.com/settings/uploading_images) を開く
3. ファイルをアップロードし「画像URLをコピー」する
4. ![description](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/xxx.png) の形式で参照する
```

### Zenn CLI の場合

```md
1. images/<article_title> フォルダに画像を配置する
2. ![description](/images/<article_title>/xxx.png) の形式で参照する
```

## 下書き一覧に表示されない

Markdown で記事の下書きを書いているとします。Qiita も Zenn も CLI でプレビューが可能ですが、実際のところ見た目は公開記事と若干異なります。具体的には目次やサイドバーの有無、ページ幅の誤差による改行の位置ズレ、ダークモード対応の有無などです。

これら理由より CLI のプレビューだけでなく、下書きページから実際のページを確認したいです。ところが Qiita の場合、CLI で記事を執筆し `ignorePublish: true` を指定している状態だと「下書き一覧」に表示されません。

![qiita_draft.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/34db136c-a4fe-bf16-a346-5cb5b994d6f4.png)

一方 Zenn の場合、`published: false` でも「記事一覧」に表示されるため、公開せずに実際の見た目の確認ができます。

![zenn_draft.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/662862dc-842c-5add-fe30-abbcc2db8c99.png)

## GitHub のデータが御本尊とは限らない

Qiita CLI では GitHub のデータが御本尊とは限りません。そのため記事ファイルを消しても公開されている記事は消されませんし、Qiita 上で記事を更新してしまうと Markdown のデータと差分が生まれる可能性があります（これ自体は `npx qitta pull` コマンドで[解消できます](https://github.com/increments/qiita-cli?tab=readme-ov-file#pull)）。

> 記事ファイルを Qiita と同期します。
Qiita 上で更新を行い、手元で変更を行っていない記事ファイルのみ同期されます。

一方 Zenn CLI では必ず GitHub のデータが御本尊 となります。仮に Zenn 上で記事を更新しても、次にデプロイが走れば記事データの通り巻き戻ります。逆に Qiita はウェブから更新する自由があるとも言えますが、GitHub を介したデプロイが強制される方が正しいのかなと考えています。

![zenn_warning.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/7dc4e9c2-16d8-093c-5e7f-c7fece8777a7.png)

## GitHub Actions の利用が必要

Qiita で GitHub 連携を選択した場合、GitHub Actions を利用したデプロイになります。このため GitHub Actions の利用枠を使うことに注意が必要です。

![qiita_connect.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/e46eff4d-25b0-921f-0f47-9411f16e7510.png)

Zenn の場合は Zenn Connect という GitHub App による連携となります。このため設定もより簡単（Install & Authorize ボタンを押すだけ）で、GitHub Actions の利用枠を使うことはありません。

![zenn_connect.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/e1fa7129-6b0a-82e8-8363-a633b60ba456.png)

## まとめ

Qiita CLI のツラい点を中心に書いてしまいましたが、ブラウザで書くのと違いリポジトリで記事が管理ができるという点は最高です！あなたの資産を保全するためにも、Qiita CLI で執筆、GitHub で記事管理してみませんか？
