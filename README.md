# lychee-pre-commit

A [pre-commit](https://pre-commit.com/) hook for
[lychee](https://github.com/lycheeverse/lychee), the fast, async, stream-based
link checker written in Rust.

It installs lychee from the prebuilt
[`lychee-bin`](https://pypi.org/project/lychee-bin/) wheels that the lychee
project publishes to PyPI, so it works on Windows, macOS, and Linux with no Rust
toolchain, `cargo`, or `bash` required.

## Why this exists

lychee already ships
[official pre-commit hooks](https://github.com/lycheeverse/lychee/blob/master/docs/PRE_COMMIT.md).
The recommended auto-installing `lychee` hook is a `bash` script, so it fails on
Windows, which has no `bash` by default
([lycheeverse/lychee#2238](https://github.com/lycheeverse/lychee/issues/2238)).

The natural fix — a `language: python` hook installed from PyPI — cannot live in
the lychee repository itself. pre-commit installs a `language: python` hook by
running `pip install .` against the hook repo's root, and lychee's root
`pyproject.toml` is a [maturin](https://github.com/PyO3/maturin) build config, so
that would try to compile lychee from source instead of fetching a prebuilt
binary.

A small standalone repository sidesteps that: its only dependency is the
`lychee-bin` wheel, so pre-commit installs the right prebuilt binary on every
platform. This is the same approach Ruff takes with
[ruff-pre-commit](https://github.com/astral-sh/ruff-pre-commit), and which
[rumdl-pre-commit](https://github.com/rvben/rumdl-pre-commit) and
[ryl-pre-commit](https://github.com/owenlamont/ryl-pre-commit) copied.

> [!NOTE]
> This is an **unofficial** mirror maintained by
> [@owenlamont](https://github.com/owenlamont). It only repackages the prebuilt
> `lychee-bin` wheels that the lychee project already publishes to PyPI; all
> credit for lychee belongs to its
> [authors and maintainers](https://github.com/lycheeverse/lychee). If lychee
> publishes an equivalent hook under the lycheeverse umbrella, prefer that.

## Usage

Add the hook to your `.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/owenlamont/lychee-pre-commit
    # Match the lychee release you want to pin.
    rev: v0.24.2
    hooks:
      - id: lychee
```

The hook runs `lychee --no-progress` over the text files in your commit. Pass
additional CLI options through the `args` key, for example to exclude some
schemes:

```yaml
  - repo: https://github.com/owenlamont/lychee-pre-commit
    rev: v0.24.2
    hooks:
      - id: lychee
        args: [--no-progress, --exclude, "file://", --exclude, "mailto:"]
```

On non-Windows systems you can also use lychee's
[official hooks](https://github.com/lycheeverse/lychee/blob/master/docs/PRE_COMMIT.md)
directly, including the `lychee-system` and `lychee-docker` variants.

## How releases track lychee

A scheduled workflow polls PyPI for new `lychee-bin` releases and, when one
appears, bumps the pinned version here and publishes a matching `vX.Y.Z` tag and
GitHub release. Pin `rev:` to the lychee version you want.

## License

MIT (see [LICENSE](LICENSE)). lychee itself is dual-licensed under Apache-2.0 or
MIT.
