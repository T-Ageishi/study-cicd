# study-cicd

## コンテキスト

- コンテキスト（Contexts）は実行時の情報や、ジョブの実行結果などを保持するオブジェクト。複数のプロパティで構成され、各プロパティから値を取得できる。
- コンテキストにはいくつかの種類が存在する。
  - github
  - job
  - runner
  - steps...
- コンテキストから値を取り出すには `${{ github.actor }}` のように記述する。

## 環境変数

- 環境変数をenvキーで定義する。
- ワークフロー・ジョブ・ステップの各レベルで定義可能。定義場所によって環境変数のスコープは変わる。
- 環境変数には文字列だけでなく、コンテキストも指定できる。

## シークレット

- Settings>Secrets and Variablesで登録したシークレットをワークフロー内で参照できる
- シークレットは基本的にログマスクされるが、1文字変えるだけで回避できてしまう。そのため、そもそもSecretsはログ出力しないことが大切。

## ステップ間のデータ共有

### ステップ間でデータは引き継がれない

- 1ステップ目で環境変数を定義しても、2つ目のステップでは未定義状態に戻ってしまう。

```yml
name: Missing share data
on: push
jobs:
  share:
    runs-on: ubuntu-latest
    steps:
      - run: export RESULT="hello"
      - run: echo "${RESULT}"
```

### GITHUB_OUTPUT環境変数

- 受け渡し側ステップは、GITHUB_OUTPUT環境変数へ、キーバリュー形式の文字列を書き出す。また、idキーでステップIDを定義する。

```
- id: <step-id>
  run: echo "<key>=<value>" >> "${GITHUB_OUTPUT}"
```

- 受け取り側ステップは、次のようにstepsコンテキストを参照する。

```
${{ steps.<step-id>.outputs.<key> }}
```

### GITHUB_ENV環境変数

- グローバル変数的に環境変数を管理できる。
- ワークフローが大きくなるとバグの温床になりやすいため、基本的にはGITHUB_OUTPUT環境変数を利用する。

## GitHub API

- GitHubはプログラムからアクセスできるGitHub APIと呼ばれるAPIを公開している。
