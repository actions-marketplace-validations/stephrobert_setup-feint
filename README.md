# setup-feint

Run Terraform, OpenTofu or the official Scaleway, Outscale and Exoscale CLIs in
GitHub Actions against a local emulator — **no cloud account, no credentials,
nothing billed.**

```yaml
- uses: stephrobert/setup-feint@v1
  with:
    version: 0.9.0
    provider: scaleway      # exports what the official client needs

- run: terraform apply -auto-approve
```

That job has no secret, because there is no account to authenticate to. The
provider that applies is the real one from the registry;
[Feint](https://github.com/stephrobert/feint) emulates the API it talks to.

## What it does

1. Downloads the released binary for the runner's platform.
2. **Verifies its checksum before running it** — the bytes are checked against
   the release's `checksums.txt`, not trusted because they came over HTTPS.
3. Starts the emulator through `feint start`, which detaches and waits until it
   answers rather than sleeping and hoping.
4. With `provider:`, exports the environment that client needs, so the next step
   needs no configuration at all.

## Inputs

| Input | Required | Default | What it is |
|---|---|---|---|
| `version` | yes | — | The release to install, without the leading `v` (for example `0.9.0`). Named rather than defaulted to latest: a mutable reference installs whatever is newest, which is a binary nobody can name afterwards |
| `addr` | no | `127.0.0.1:4599` | Where the emulator listens |
| `provider` | no | — | `scaleway`, `outscale` or `exoscale`: exports that client's environment into the job |

## Outputs

| Output | What it is |
|---|---|
| `endpoint` | The URL the emulator answers on |

## The other two doors

This action is for a job that does not already run in containers. If yours does,
the OCI image is the shorter path:

```yaml
services:
  feint:
    image: ghcr.io/stephrobert/feint:v0.9.0
    ports: ["4599:4599"]
```

And from a Go test, [`feinttest`](https://github.com/stephrobert/feint/tree/main/feinttest)
starts the image and cleans it up, with no dependency added to your module.

## What it does not do

The action starts the control plane. **Real machines** behind `--vm` need Incus
on the host — a GitHub-hosted runner can do it, but that is a different setup
and this action does not pretend otherwise. What each mode proves is written row
by row in
[docs/confidence.md](https://github.com/stephrobert/feint/blob/main/docs/confidence.md).

## Source

The action's body lives in the [Feint
repository](https://github.com/stephrobert/feint/blob/main/.github/actions/setup-feint/action.yml)
and is mirrored here on release, so there is one source and one CI. Report issues
there.

Apache-2.0.
