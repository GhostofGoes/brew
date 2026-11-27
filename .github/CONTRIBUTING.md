# Contributing to CherryMountain Brewery

Thank you for your interest in the CherryMountain Brewery range. This guide
describes how to propose a change and what a change must pass before it merges.

## Table of Contents

- [Getting Started](#getting-started)
- [How to Contribute](#how-to-contribute)
  - [Reporting Issues](#reporting-issues)
  - [Suggesting Enhancements](#suggesting-enhancements)
  - [Submitting Code](#submitting-code)
- [Code Quality](#code-quality)
  - [Dev container](#dev-container)
  - [Generated files](#generated-files)
  - [Companion repositories](#companion-repositories)
- [Updating Actions](#updating-actions)
- [License](#license)

## Getting Started

1. **Fork the Repository**: Click the "Fork" button at the top right of the repository page to create your own copy of the project.

2. **Clone Your Fork**: The OpenPLC program archives (`*.zip`) are in [Git LFS](https://git-lfs.com/), so install it first:
   ```bash
   git lfs install
   git clone https://github.com/<your-username>/brew.git
   ```

3. **Set Upstream Remote**: Add the original repository as an upstream remote to keep your fork up to date:
    ```bash
    git remote add upstream https://github.com/GhostofGoes/brew.git
    ```

4. **Install the Hooks**: Install the dev tooling and register the git hooks once:
    ```bash
    make install-dev
    ```

## How to Contribute

### Reporting Issues

If you encounter a bug or have a feature request, please open an issue in the [Issues](https://github.com/GhostofGoes/brew/issues) section. Be sure to include:

- A clear description of the issue.
- The variant of the range (`ci`, `lite`, `full`, or a site of the multi-site variant) and the commit.
- Steps to reproduce the issue.
- Any relevant screenshots or logs.

The range is a practice network. Its hosts have weak credentials and insecure
settings on purpose, and those are not bugs. `docs/DESIGN.md` records each one.

### Suggesting Enhancements

We welcome suggestions for improvements! Please open an issue to discuss your ideas before implementing them. `docs/FUTURE_WORK.md` lists the work that is already planned.

### Submitting Code

1. **Create a Branch**: Create a new branch for your feature or bug fix using [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) notation. The branch name should follow this format:
    ```bash
    type-description
    ```
    Where `type` can be one of the following:
    - `feat`: A new feature
    - `fix`: A bug fix
    - `ci`: Changes to the CI workflows and the hooks
    - `docs`: Documentation only changes
    - `style`: Changes that do not affect the meaning of the code (white-space, formatting, etc.)
    - `refactor`: A code change that neither fixes a bug nor adds a feature
    - `test`: Adding missing tests or correcting existing tests
    - `chore`: Changes to the build process, generated files, or auxiliary tools and libraries

    Example:
    ```bash
    git checkout -b feat-add-keg-washer
    ```

    A hook rejects a branch name that is not kebab-case (letters, digits, `-`, and `_` only), and a commit on `main`.

2. **Make Your Changes**: Implement your changes, and regenerate the files that depend on them (see [Generated files](#generated-files)).

3. **Commit Your Changes**: Commit your changes using [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) notation. A hook checks the format of the message:
    ```bash
    type(scope): subject
    ```
    * **Scope** is optional and names the part of the range that changes, for example `sim`, `plantgen`, `plc-cellar`, `fuxa`, `soc`, `viz`, or `images`.
    * **Subject** should be a short description of the change.

    Example:
    ```bash
    git commit -m "feat(sim): add the keg washer to the packaging model"
    ```

    Keep these in separate commits, so that a reviewer can read the change without the files that a tool wrote:
    * the change itself,
    * the generated files (`chore: regenerate ...`),
    * the documentation.

4. **Rebase Your Branch**: Before opening a pull request, ensure your branch is up to date with the main branch.
    ```bash
    git fetch upstream
    git rebase upstream/main
    ```

5. **Push to Your Fork**: If you had to rebase a branch that you already pushed, you need to force push it:
    ```bash
    git push origin feat-add-keg-washer --force-with-lease
    ```

6. **Open a Pull Request**: Go to the original repository and open a [pull request](https://github.com/GhostofGoes/brew/pulls). Provide a clear description of your changes and reference any related issues.

## Code Quality

This project uses [prek](https://prek.j178.dev/) (a Rust drop-in alternative to `pre-commit`) for every check: lint (shell, YAML, Dockerfiles, compose files, systemd units, spelling, GitHub Actions security with [zizmor](https://docs.zizmor.sh/)), the phenix JSON Schemas, the network plan, that generated files are current, and the test suites. The same hooks run in CI via the [CI workflow](workflows/ci.yml). The [README](../README.md#lint) describes what each group of hooks examines.

Run every hook against every file manually:

```bash
make lint
# or, equivalently
prek run --all-files
```

Run one hook, for example the tests of the simulation:

```bash
prek run --all-files cm-sim-tests
```

prek installs the hook tools itself (Python, Node). These hooks need something on the machine; the dev container has the same:

| Hook | Needs | Install |
|---|---|---|
| `hadolint-docker` (Dockerfiles) | Docker, which pulls the pinned hadolint image | [Docker Engine](https://docs.docker.com/engine/install/) |
| `cm-systemd-units` (systemd units) | `systemd-analyze` | `sudo apt-get install systemd` |
| `desktop-file-validate` (`.desktop` files) | `desktop-file-validate` | `sudo apt-get install desktop-file-utils` |
| `cm-fuxa-batch-tests` (FUXA batch engine) | Node 18 or newer | [Node.js](https://nodejs.org/) |

The hooks that check phenix configs fetch the pinned sceptre-phenix JSON
Schemas over HTTPS on first use and cache them. See the comments in
`prek.toml`.

Some tests are not hooks, because they need Docker, a browser, or a phenix
host. The [README](../README.md#regenerate-and-test) lists them and the
commands that run them.

### Dev container

CI runs the hooks in the Linux dev container in [`.devcontainer/`](../.devcontainer/).
Use the same container when a hook needs a tool that your machine does not have.
Docker must be installed.

```bash
make container-lint                                  # all hooks, as CI runs them
make container-run CMD='prek run --files <file>'     # hooks on some files
make container-shell                                 # a shell in the container
```

The image comes from GHCR, so you do not have to build it. For Windows, VS Code,
and the Dev Containers CLI, see [AGENTS.md](../AGENTS.md).

### Generated files

`plant/plant.yaml` is the source of the plant model. Tools write many files
from it: `plant/generated/`, the app blocks in the scenarios, the FUXA projects,
the copies under `phenix-injects/`, and the PLC program archives. Do not edit a
generated file by hand. Change the source, and run the tools again. The
[README](../README.md#regenerate-and-test) has the commands, in order.

The hooks fail when a generated file is stale (`cm-sync-check`,
`cm-fuxa-check`, and the tests of each generator).

### Companion repositories

The range needs code that lives in other repositories:

| Repository | What the range takes from it |
|---|---|
| [sceptre-phenix](https://github.com/GhostofGoes/sceptre-phenix) | phenix itself, with the changes that the range needs |
| [sceptre-phenix-apps](https://github.com/GhostofGoes/sceptre-phenix-apps) | The phenix apps that the range runs on. `prek.toml` pins a commit of it for the hooks that test the `cherrymountain` app and the `plantevent` SCORCH component |
| [sceptre-phenix-images](https://github.com/GhostofGoes/sceptre-phenix-images) | The recipes that build the disk images. Its `configs/cherrymountain/` holds copies of `phenix-images/` |

A change here can make a pin or a copy there stale, and the reverse. Say so in
the pull request, and link the pull request of the other repository.

## Updating Actions

[Dependabot](dependabot.yml) opens a pull request each month for the pinned
Actions. To update them by hand:

1. Change the version comment, e.g. `# v4` to `# v7` to update an action from v4 to v7.
1. Update the pinned hash:

    ```bash
    make update-actions
    ```

## License

By contributing to this project, you agree that your contributions will be licensed under the [GNU Affero General Public License v3.0](../LICENSE).
