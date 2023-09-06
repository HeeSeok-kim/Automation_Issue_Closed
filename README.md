# GitHub Action to Automation Issue Closed
This is an action that automatically terminates Issue when PR is done in a branch other than the default branch.

## Usage
Place in a `.yml` file such as this one in your `.github/workflows` folder. [Refer to the documentation on workflow YAML syntax here.](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)

```bash
name: Close associated issue

on:
  pull_request:
    branches:
      - dev
    types:
      - closed

jobs:
  close-issue:
    runs-on: ubuntu-latest
    steps:
    - name: Close associated issue
      run: |
        PR_NUMBER=${{ github.event.pull_request.number }}
        PR_URL="https://api.github.com/repos/${{ github.repository }}/pulls/$PR_NUMBER"
        PR_BODY=$(curl -s -H "Authorization: token ${{ secrets.ACTION_TOKEN }}" $PR_URL | jq -r '.body')
        ISSUE_NUMBER=$(echo $PR_BODY | grep -oE "close #[0-9]+" | tr -d 'close #')
        if [[ ! -z "$ISSUE_NUMBER" ]]; then
          curl -s -H "Authorization: token ${{ secrets.ACTION_TOKEN }}" -X PATCH "https://api.github.com/repos/${{ github.repository }}/issues/$ISSUE_NUMBER" -d '{"state": "closed"}'
        fi
      shell: bash
```

## Configuration

The following settings must be passed as environment variables as shown in the example. Sensitive information, especially `ACTION_TOKEN`
should be [set as encrypted secrets](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners#creating-and-using-secrets-encrypted-variables) — otherwise, they'll be public to anyone browsing your repository's source code and CI logs.

## How to use it
Create `close #${issue number}` on body when PR is created.
<img width="884" alt="image" src="https://github.com/HeeSeok-kim/Automation_Issue_Closed/assets/106604926/569b7f3a-185f-423b-b798-91ac96c26fd9">

## License
This project is distributed under the [MIT license](LICENSE.md).
