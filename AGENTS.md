# AGENTS.md

Instructions for AI coding agents that work in this repository. Read
[CONTRIBUTING.md](.github/CONTRIBUTING.md) for the branch, commit, and pull
request rules.

## Repository

This repository is the CherryMountain Brewery range: an OT cyber range for
[phenix](https://github.com/sandialabs/sceptre-phenix). [README.md](README.md)
describes the range and lists what each directory holds.
[USER_GUIDE.md](USER_GUIDE.md) is the procedure to run it, and
[docs/DESIGN.md](docs/DESIGN.md) is the design.

- The repository root is the topology directory. On a phenix host it is cloned
  to `/phenix/topologies/cherrymountain-brewery`, and the configs name that
  path. Do not rename the top-level directories that the configs name
  (`phenix-configs/`, `phenix-images/`, `phenix-injects/`).
- `plant/plant.yaml` is the source of the plant model. Do not edit a generated
  file (`plant/generated/`, the app blocks in the scenarios, the FUXA projects,
  the copies under `phenix-injects/`, the PLC program ZIPs). Change the source
  and run the tools again, in the order that "Regenerate and test" in the
  README gives.
- Commit the change, the generated files (`chore: regenerate ...`), and the
  documentation as separate commits.
- Every `*.zip` is in Git LFS. Run `git lfs pull` before the hooks: without the
  program ZIPs, some tests fail for a reason that is not your change.
- The range runs on forks of phenix, of its apps, and of its image recipes.
  They are other repositories, and `prek.toml` pins a commit of the apps
  repository for the hooks that test the `cherrymountain` app
  ([Companion repositories](.github/CONTRIBUTING.md#companion-repositories)).
  When a change here makes that pin stale, say so. Do not skip the hook.
- The Spikes workflow (`.github/workflows/spikes.yml`) boots real VMs and is
  slow. It runs on a push to `main` that changes `spikes/**`, the WAN generator,
  or the workflow itself, and on request (`workflow_dispatch`, which can select
  the spikes to run). Put `[skip ci]` in the message of a commit to those paths
  when a run is not wanted.

## Checks

All checks, the test suites included, are [prek](https://prek.j178.dev/) hooks
in `prek.toml`. CI runs them in the Linux dev container in `.devcontainer/`.
Some hooks need Linux tools (for example, `systemd-analyze`), so run the checks
in the same container.

Do not skip a hook with `SKIP` or `PREK_SKIP` because a tool is missing on
your machine. Run the hook in the dev container.

### Linux and macOS

Docker must be installed. Run these commands from the repository root:

| Task | Command |
|---|---|
| Run all hooks on all files, as CI does | `make container-lint` |
| Run hooks on some files | `make container-run CMD='prek run --files <file> <file>'` |
| Run a different command | `make container-run CMD='<command>'` |
| Start a shell | `make container-shell` |

`make` calls `.devcontainer/run.sh`. You can also call the script directly,
for example `.devcontainer/run.sh prek run --all-files`. The script:

- Pulls the image from GHCR. It builds the image locally only when the pull
  fails, for example on a branch that changes `.devcontainer/Dockerfile`.
- Mounts the checkout at the same path as on the host, so it also works in a
  git worktree.
- Runs the command as your user, so files that a hook changes stay yours.

Behind a network proxy that inspects TLS, prek cannot download the tools of
some hooks (the error is `invalid peer certificate: UnknownIssuer`). Set
`DEVCONTAINER_CA_FILE` to a PEM file that contains the CA certificates of the
proxy. Get this file from your IT department. Only `run.sh` uses this variable.

### Windows

Windows usually does not have GNU make. Use one of these procedures in
PowerShell. Use a normal clone, not a git worktree, and clone with
`core.autocrlf=false`. CRLF line endings cause hook failures.

With the Dev Containers CLI (Node.js must be installed):

1. Start the container:

   ```powershell
   npx @devcontainers/cli up --workspace-folder .
   ```

2. Run the hooks:

   ```powershell
   npx @devcontainers/cli exec --workspace-folder . prek run --all-files
   ```

With Docker only:

1. Pull the image:

   ```powershell
   docker pull ghcr.io/ghostofgoes/brew/devcontainer:latest
   ```

2. Run the hooks:

   ```powershell
   docker run --rm -v "${PWD}:/workspace" -w /workspace `
     -v brew-cache-root:/cache `
     -v /var/run/docker.sock:/var/run/docker.sock `
     ghcr.io/ghostofgoes/brew/devcontainer:latest `
     prek run --all-files
   ```

The `latest` image is built from the `.devcontainer/Dockerfile` of the `main`
branch. If the pull fails, or if your branch changes the Dockerfile, build the
image first with `docker build -t <name> .devcontainer`, and then use `<name>`
in place of the GHCR image name.

In VS Code, the Dev Containers extension also opens the repository in the
container (command "Dev Containers: Reopen in Container").

## Dev container

- `.devcontainer/Dockerfile` contains every pinned tool version.
  `PREK_VERSION` must be at least `minimum_prek_version` in `prek.toml`.
- The image tag is a hash of the Dockerfile. The "Dev container" workflow
  publishes the image when the Dockerfile changes on `main`.
- When a hook needs a new system tool, add the tool to the Dockerfile. Do not
  install it in a workflow step.
