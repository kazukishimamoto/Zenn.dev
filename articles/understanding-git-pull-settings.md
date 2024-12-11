---
title: 'ん？俺のgitconfig、rebase'
emoji: '🙈'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: [git, github]
published: false
---

## はじめに

ふと .gitconfig を眺めていると、以下のような設定がありました。

```text
[pull]
  rebase = true
  ff = only
```

あれ、この設定ってなんだっけ？なんか軽くググってみたら、この 2 つって矛盾してるんじゃね…🙈
ということで、この設定を理解した上で自分の意図したものにするべく、復習がてら merge や rebase、pull の設定について調べることにしました。

---

## pull とは

git pull とは `fetch + merge or rebase` を行うコマンドです。
今回テーマとなった pull の設定は、fetch 後に merge か rebase のどっちにするかであったり、merge のオプションを指定できるものです。
そこで、merge, rebase についての基本から確認していきます。

## merge について

merge には主に 2 つの種類があります。

### Fast Forward マージ

Fast Forward（早送り）マージは、ブランチのポインタを先に進めるだけの簡単な操作です。
以下のような状況を例に見てみましょう：

```bash
❯ g log
commit 269cafb9656a41fbc61c65e29b745a2d2a48187e (HEAD -> feature/add-hige)
Author: Kazuki Shimamoto <jakky0529@gmail.com>
Date:   Mon Dec 9 23:21:29 2024 +0900

    add left and right hige

commit 499aef7db7bdac177a3923bbf7e3b621688353fd
Author: Kazuki Shimamoto <jakky0529@gmail.com>
Date:   Mon Dec 9 23:20:00 2024 +0900

    add hige

commit 40aa8f4eaddc7549151f3faeab5745c1dca07a4b (main)
Author: Kazuki Shimamoto <jakky0529@gmail.com>
Date:   Mon Dec 9 23:15:27 2024 +0900

    first commit
```

main ブランチが first commit を指しており、そこから feature/add-hige ブランチで 2 つのコミットが追加されている状況です。

ここで git merge feature/add-hige を実行すると：

```bash
❯ g merge --ff feature/add-hige
Updating 40aa8f4..269cafb
Fast-forward
 neko.html | 3 +++
 1 file changed, 3 insertions(+)
```

main ブランチのポインタが単純に feature/add-hige の先端まで進みます。
このポインタが先に進められるだけのマージを Fast Foward マージといいます。

### Auto Merge

一方、両方のブランチで変更が行われている場合は、マージコミットを生成して変更を取り込む必要があります。
例えば、main ブランチと feature/mouth ブランチで異なる変更が行われている場合：

```bash
❯ git merge feature/mouth
Auto-merging neko.html
Merge made by the 'ort' strategy.
 neko.html | 2 ++
 1 file changed, 2 insertions(+)
```

この場合、2 つの親を持つマージコミットが生成されます：

```bash
commit 2113fa098b65ee5f437643ba6a002d9cc3c11ec5 (HEAD -> main)
Merge: 8625103 216934f
Author: Kazuki Shimamoto <jakky0529@gmail.com>
Date:   Mon Dec 9 23:54:28 2024 +0900

    Merge branch 'feature/mouth'
```

:::message
git log って CLI の表現上、1 直線に履歴が表示されるけど、merge しても分岐はそのまま。
だから合流させるためのマージコミットは親が 2 つになる。`8625103 216934f`
:::

## rebase の理解

### 基本的な動作

rebase はコミットの付け替え操作です。マージコミットを作らずに変更を統合できるため、履歴が見やすくなるという利点があります。
例えば、main ブランチと feature/ear ブランチがあり、それぞれに変更がある場合：

```bash
❯ g rebase feature/ear
Successfully rebased and updated refs/heads/main.
```

rebase を実行すると、main ブランチのコミットが feature/ear ブランチの先端に付け替えられます。

### 注意点

rebase には重要な注意点があります：

1. コミット履歴が書き換えられる
   - rebase 前後でコミットハッシュが変化
   - リモートブランチとの差分が生まれる可能性
2. push -f との組み合わせに要注意
   - 他の開発者のローカルブランチと差分が生まれる
   - チーム開発に支障をきたす可能性

## 設定の検証

ここまでくれば、設定について考える知識は十分です。

### pull.rebase と pull.ff の関係

最初に疑問に思った設定を検証してみましょう：

```text
[pull]
rebase = true
ff = only
```

ある解説では「`rebase = true` を設定すると Fast Forward できない場合は rebase してくれる」という記載も目にしましたが、実際に検証したところ異なる動作をしました：

```bash
fatal: Not possible to fast-forward, aborting.
```

ff = only が優先され、Fast Forward できない場合はエラーとなることがわかりました。
矛盾した設定の .gitconfig を使い続けていたんですね 😇

## 実践的な運用方針

### main ブランチでの考え方

典型的な開発フローでは：

- main ブランチは直接 push 不可
- PR のマージを通じてのみ更新される

この場合：

- ff = only の設定で十分
- Fast Forward できない状況は基本的に発生しない

### feature ブランチでの考え方

複数人で開発する feature ブランチでは、Fast Forward できないケースが発生します。
その場合の選択肢は：

1. merge を実行してマージコミットを作成
2. rebase で自分のコミットを付け替え

これらは状況に応じて明示的に選択するのが望ましいです。

### 推奨設定

現時点での推奨設定は：

```text
[pull]
ff = only
理由：
```

- 意図しない rebase を防げる
- Fast Forward できない場合は明示的にエラーとなり、適切な対処を選択できる
- 必要に応じて手動で rebase するアプローチが安全

## まとめ

- merge には Fast Forward と Auto Merge の 2 種類がある
- rebase は履歴をクリーンに保てるが、使用には注意が必要
- pull.ff = only の設定で、意図しない merge/rebase を防ぐことができる
- 複雑な状況では、手動での merge/rebase の選択が望ましい

Git の設定は、チームの開発スタイルや規模によって最適解が変わります。まずは安全な設定から始めて、必要に応じて調整していくのがよいでしょう。
