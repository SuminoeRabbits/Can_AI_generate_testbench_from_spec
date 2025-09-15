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


## GitHub Actions for generation

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

## GitHub Actions for execution

