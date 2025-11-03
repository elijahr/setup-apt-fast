# Setup apt-fast

A GitHub Action to install and configure [apt-fast](https://github.com/ilikenwf/apt-fast) on Ubuntu/Debian runners. apt-fast is a shell script wrapper for apt-get that dramatically speeds up package downloads by using parallel connections.

## What is apt-fast?

apt-fast accelerates apt package downloads by leveraging aria2c to download packages in parallel with multiple connections per package. This can significantly reduce CI/CD build times when installing dependencies.

## Quick Start

Add this step to your workflow before installing packages:

```yaml
- name: Setup apt-fast
  uses: elijahr/setup-apt-fast@v1

- name: Install dependencies
  run: |
    sudo apt-fast update
    sudo apt-fast install -y <all the things>
```

## Configuration

All inputs are optional. The action uses sensible defaults optimized for GitHub Actions runners.

### Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `apt-manager` | Package manager to use (`apt-get`, `aptitude`, or `apt`) | `apt-get` |
| `max-downloads` | Maximum number of parallel downloads | `16` |
| `max-connections-per-server` | Maximum connections per server | `10` |
| `split-connections` | Maximum connections per file | `16` |
| `min-split-size` | Size of each split piece (e.g., `1M`, `5M`, `10M`) | `1M` |
| `piece-algorithm` | Piece selection algorithm (`default`, `inorder`, `geom`) | `default` |
| `download-before` | Skip confirmation dialog (`true`/`false`) | `true` |
| `mirrors` | Comma or space-separated list of mirror URLs | (empty) |
| `verbose-output` | Display detailed aria2 download information (`true`/`false`) | `false` |
| `apt-auth` | Enable authentication for sources requiring login (`true`/`false`) | `true` |

### Example: Basic Usage

```yaml
- name: Setup apt-fast
  uses: elijahr/setup-apt-fast@v1
```

### Example: Custom Configuration

```yaml
- name: Setup apt-fast with custom settings
  uses: elijahr/setup-apt-fast@v1
  with:
    max-downloads: 20
    max-connections-per-server: 16
    split-connections: 20
    min-split-size: 5M
    verbose-output: true
```

### Example: With Custom Mirrors

```yaml
- name: Setup apt-fast with mirrors
  uses: elijahr/setup-apt-fast@v1
  with:
    mirrors: 'http://mirror1.example.com/ubuntu, http://mirror2.example.com/ubuntu'
```

## Examples

### Example: Before

Standard apt-get installation:

```yaml
name: CI
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install dependencies
        run: |
          sudo apt-get update
          sudo apt-get install -y build-essential cmake libssl-dev

      - name: Build project
        run: make
```

### Example: After

With apt-fast for faster downloads:

```yaml
name: CI
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup apt-fast
        uses: elijahr/setup-apt-fast@v1

      - name: Install dependencies
        run: |
          sudo apt-fast update
          sudo apt-fast install -y build-essential cmake libssl-dev

      - name: Build project
        run: make
```

### Example: High-Performance Setup

For maximum speed when installing many packages:

```yaml
name: CI
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup apt-fast (high performance)
        uses: elijahr/setup-apt-fast@v1
        with:
          max-downloads: 30
          max-connections-per-server: 16
          split-connections: 30
          min-split-size: 10M

      - name: Install dependencies
        run: |
          sudo apt-fast update
          sudo apt-fast install -y build-essential cmake libssl-dev libboost-all-dev

      - name: Build project
        run: make
```

## Requirements

- Ubuntu or Debian-based GitHub Actions runners
- Works with: `ubuntu-latest`, `ubuntu-22.04`, `ubuntu-20.04`, etc.

## How it works

This action:
1. Verifies the runner is Ubuntu/Debian-based
2. Installs required prerequisites
3. Adds the official apt-fast PPA
4. Installs apt-fast with your specified configuration
5. Configures all apt-fast settings according to your inputs
6. Displays a configuration summary for verification

After installation, use `apt-fast` instead of `apt-get` in your workflow commands.

## Performance

apt-fast can significantly reduce package installation time, especially when installing many or large packages. The actual speedup depends on:
- Network bandwidth
- Mirror availability
- Number and size of packages being installed

Typical improvements range from 2-5x faster download speeds.

## Development

### Pre-commit Hooks

This repository uses [pre-commit](https://pre-commit.com/) to ensure code quality and consistency. The hooks run automatically before each commit to check:

- **YAML linting** (yamllint)
- **GitHub Actions linting** (actionlint)
- **Shell script linting** (shellcheck)
- **Shell script formatting** (shfmt)
- **Basic file checks** (trailing whitespace, end-of-file, etc.)

#### Setup

Install pre-commit:

```bash
# Using pip
pip install pre-commit

# Or using homebrew (macOS)
brew install pre-commit
```

Install the git hooks:

```bash
pre-commit install
```

#### Usage

The hooks will run automatically on `git commit`. To run manually:

```bash
# Run on all files
pre-commit run --all-files

# Run on specific files
pre-commit run --files action.yml
```

To skip hooks (not recommended):

```bash
git commit --no-verify
```

#### Hook Configuration

- `.pre-commit-config.yaml` - Pre-commit hook configuration
- `.yamllint` - YAML linting rules
- `.shellcheckrc` - Shell script linting configuration

## License

MIT

## Related Links

- [apt-fast repository](https://github.com/ilikenwf/apt-fast)
- [apt-fast PPA](https://launchpad.net/~apt-fast/+archive/ubuntu/stable)
