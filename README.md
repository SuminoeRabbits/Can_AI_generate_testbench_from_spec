# Can_AI_generate_testbench_from_spec
## PROFACE
Can AI generate testbench just from engineering specification? 

Key specifications are only in, 
- Testbench spec is described in `docs/specification.md`
- Test scenario is described in `docs/testscenario.md`
- DUT(Device Under Test) is described in `docs/DUT.md`

AI will implement codes for these key spec by GitHub Actions.

## TOC

## Learning with CoPilot
面倒なので全部AIにやらせてみよう。まずやり方がよくわからなかったので聞いてみた。

`GitHub ActionsやAPI連携で自動化 の部分を詳しく教えて。今、GithubでRepositoryを作りました。そして docs/specification.md が存在するとします。このファイルをCheckInするたびにGithub　ActionでAIに対して実装を行うように指示したい。どうやって？`


## Setting GitHub Actions

### Setting AI backend

例：OpenAIなら https://api.openai.com/v1/chat/completions
APIキーを取得して、GitHubのSecretsに保存 → Settings > Secrets and variables > Actions にて OPENAI_API_KEY を追加

### Github Actions flow for generation

Adding in `.github/workflows/generate_code.yml`.

```
name: Generate Code from Spec

on:
  push:
    paths:
      - 'docs/specification.md'  # このファイルが更新されたときにトリガー

jobs:
  generate_code:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Read specification
        id: read_spec
        run: |
          SPEC=$(cat docs/specification.md | jq -Rs .)
          echo "spec=$SPEC" >> $GITHUB_OUTPUT

      - name: Call OpenAI API
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
        run: |
          curl https://api.openai.com/v1/chat/completions \
            -H "Content-Type: application/json" \
            -H "Authorization: Bearer $OPENAI_API_KEY" \
            -d '{
              "model": "gpt-4",
              "messages": [
                {"role": "system", "content": "You are a helpful assistant that writes code based on engineering specifications."},
                {"role": "user", "content": '${{ steps.read_spec.outputs.spec }}'}
              ]
            }'
```


### Github Actions flow for auto check-in(optional)

## docs/specification.md

### initinal geneneration
これも面倒なのでAIに書かせる。

`最終的にはAIに実装を依頼することを想定したdocs/specification.md のサンプルを考えてみたい。これは仕様書なので、具体的な実装は行わないがこの仕様書を読んで実装が可能であるレベルの具体的な仕様書を作る事がっ目的である。AIが解析可能なMarkdownフォーマットで記述をお願いしいます。以下は仕様書です。
言語使用はSystem Verilgとし、階層のTopはtop_tb()と命名。top_tb()内には下記のModuleが存在する。 reset_gen(), clk_gen(), dut_top(), memory_target_if(), dut_initiator_if0(),dut_initiator_if1()。
- reset_gen()とclk_gen()はリセット、クロックを与えるだけのModuleであり、出力のみを持っている。
- memory_target_if()はAMBA inferfaceを持つが、そのプロトコルは｛default:AXI4-128bit(address=48bit), AHB-32bit(address=32bit), CHI-256bit(address=48bit)}から選択可能で、かつ、System Verilogのinferface,package構文により抽象化されてdut_top()とバス接続されているとする。
- dut_initiator_ifx()はAMBA inferfaceを持つが、そのプロトコルは｛default:AXI4-128bit(address=48bit), AXI4-64bit(address=48bit), APB-64bit(address=32bit), NotPresent}から選択可能で、かつ、System Verilogのinferface,package構文により抽象化されてdut_top()とバス接続されているとする。
- どのinterfaceがどのプロトコルを利用しているかは、top_tb()上位でグローバルに明記されるとする。明記されない場合はdefaultが適応される。Not-Presentが設定された場合は、インスタンスとして存在しても害を及ぼさないようにする。
- top_tb()と、含まれるModuleは＜Module名＞.svとして実装されることを明記。
- すべての.svファイルをVerilator.5.xを使ってBuildするためのCMakefileも同時に生成することを明記。

下記の点をコーディングの注意点として、仕様書の中で明確にしておいてほしい。
１．すべてのモジュールはベンダー固有のPrimitiveを利用しない、SystemVerilogの仕様のみに準拠する。
２．インターフェースで"NotPresent"が選択された場合は、そのモジュールは害を及ぼさないように隔離される。
３．SystemVerilogの(interface, package, modport)はAMBAバスインターフェース部分の抽象化のみに利用する。

ここから仕様書docs/specification.mdを作ってみてください。`

### tuning by hands, with github copilot



