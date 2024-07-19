# ブランチ

## 意義

共同開発をするということは、ある時点をベースに各開発者が各々にパッチを作るということです。
パッチを提出するとき、他のパッチと同期を取るべき理由はありません。

そこで、**ブランチ**の概念が登場します。
ブランチとは、世界線のようなものです。
それぞれに歴史があり、交叉したり消滅したりします。

Gitでは普通、**ブランチ**を主ブランチから分けて、そこでコミットを溜め、主ブランチに**マージ**することでパッチ当てを行います。



## やってみる

`git branch`を実行することでブランチの一覧を取得できます。
`*`が付いているブランチが現在編集中のブランチです。

```shellsession
$ git branch
* master
```

`git branch ブランチ名`を実行することで新しくブランチを作成できます。
`git switch ブランチ名`を実行することでブランチを移動できます。

```shellsession
$ git branch foo

$ git branch
  foo
* master

$ git switch foo
Switched to branch 'foo'

$ git branch
* foo
  master
```

ブランチを移動したら普段通りにコミットを記録していきます。
`foo`上でのコミットは、`master`には反映されません。

```shellsession
$ git commit -m "commit2"

$ git log --oneline
698f65c (HEAD -> foo) commit2
461779c commit1

$ git switch master

$ git log --oneline
461779c (HEAD -> master, origin/master) commit1
```

ところで、前章では`git push`をコミットの反映と説明しましたが、正確には「あるブランチのコミットの反映」です。
`git push A B`は、リモートリポジトリ`A`に`B`ブランチのコミットを反映するという意味です。
つまり、次を実行することで`foo`ブランチをリモートに反映できます。

```shellsession
$ git push origin foo
```

いちいち`origin foo`や`origin master`と書くのが煩わしい場合は次の条件で省略できます。

- 現在いるブランチと同じ名前のブランチがリモートにもある (`-u`オプションをつけると条件を消せる)
- 現在いるブランチを現在いるブランチと同じ名前のリモート上のブランチに反映する

また、前章では`git pull`をリモートの変更の取込みと説明しましたが、正確には「あるリモートのブランチから現在いるブランチへのマージ」です。
つまり、次を実行することでリモート上の`foo`ブランチを現在いるブランチにマージできます。
このとき、現在いるブランチが`foo`でない場合や、リモートとは異なる歴史を持つ場合は、パッチ当てのような状況になります。

```shellsection
$ git pull origin foo
```

`git push`と同様に、`origin foo`の部分を次の条件で省略できます。

- 現在いるブランチと同じ名前のブランチがリモートにもある
- 現在いるブランチと同じ名前のリモート上のブランチから現在いるブランチへのマージを行う

`foo`ブランチが熟成し、`master`ブランチに取り込みたいときは、`git merge`でマージを行えます。
`git merge A`は、現在いるブランチに`A`ブランチを取り込む、という意味です。
ただし、GitHubを使っている場合は、次節で説明するプルリクエストによってマージを行えるので読み流して構いません。

```shellsession
$ git merge foo
$ git log --oneline
4b13192 (HEAD -> master) Merge branch 'foo'
698f65c (foo) commit2
461779c (origin/master) commit1
```



## プルリクエスト

**プルリクエスト**(プルリク、PRとも略される)とは、あるブランチからブランチへのマージ依頼です。
~~なんでマージリクエストじゃないんだよ。~~

プルリクエストこそがパッチに相当するものであり、基本的にはプルリクエストを作成・マージしていくことによってプロジェクトを育てていくべきでしょう。

マージすることでコンフリクトが発生する場合、マージできないようになっています。
つまり、マージ以前にコンフリクトを解消しなければなりません。
では、わざとコンフリクトを起こすプルリクエストを作ってみましょう。

リモート上の`master`ブランチに次のファイルがあるとします。

```c
#include <stdio.h>
int main() {
    printf("Current.\n");
    return 0;
}
```

```
23743ba commit1
```

**この時点の**`master`ブランチをベースに`bar`ブランチを作り、次のように編集し、プルリクエストを作成します。

```c
#include <stdio.h>
int main() {
    printf("Updated on bar.\n");
    return 0;
}
```

```
7b17042 updated on bar
23743ba commit1
```

しかし、プルリクエストがマージされる前に`master`ブランチに変更があり、次のようになったとします。

```c
#include <stdio.h>
int main() {
    printf("Hello, world!\n");
    return 0;
}
```

```
949112b hello world
23743ba commit1
```

ここで、プルリクエストはマージできなくなります。
なぜなら、コンフリクトが発生するためです。
`bar`上でコンフリクトを解消しなければなりません。
解消手段は幾つかありますが、最も簡単な方法を紹介します。
それは、`master`を`bar`にマージしてしまうことです。

```shellsession
$ git branch
* bar
  master

$ git pull origin master
From github.com:user/repo-name
 * branch            master     -> FETCH_HEAD
Auto-merging file.name
CONFLICT (content): Merge conflict in file.name
Automatic merge failed; fix conflicts and then commit the result.
```

コンフリクトが発生するので、コンフリクトマーカを頼りにコンフリクト箇所を修正します。
その後、修正をコミット、pushします。

```shellsession
$ git commit -m "fix conflict"
$ git push
$ git log --oneline
c906bb4 (HEAD -> bar, origin/bar) fix conflict
949112b (origin/master) hello world
7b17042 updated on bar
23743ba (master) commit1
```

これにより、コンフリクトが解消され、マージできるようになりました。
