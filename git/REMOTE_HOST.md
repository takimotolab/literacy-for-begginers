# リモートホスト

## ベアリポジトリ

[やってみる](./TRY.md)章には、「管理者が存在している」という問題点があります。

- パッチを集めてコンフリクトを解消して統合するのが管理者の手間
- 開発者全員の足並みを揃えるためにリポジトリを配布する必要がある
- 管理者がやりたい放題できる
- 管理者がリポジトリを消したら取り返しのつかないことになるかもしれない

問題点を解析すると、次のことが言えそうです。

- 変更の集積
  - 変更の集積は機械的な作業
  - 変更の集積場所を用意すれば解決
- コンフリクトの解消
  - そもそも統合前に変更がコンフリクトを起こさないものであれば良い
  - 変更を行った開発者にコンフリクトの解消をさせれば解決
- 足並み揃え
  - 開発者が現状にアクセスできれば解決
- やりたい放題
  - 管理者のコミットを禁止すれば解決
- リポジトリの誤削除
  - リモート環境にバックアップを取っておけば解決

総合的に考えて、管理者なんて不要であることが言えます。
管理者の代わりに、変更の集積場所がリモート環境にあれば良いですね。
––それが、**ベアリポジトリ**です。

ベアリポジトリは普通のリポジトリと殆ど同じですが、次の特徴を持ちます。

- `.git`ディレクトリしか持たない
  - 単独でコミットを作ることができない
  - 変更を集積するだけ
- SSH通信等でアクセスできる
  - 誰でもいつでも変更を送れる・同期を取れる
  - しかも、ファイルで送る必要がない



## リモートホスティングサービス

ベアリポジトリを作ってくれるサービスを**リモートホスティングサービス**と言います。
この世には沢山のリモートホスティングサービスがありますが、本記事で扱うGitHubはそのうちの一つです。

少なくともGitHubは、ベアリポジトリを作ってくれるだけでなく、ベアリポジトリを容易に扱うためのGUIを提供する他、イシュー、プルリクエスト、CI、静的サイトホスティング等々の開発や運営に役立つサービスを提供しています。

**GitHubの始め方は説明しません。各自調べてください。**



## やってみる

では、GitHubで(ベア)リポジトリを作ったことを前提に、開発環境を刷新してみましょう。
まず、GitHub上で空のリポジトリを作ります。
GitHub上で作ったリポジトリには、必ずアドレスが与えられます。
今回は次のアドレスだったとします。

```
git@github.com:username/sample-repo.git
```

ローカルで空のリポジトリを作ります。
そして、`git remote add`を実行してリモートのリポジトリを結びつけます。
ただし、`git remote add A B`は、`A`を`B`という名前で追加する、という意味です。

```shellsession
$ pwd
/.../documents/sample-repo
$ git init
$ git remote add origin git@github.com:username/sample-repo.git
```

何か変更を行い、コミットします。
このコミットをリモートに反映させるには、`git push`を実行します。
`-u origin HEAD`は初回に必要なもので、次回以降不要になります。
(厳密には違います。詳しくは[ブランチ](./BRANCH.md)で説明します)

```shellsession
$ git commit -m "commit message"
$ git push -u origin HEAD
```

では、新しく開発者を参加させます。
`git clone`によりリポジトリをリモートから複製して持ってこられます。

```shellsession
$ pwd
/.../documents
$ git clone git@github.com:username/sample-repo.git
$ ls sample-repo
sample-repo
```

誰かがリモートに変更を加えたとします。
開発者は足並みを揃えるために、`git pull`を実行することで変更を取り込むことができます。

```shellsession
$ git pull
```



## 歴史は正しく並べ

実は、現状の知識ではコンフリクトが起きえません。
なぜなら、リモートのリポジトリにおける「パッチの当て方」を知らないからです。
`git push`で行っているのは、コミットの反映でしかありません。

つまり、実質的に、一人で開発をしている状態なのです。
Aさんが編集し、Bさんが編集し、Cさんが編集したという状況は、自宅で編集し、研究室で編集し、旅行先で編集したという状況と何も変わりないのですから。
そして、一人で開発しているということは、**差分の記録の順番がおかしくあってはいけません**。

では、わざとおかしくしてみましょう。
リモートのリポジトリのコミットログが次のようだとします。

```
461779c commit1
```

Aさんがコミットをpushします。

```shellsession
$ git log --oneline
461779c (HEAD -> master, origin/master) commit1

$ git commit -m "commit2"

$ git log --oneline
698f65c (HEAD -> master, origin/master) commit2
461779c commit1

$ git push
```

Bさんがpullを忘れてコミットをpushします。

```shellsession
$ git log --oneline
461779c (HEAD -> master, origin/master) commit1

$ git commit -m "commit3"

$ git log --oneline
68bfe02 (HEAD -> master, origin/master) commit3
461779c commit1

$ git push
To github.com:username/sample-repo.git
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'github.com:username/sample-repo.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. If you want to integrate the remote changes,
hint: use 'git pull' before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

エラーが出てpushできませんでした。
このように、歴史は正しく並んでいなければならないのです。

(ちなみに、`git push -f`とすると有無を言わさず歴史を上書きできます。force pushと言います。使うべき状況もありますが、そうでない状況では非常に危ない行為なので、なるべく使わないようにしましょう)
