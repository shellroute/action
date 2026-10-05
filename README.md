# Shellroute Run

Run a trusted shell command through a Shellroute country route in GitHub Actions.

Use it for country-sensitive HTTP checks, redirect tests, API responses, localization checks, and other workflows where network origin is one input to the result.

Shellroute is a metered service. You need a Shellroute account with available credit and an API key stored in GitHub Actions secrets.

## Usage

```yaml
steps:
  - name: Check from Germany
    uses: shellroute/action@v1
    with:
      country: DE
      command: curl -fsSI https://your-site.example/
      api-key: ${{ secrets.SHELLROUTE_API_KEY }}
```

A routed run opens one metered Shellroute session and consumes Shellroute credit. An invocation rejected before the routed run, such as an invalid country code, an invalid version, or a failed credential check, opens no session.

## Inputs

- `country`: required. Two-letter country code such as `US`, `DE`, or `GB`.
- `command`: required. Trusted shell command to run through the route.
- `api-key`: required. Shellroute API key. Pass it from a GitHub Actions secret.
- `version`: optional. Shellroute CLI version to install. Defaults to `0.1.6`.

## Output

- `exit-code`: the exit status of the routed run. When the command ran, this is the command's own exit code. When the route could not be established, it is the Shellroute CLI's error code instead. The Action exits with that status, so a failing command fails the step. If the Action rejects its inputs before running, the step fails and `exit-code` is not set.

## How it runs

The Action installs the requested Shellroute CLI version, then runs:

```sh
shellroute run --no-stat "<country>" -- bash -c "<command>"
```

The full `command` runs as shell code inside the routed child process. Compound commands are routed as one command.

Only traffic from programs that honor the supplied proxy settings will use the route. A client can ignore or override those settings.

## Runner and client scope

The current hosted verification covers GitHub-hosted `ubuntu-latest`.

Other runners are not covered by that verification.

Playwright browser traffic requires explicit browser proxy configuration. Wrapping a Playwright command with this Action does not by itself make browser traffic use the proxy.

## Security

Treat `command` like a workflow `run:` block. Use fixed, trusted workflow-controlled commands.

Do not pass untrusted pull-request text, issue text, commit messages, or other untrusted event data directly into `command`.

Keep the Shellroute API key in GitHub Actions secrets and pass it through the `api-key` input. Avoid printing credentials or other sensitive values.

## Limits

Changing the route changes network origin. It does not guarantee that a third-party service will classify the exit address as a particular country or return a particular country-specific response.

Shellroute does not relocate the GitHub runner or emulate browser GPS.
