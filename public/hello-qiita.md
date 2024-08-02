---
title: Qiitaに入門してみた！～GitHubリポジトリ連携編～
tags:
  - Qiita
  - GitHub
private: false
updated_at: '2024-08-03T02:48:28+09:00'
id: db8d7f984d7d056c46f8
organization_url_name: null
slide: false
ignorePublish: false
---
## Qiita に入門してみた

以下記事を参考に、Qiita に入門してみました！

https://qiita.com/Qiita/items/666e190490d0af90a92b

https://qiita.com/Qiita/items/32c79014509987541130

## 記事投稿までの流れ

詳しい手順は上記公式ドキュメントを参考にしてください。およそ 15 分で Qiita に入門できました！

| 項番 | 内容                                      | 所要時間 |
| ---- | ----------------------------------------- | -------- |
| 1    | GitHub でリポジトリ作成                   | 1 min    |
| 2    | Qiita CLI のインストール                  | 2 min    |
| 3    | Qiita トークンの発行                      | 1 min    |
| 4    | GitHub Actions の Secret にトークンを設定 | 2 min    |
| 5    | 適当な記事を作成                          | 5 min    |
| 6    | 変更をコミット、GitHub へプッシュ         | 3 min    |
| 7    | Qiita へのデプロイを確認                  | 1 min |

## ポイント

**GitHub Actions による連携を選択**することにより、どこからでも簡単に記事を投稿できます！GitHub Actions 用のファイルは `npx qiita init` コマンドで自動作成してくれるので、自身での設定は不要です！

![github_actions.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/8187b6b6-822a-c617-7560-44014d984839.png)

### Qiita CLI を使う場合の記事投稿例

```bash
# Qiita CLI による記事のデプロイ
npx qiita login
npx qiita new ${記事ファイル}
npx qiita publish ${記事ファイル}
```

### Qiita CLI でのデプロイを選択しなかった理由

- 環境や端末ごとに `npx qiita login` する必要があり、Qiita トークンの管理が煩雑
- Git 管理しているのにコミットやプッシュと別に `npx qiita publish` するのが面倒
