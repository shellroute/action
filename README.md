# Shellroute Run

Run a command through a country-specific proxy from a GitHub Actions workflow.

Use it for country-sensitive HTTP checks, redirect tests, API responses, and other commands where network origin matters. The command runs on the GitHub runner; Shellroute supplies proxy settings to the child process.

Shellroute is a metered service. You need a [Shellroute account](https://shellroute.com/docs/quickstart) with available credit and an API key stored as the `SHELLROUTE_API_KEY` GitHub Actions secret.

## Usage

```yaml
steps:
  - uses: shellroute/action@v1
    with:
      api-key: ${{ secrets.SHELLROUTE_API_KEY }}
      country: DE
      command: curl -sI https://your-site.example/
```

Compound commands stay inside the same route:

```yaml
  - uses: shellroute/action@v1
    with:
      api-key: ${{ secrets.SHELLROUTE_API_KEY }}
      country: US
      command: 'curl -sI https://store.google.com/ && curl -s https://ipinfo.io/country'
```

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `api-key` | yes | — | Shellroute API key (pass `${{ secrets.SHELLROUTE_API_KEY }}`) |
| `country` | yes | — | Two-letter country code (US, DE, GB, etc.) |
| `command` | yes | — | Command to run through the proxy |
| `version` | no | `0.1.6` | Shellroute CLI version to install |

## Output

| Output | Description |
|---|---|
| `exit-code` | Exit code of the child command |

The step succeeds when the child exits 0 and fails on non-zero. Auth or session-start failures also produce non-zero exits, so a failed step does not always mean the child command failed — check the logs.

## Runner support

Linux hosted runners (`ubuntu-latest`). macOS is expected to work but untested. Windows is not supported.

## Security

The `command` input is executed as shell code. Use fixed, trusted commands — do not pass untrusted PR titles, issue text, or commit messages into `command`.

Pass `SHELLROUTE_API_KEY` from GitHub Actions secrets via the `api-key` input. GitHub masks secret values in logs.

## Limits

Changing the route changes network origin. It does not guarantee that a third-party service will classify the proxy IP as a particular country. Geo-IP databases can disagree.

Shellroute does not relocate the runner, emulate browser GPS, or make every program use a proxy. Programs must read the proxy environment variables — tools that ignore them are not routed. For Playwright, see the [explicit proxy config](https://github.com/shellroute/playwright-country-routing-example).

## License

Apache 2.0
