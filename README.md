# pin-github-action

This is a tool that allows you to pin your GitHub actions dependencies to a
specific sha without requiring that you update every action manually each time
you want to use a newer version of an action.

It achieves this by converting your workflow to use a specific commit hash,
whilst adding the original value as a comment on that line. This allows us to
resolve newer shas for that target ref automatically in the future.

It converts this:

```yaml
name: Commit Push
on:
  push:
    branches:
      - master
jobs:
  build:
    name: nexmo/github-actions/submodule-auto-pr@master
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@master
    - name: nexmo/github-actions/submodule-auto-pr
      uses: nexmo/github-actions/submodule-auto-pr@master
```

In to this:

```yaml
name: Commit Push
on:
  push:
    branches:
      - master
jobs:
  build:
    name: nexmo/github-actions/submodule-auto-pr@master
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@db41740e12847bb616a339b75eb9414e711417df # pin@master
      - name: nexmo/github-actions/submodule-auto-pr
        uses: nexmo/github-actions/submodule-auto-pr@73549280c1c566830040d9a01fe9050dae6a3036 # pin@master
```

For more information, see [How it works](#how-it-works).

## Installation

```
npm install -g pin-github-action
```

## Usage

```bash
pin-github-action /path/to/.github/workflows/your-name.yml
```

If you use private actions (or are hitting rate limits), you'll need to provide
a GitHub access token:

```bash
GH_ADMIN_TOKEN=<your-token-here> pin-github-action /path/to/.github/workflows/your-name.yml
```

Run it as many times as you like! Each time you run the tool the exact sha will
be updated to the latest available sha for your pinned ref.

## How it works

* Load the workflow file provided
* Tokenise it in to an AST
* Extract all `uses` steps, skipping any `docker://` or `./local-path` actions
* Loop through all `uses` steps to determine the target ref
  * If there's a comment in the step, remove `pin@` and use that as the target
  * Otherwise, fall back to the ref in the action as the default
* Look up the current sha for each repo on GitHub and update the action to use the specific hash
  * If needed, add a comment with the target pinned version
* Write the workflow file with the new pinned version and original target version as a comment
