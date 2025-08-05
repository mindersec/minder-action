# Minder GitHub Action

This action [installs the Minder CLI](https://mindersec.github.io/getting_started/install_cli), authenticates with the chosen server using a GitHub Actions token, and (optionally) loads ruletypes, data sources, and profiles from the specified directory, allowing you to manage your [Minder](https://mindersec.github.io) project using infrastructure-as-code (IaC) techniques.

## Setup

Before using this action, you will need to grant the GitHub Action role permission to manage your Minder project.  You can use the `project role grant` sub-command to grant permissions on the server, for example:

```bash
minder project role grant \
  --project 00000000-0000-0000-0000-000000000000 \
  --sub githubactions/repo:myorg/myrepo:ref:refs/heads/main \
  --role admin
```

The above command grants GitHub actions running from the `main` branch of `myorg/myrepo` **admin** permission on the specified project.  ([`admin` is currently required to create or update data sources](https://github.com/mindersec/minder/blob/main/internal/authz/model/minder.fga#L94))  You can get your project ID from `minder auth whoami` if you are logged in to the minder server.

## Usage

```yaml
name: Apply Minder Profiles on Update
on:
  push:
    branches: [ main ]
    paths:
    - .github/minderconfig

permissions:
  # id-token: write is needed to fetch a GitHub Actions token for other services
  id-token: write
  # contents: read is needed to checkout the current repo
  contents: read

jobs:
  minder-config:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout repo
      uses: actions/checkout@v4
    - name: Apply Minder Config
      # You should look into pinning the action to a specific SHA, but that
      # does not work well for checked-in documentation.
      uses: custcodian/minder-action@v1
      with:
        server: api.custcodian.dev
        # Get the project from `minder auth whoami`.  At some point, project
        # names will be supported, but they aren't yet.
        project: 00000000-0000-0000-0000-000000000000
        # Set directory if you want to load and manage configuration from
        # checked-in manifests.  If you do not set the directory argument,
        # the action will install the minder CLI and add it to the PATH, but
        # will not apply any configuration, so you can (for example) generate
        # configuration and apply it in a later step.
        directory: .github/minder
```

## Inputs

### `server`

The Minder server to authenticate to, and send commands to.  If not set, defaults to `api.custcodian.dev`.

### `project`

The Minder project to apply the configuration to.  Both `project` and `directory` must be set to load rules from a directory.

### `directory`

A directory (or file) containing Minder configuration YAML to apply to the server.  Both `project` and `directory` must be set to load rules.

### `release`

_(optional)_ The Minder client release to download and use.  By default, this is hard-coded to a specific version for each release of the action, so that if you pin the action, you won't pick up new versions of the CLI until requested.