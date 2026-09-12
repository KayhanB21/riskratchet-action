# riskratchet-action

GitHub Marketplace wrapper for [`KayhanB21/riskratchet`](https://github.com/KayhanB21/riskratchet) — a
maintainability ratchet for AI-assisted Python and TypeScript.

This repo exists for Marketplace discoverability. The action logic lives in the
root `action.yml` of the `riskratchet` repo; this wrapper's `action.yml`
delegates to it with input passthrough so both shapes share one source of truth.

## Usage

```yaml
on: [pull_request]

jobs:
  riskratchet:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683  # v4.2.2
        with:
          # Full history: churn uses `git log --since`, which on the default
          # shallow (depth-1) clone sees only HEAD and silently scores every
          # function's churn as zero — so CI would disagree with your baseline.
          fetch-depth: 0
      - uses: actions/setup-python@5fda3b95a4ea91299a34e894583c3862153e4b97  # v7.0.0
        with:
          python-version: '3.12'
      # Install your project and its test dependencies however you normally do:
      - run: pip install -e '.[dev]'
      - run: pytest --cov --cov-branch --cov-report=json:coverage.json -q
      - uses: KayhanB21/riskratchet-action@v1
        with:
          coverage: coverage.json
```

The two steps before the action are not optional. `coverage: coverage.json` names a
report, and **a path you name must exist** — riskratchet exits `2` rather than scoring
every function as uncovered. Leaving the input out does not help either: auto-coverage
shells out to `pytest`, which is not on the `PATH` of the `uv tool install` environment
the action creates, so that is exit `2` as well. Whatever writes `coverage.json` is
yours to choose — `coverage run -m unittest discover && coverage json -o coverage.json`
works just as well — as long as the workflow writes it before the action runs.

If you already use `riskratchet`, you can skip this wrapper and reference the
root action directly:

```yaml
- uses: KayhanB21/riskratchet@v0.3.7
```

Both forms accept the same inputs. See the
[main README](https://github.com/KayhanB21/riskratchet#github-action) for the
full inputs table and CLI documentation.

Since `v1.1.0` the wrapper also passes through `typescript`, `ts-coverage`, and
`ts-entry` (riskratchet 0.3.6+), so a repo whose `[tool.riskratchet]` sets
`typescript = true` gates its TypeScript through `@v1` too:

```yaml
      - uses: KayhanB21/riskratchet-action@v1
        with:
          typescript: 'true'
          ts-coverage: coverage/lcov.info
```

## Versioning

This wrapper repo follows its own version line independent of the riskratchet
package. The internal `uses:` ref is bumped manually when a new riskratchet
release should be the default for `@v1` consumers. Pin to a specific tag
(`@v1.0.0`) if you need stability across wrapper updates.

## License

MIT. See [LICENSE](LICENSE).
