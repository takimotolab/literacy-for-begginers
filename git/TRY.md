# やってみる

Gitのインストール方法は書きません。
各自調べてください。



## リポジトリ作成

適当な場所に作業ディレクトリを作りましょう。
今回の例では、次の`git-learning`を作ります。

```
~/documents/git-learning
```

作業ディレクトリ上で`git init`を実行します。
これにより、作業ディレクトリ以下のファイル・ディレクトリがGit管理対象となります。
この状態の作業ディレクトリ・それ以下ファイル・ディレクトリを**リポジトリ**と言います。
また、`.git`ディレクトリが生成されるはずです。
Gitのすべて作業情報がこのディレクトリに保存されます。

```shellsession
$ git init
Initialized empty Git repository in /.../documents/git-learning/.git/
```



## コミット

では、ファイルを作りましょう。
次の`main.c`を作ります。

```c
#include <stdio.h>
int main() {
    return 0;
}
```

ファイルを作ったという**差分**(変更のこと)を記録しましょう。
差分の記録には、(1)記録対象を指定、(2)記録の手順を踏みます。
それぞれ、`git add`と`git commit`を実行します。
ちなみに、`git add`にはファイル名・ディレクトリ名を与えますが、`-A`オプションで指定すると、リポジトリ全体を記録対象にできます。
わざわざ記録対象を分ける必要のない限り`git add -A`を使うと良いでしょう。

```shellsession
$ git add -A

$ git commit -m "add main.c"
[master (root-commit) 9e1ced3] add main.c
 1 file changed, 4 insertions(+)
 create mode 100644 main.c
```

さらに`main.c`を編集して記録しましょう。

```c
#include <stdio.h>
int main() {
    printf("Hello, world!\n");
    return 0;
}
```

```shellsession
$ git add -A

$ git commit -m "add hello world"
[master 76ae5f0] add hello world
 1 file changed, 1 insertion(+)
```

これまでどのような記録がされているかを`git log`を実行することで確認できます。
記録されている差分・あるいは差分を記録することを**コミット**と呼びます。

```shellsession
$ git log
commit 76ae5f087724b9f0d0fc7a18abc467f122ad7b49 (HEAD -> master)
Author: username <email>
Date:   Fri Jul 19 17:43:03 2024 +0900

    add hello world

commit 9e1ced37f5de42f9c4b7bd270a30747d2a500de8
Author: username <email>
Date:   Fri Jul 19 17:34:38 2024 +0900

    add main.c
```



## パッチ

**この節に書かれていることは一切覚えなくていいです**。

さて、Gitの本領である共同開発を体験してみましょう。
まず、別人にソースコードの複製を送ります。
ここでは、作業ディレクトリを`git-learning-copy`として複製することで、別人に送ったとみなします。

`main.c`を編集して差分を記録します。

```c
#include <stdio.h>
int main() {
    printf("Hello, world!\n");
    printf("I'm learning Git.\n");
    return 0;
}
```

```shellsession
$ pwd
/.../documents/git-learning-copy

$ git add -A

$ git commit -m "add a message"
[master b940b2c] add a message
 1 file changed, 1 insertion(+)

$ git log --oneline
b940b2c (HEAD -> master) add a message
76ae5f0 add hello world
9e1ced3 add main.c
```

Gitを使っていれば、わざわざソースコードを渡す必要はありません。
代わりに、**差分を送ります**。
この差分をパッチとして適用することで、変更が反映されるのです。
パッチを作るには、`git format-patch`を実行します。
今回はパッチファイルとして`0001-add-a-message.patch`が生成されます。

```shellsession
$ git format-patch HEAD^
0001-add-a-message.patch
```

このパッチファイルを送ります。
そして、パッチファイルを貰ったら、`git am`を実行して適用します。
これで、コミットが追加され、`main.c`に変更が反映されました。

```shellsession
$ pwd
/.../documents/git-learning

$ git log --oneline
76ae5f0 (HEAD -> master) add hello world
9e1ced3 add main.c

$ git am -3 0001-add-a-message.patch
Applying: add a message

$ git log --oneline
eae8274 (HEAD -> master) add a message
76ae5f0 add hello world
9e1ced3 add main.c

$ cat main.c
#include <stdio.h>
int main() {
    printf("Hello, world!\n");
    printf("I'm learning Git.\n");
    return 0;
}
```



## コンフリクト

Gitを使う利点はコンフリクトを上手く対処できることだと説明しました。
では、実際にコンフリクトを起こしてみましょう。
今回は、前節の`main.c`の4行目でコンフリクトを起こします。

`git-learning`側で変更を加えます。

```c
#include <stdio.h>
int main() {
    printf("Hello, world!\n");
    printf("The message is changed by A.\n");
    return 0;
}
```

```shellsession
$ git add -A
$ git commit -m "update line 4 by A"
```

`git-learning-copy`側でも変更を加えます。
`git-learning`と同じ箇所(4行目)を変更し、2行目の下に新しい行を追加しています。
そして、こちら側でパッチを作ります。

```c
#include <stdio.h>
int main() {
    printf("The message is written by B.\n");
    printf("Hello, world!\n");
    printf("The message is changed by B.\n");
    return 0;
}
```

```shellsession
$ git add -A
$ git commit -m "update line 4 by B"
$ git format-patch HEAD^
```

生成されたパッチファイルを`git-learning`側に送り、適用します。

```shellsession
$ git am -3 0001-update-line-4-by-B.patch
Applying: update line 4 by B
Using index info to reconstruct a base tree...
M       main.c
Falling back to patching base and 3-way merge...
Auto-merging main.c
CONFLICT (content): Merge conflict in main.c
error: Failed to merge in the changes.
hint: Use 'git am --show-current-patch=diff' to see the failed patch
Patch failed at 0001 update line 4 by B
When you have resolved this problem, run "git am --continue".
If you prefer to skip this patch, run "git am --skip" instead.
To restore the original branch and stop patching, run "git am --abort".
```

パッチが適用できませんでした。
なぜなら、同じ4行目が変更されている所為で、コンフリクトが発生したからです。
これにより、変更の統合によるバグの混入を防ぐことができました。

コンフリクトが発生すると、コンフリクト発生位置にコンフリクトマーカが挿入されます。
このマーカを頼りに適切にファイルを編集することで、パッチを適用できます。

```c
#include <stdio.h>
int main() {
    printf("The message is written by B.\n");
    printf("Hello, world!\n");
<<<<<<< HEAD
    printf("The message is changed by A.\n");
=======
    printf("The message is changed by B.\n");
>>>>>>> update line 4 by B
    return 0;
}

```

今回は、`git-learning`側の変更を優先することにします。
次のようにファイルを編集し、コンフリクトを解消します。

```c
#include <stdio.h>
int main() {
    printf("The message is written by B.\n");
    printf("Hello, world!\n");
    printf("The message is changed by A.\n");
    return 0;
}
```

```shellsession
$ git add main.c
$ git am --continue
Applying: update line 4 by B
$ git log --oneline
951db02 (HEAD -> master) update line 4 by B
815bdb5 update line 4 by A
eae8274 add a message
76ae5f0 add hello world
9e1ced3 add main.c
```

無事にコンフリクトを解消し、パッチを適用できました。
