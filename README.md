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
      - uses: KayhanB21/riskratchet-action@v1
        with:
          coverage: coverage.json
```

If you already use `riskratchet`, you can skip this wrapper and reference the
root action directly:

```yaml
- uses: KayhanB21/riskratchet@v0.3.5
```

Both forms accept the same inputs. See the
[main README](https://github.com/KayhanB21/riskratchet#github-action) for the
full inputs table and CLI documentation.

## Versioning

This wrapper repo follows its own version line independent of the riskratchet
package. The internal `uses:` ref is bumped manually when a new riskratchet
release should be the default for `@v1` consumers. Pin to a specific tag
(`@v1.0.0`) if you need stability across wrapper updates.

## License

MIT. See [LICENSE](LICENSE).
