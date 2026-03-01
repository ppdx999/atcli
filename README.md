# atcli

AtCoder用のシェルスクリプト製の小さなコマンドラインツール。

## 準備

### セッションの保存

AtCoderのセッションを保存するには、ブラウザの開発者ツールを使用して、`Cookie`の`REVEL_SESSION`を環境変数`ATCODER_SESSION`に設定します。

```bash
# .bashrcや.zshrcなどに追加
export ATCODER_SESSION="your_session_value"
```

## サブコマンド一覧

### problem ls

コンテストIDを指定して、そのコンテストの問題IDを一覧表示する。

```bash
$ atcli problem ls abc300
abc300_a
abc300_b
abc300_c
abc300_d
abc300_e
abc300_f
abc300_g
abc300_h
```

### testcase fetch

コンテストIDと問題IDを指定して、その問題のテストケースを区切り文字形式で出力する。

```bash
$ atcli testcase fetch abc300 abc300_a
---INPUT---
1 2
---OUTPUT---
3
---INPUT---
10 20
---OUTPUT---
30
```

**出力形式**:
- `---INPUT---` で入力の開始を示す
- `---OUTPUT---` で出力の開始を示す
- 入力と出力のペアが順番に続く

### testcase run

`testcase fetch`で取得した区切り文字形式のテストケースを読み込み、指定したコマンドでテストを実行する。

```bash
$ atcli testcase fetch abc300 abc300_a | atcli testcase run ./a.out
Test 1: Passed
Test 2: Passed
```

失敗時は差分を表示:

```bash
$ atcli testcase fetch abc300 abc300_a | atcli testcase run ./a.out
Test 1: Failed
  Input:
    1 2
  Expected:
    3
  Actual:
    4
Test 2: Passed
```


## 便利な使い方

`.bashrc` or `.zshrc`に以下の内容を記載

```bash
export ATCODER_SESSION="your_session_value"
atcli_i() {
    contest_id=$1
    mkdir -p "$contest_id"
    atcli problem ls "$contest_id" | while read -r problem_id; do
        mkdir -p "$contest_id/$problem_id"
        echo "作成: $contest_id/$problem_id/"
        cp template.c "$contest_id/$problem_id/main.c"
        echo "作成: $contest_id/$problem_id/main.c"
    done
}
atcli_d() {
    contest_id=$(basename "$(dirname "$(pwd)")")
    problem_id=$(basename "$(pwd)")
    echo "$contest_id/$problem_id のテストケースを取得しています..."
    atcli testcase fetch "$contest_id" "$problem_id" > testcases.txt
    echo  "完了"
}
atcli_t() {
    if [ ! -f testcases.txt ]; then
        echo "testcases.txtが見つかりません。テストケースを取得します..."
        atcli_d
    fi
    echo "main.cをコンパイルしています..."
    if ! gcc -o main main.c; then
        echo "コンパイルエラー"
        return 1
    fi
    echo "完了"
    echo "テストケースを実行します..."
    atcli testcase run "./main" < testcases.txt
    echo "完了"
    echo "クリーンアップしています..."
    rm -f main
    echo "完了"
}
atcli_s() {
    echo "クリップボードにmain.cをコピーしました"
    cat main.c | pbcopy
    contest_id=$(basename "$(dirname "$(pwd)")")
    problem_id=$(basename "$(pwd)")
    echo "提出URL"
    echo "https://atcoder.jp/contests/$contest_id/submit?taskScreenName=$problem_id"
}
```

次のように使います(ABC300 A問題を解く場合):

```bash
$ atcli_i abc300
作成: abc300/abc300_a/
作成: abc300/abc300_a/main.c
作成: abc300/abc300_b/
作成: abc300/abc300_b/main.c
...

$ cd abc300/abc300_a
$ nvim main.c # Write code here
$ atcli_t
testcases.txtが見つかりません。テストケースを取得します...
abc300/abc300_a のテストケースを取得しています...
完了
main.cをコンパイルしています...
完了
テストケースを実行します...
Test 1: Passed
Test 2: Passed
$ atcli_s
クリップボードにmain.cをコピーしました
提出URL
https://atcoder.jp/contests/abc300/submit?taskScreenName=abc300_a
```
