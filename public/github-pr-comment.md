---
title: GitHub の PR で編集されていない箇所にレビューコメントしたかった
tags:
  - ''
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

## 概要

GitHub の PR で編集されていない箇所にレビューコメントしたかったのですが、どうしてもできなかったため代替策を記録しておきます。

## 課題

GitHub で コードレビューをしていると、その PR 内で編集されていない箇所にコメントしたいことがあります。ですが PR では編集された近辺の行にしかコメントを追加できません。

![1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/7a6eb31b-da42-d29c-b436-d7469e23f8dc.png)
*コード編集箇所とその近辺では、青い＋マークが表示される*

![2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/79a6779f-a553-9810-903a-9051beaa3105.png)
*＋マークをクリックし、コメントを追加する*

![3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/cb6bcae7-73df-8496-bb91-e4ce73bd73b7.png)
*しかし編集箇所ではないところには＋マークが表示されずコメントできない*

## 対策

### 基本

この場合、PR のコード内に直接コメントをできる方法はありません。そのため以下の代替手順でコメントを残すことにしました。

1. View File を選択してファイルを開く
1. 該当する行を選択して Copy permalink をクリック
1. Conversation にコピーした URL を含めてコメント
1. Request changes にコメントを書いて Submit review をクリック

![4.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/91f84ba6-d89c-4142-3bdf-759c549b448a.png)
*View File を選択してファイルを開く*

![5.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/7869ae3c-d566-b755-13d0-c0059f65a4e1.png)
*該当する行を選択して Copy permalink をクリック*

:::note
複数行を選択するには、先頭行をクリック→末尾行をShift+クリックしてください
:::

![6.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/c3981535-cb67-e1ba-dad1-60a82f6b6425.png)
*Conversation にコピーした URL を含めてコメント*

![7.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/db4ddf21-0f34-45ea-2628-472cd592a10f.png)
*Conversation にコピーした URL を含めてコメント*

![8.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/aea34896-1f3d-8845-439e-b75239c81351.png)
*Request changes にコメントを書いて Submit review をクリック（自作自演 PR のため Request changes にチェックが入れられず）*

### レンダリングされるファイルの場合

Markdown や CSV などレンダリング表示されるファイルの場合、Copy permalink が選択できない場合があります。この場合はコード表示に切り替えることで同様の操作が可能です。

![9.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/1a2eae31-2b23-33f7-784d-2256a016e265.png)
*Markdown では Copy permalink などのメニューが表示されない*

![10.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/f7bc78c0-ce08-d726-a355-4f2aee42b90b.png)
*Code ボタンを押す事でコード表示に*

GitHub Enterprise を使っている場合、Display the source blob という表示の場合があります。

![11.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/6b4ad257-5066-25f3-3089-37acc3a47c9c.png)
*Display the source blob を押せばコード表示に*

## まとめ

どの箇所をどのように直すべきか一目でわかるようにしてあげる事で、レビュアー/レビュイー間の齟齬がなくなり結果的に早く仕事が進みます。良いコードレビューを！
