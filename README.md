# sigstore-git-verifier

A Github Action to verify that new commits are present in the sigstore transparency log.

This is currently a proof-of-concept, and **not** ready for production use! Play around with it at your leisure, but be ready for things to break.

## How it works

This is effectively the verification stage of [Git commit signing with `cosign`](https://github.com/sigstore/cosign/blob/main/FUN.md) contained in a Github Action.

### Signing and uploading commits locally

This action assumes the commit it's verifying has been signed and uploaded to the Sigstore transparency log on a local developer's machine. That step is detailed [here](https://github.com/sigstore/cosign/blob/main/FUN.md#hard-mode); however, this repo also includes a [pre-push hook](/hooks/pre-push) that runs everything required before pushing a commit to Github.

For the pre-push hook to work, you'll need to do the following:

- Install [Cosign](https://github.com/sigstore/cosign#installation)
- Generate a Cosign keypair and store it in your repository's root directory
- Export your keypair's password as the `COSIGN_PASSWORD` environment variable with `export COSIGN_PASSWORD=***`

To install the pre-push hook, move [`hooks/pre-push`](/hooks/pre-push) to your `.git/hooks` directory:

```bash
mv hooks/pre-push .git/hooks/
```

Once moved, it will run before every push.

### Verifying commits on Github

The action can be used in a workflow that:

- performs a checkout of your repo
- installs Go `1.16` or later
- installs Cosign using [`cosign-installer`](https://github.com/sigstore/cosign-installer)

Here is an example workflow that will run the check on all pushes and incoming pull requests:

```yml
name: sigstore-git-verifier
on: [push, pull_request]

jobs:
  verify-commit:
    runs-on: ubuntu-latest
    name: Verify latest commit is in tlog
    steps:
      - uses: actions/checkout@v2
      - name: Set up Go
        uses: actions/setup-go@v2
        with:
          go-version: 1.16
      - uses: sigstore/cosign-installer@main
        with:
          cosign-release: 'v0.1.0'
      - name: Verify latest commit is in tlog
        uses: sigstore/sigstore-git-verifier@main
```

A working example is present on this repository: see [the test workflow.](/.github/workflows/test-action.yml)
