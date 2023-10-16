Blog Zenn
===

### Zenn URL
https://zenn.dev/takayoshi <br />
※ マイページへのリンクです

### 📘 How to use
https://zenn.dev/zenn/articles/zenn-cli-guide

#### 基本操作

- プレビュー画面の表示

```shell
npx zenn preview

# port指定したいときは
npx zenn preview --port {ポート番号}
```

- 新規空記事の作成

```shell
npx zenn new:article

# 指定したければ
# slug はa-z0-9、ハイフン-、アンダースコア_の 12〜50 字の組み合わせにする必要がある
npx zenn new:article --slug 記事のスラッグ --title タイトル --type idea --emoji ✨
```

- 記事の削除
  - Zennのダッシュボー上で記事の削除を行う
