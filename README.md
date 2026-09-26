# PR-Agent GitHub Workflow <!-- omit from toc -->

---

Example GitHub Actions workflow for running [PR-Agent](https://github.com/The-PR-Agent/pr-agent) on pull requests, configured here to use OpenAI.

PR-Agent can automatically review and suggest improvements for pull requests, as well as respond to supported slash commands (like [`/ask`](https://docs.pr-agent.ai/tools/ask/)) posted in PR comments.

This example adds some security hardening beyond PR-Agent's [simpler GitHub Actions setup](https://docs.pr-agent.ai/installation/github/#configuration-examples), including image digest pinning and GitHub artifact attestation verification.

Official documentation: <https://docs.pr-agent.ai/>

---

## Table of contents <!-- omit from toc -->

- [Included files](#included-files)
- [Why this workflow is more involved](#why-this-workflow-is-more-involved)
- [Setup](#setup)
- [Configuration](#configuration)
  - [PR-Agent configuration](#pr-agent-configuration)
  - [Workflow configuration](#workflow-configuration)
- [Updating PR-Agent](#updating-pr-agent)

---

## Included files

- [`.github/workflows/pr-agent.yml`](.github/workflows/pr-agent.yml) — runs PR-Agent and handles GitHub Actions integration.
- [`.pr_agent.toml`](.pr_agent.toml) — repository-level PR-Agent configuration for review behavior, code suggestions, and other tool settings.

## Why this workflow is more involved

PR-Agent's documentation includes a much simpler GitHub Actions setup:

<https://docs.pr-agent.ai/installation/github/#configuration-examples>

This workflow adds a few security-focused controls:

- Pins the PR-Agent container image to an immutable SHA-256 digest.
- Verifies the image's GitHub artifact attestation before executing it.
- Uses the same `PR_AGENT_IMAGE` value for both attestation verification and execution.
- Restricts PR comment commands to repository owners, members, and collaborators.
- Skips PRs originating from forks before passing secrets to PR-Agent.

The explicit `docker run` is intentional: it allows the exact image reference that is verified to also be the image that is executed.

## Setup

1. **Add the OpenAI API key**

   This example uses OpenAI, so create an `OPENAI_KEY` repository secret in GitHub.

   > **Note:** If `OPENAI_KEY` is not configured, the workflow exits successfully without running PR-Agent and adds a note to the GitHub Actions job summary.

2. **Add the workflow**

   Copy [`.github/workflows/pr-agent.yml`](.github/workflows/pr-agent.yml) to the same path in your repository:

   ```text
   .github/workflows/pr-agent.yml
   ```

   The workflow uses GitHub's automatically provided `GITHUB_TOKEN` for GitHub API access.

3. **Add repository configuration (optional)**

   To use the included configuration, copy [`.pr_agent.toml`](.pr_agent.toml) to the root of your repository.

   You can also create your own `.pr_agent.toml` with only the settings you want to override. See the official PR-Agent configuration documentation:

   <https://docs.pr-agent.ai/usage-guide/configuration_options/>

## Configuration

PR-Agent can be configured in two places:

- In `.pr_agent.toml` — controls PR-Agent's review and suggestion behavior.
- In the workflow — controls how and when PR-Agent runs in GitHub Actions.

### PR-Agent configuration

PR-Agent supports repository-level configuration through a `.pr_agent.toml` file in the repository root.

The included [`.pr_agent.toml`](.pr_agent.toml) configures things such as:

- Additional review instructions.
- Review output behavior.
- Code suggestion behavior and thresholds.
- Whether suggestions should focus only on identified problems.
- Changelog behavior.

Repository-specific settings can be changed without modifying the GitHub Actions workflow.

PR-Agent configuration documentation:

<https://docs.pr-agent.ai/usage-guide/configuration_options/>

The complete set of available configuration options and defaults can also be found in PR-Agent's `configuration.toml`:

<https://github.com/The-PR-Agent/pr-agent/blob/main/pr_agent/settings/configuration.toml>

You generally only need to include settings you want to override in your local `.pr_agent.toml`.

### Workflow configuration

The [`.github/workflows/pr-agent.yml`](.github/workflows/pr-agent.yml) workflow exposes the main automatic tool settings near the top of the file:

```yaml
PR_AGENT_AUTO_REVIEW: "true"    # Auto-run the review tool: https://docs.pr-agent.ai/tools/review
PR_AGENT_AUTO_DESCRIBE: "false" # Auto-run the describe tool: https://docs.pr-agent.ai/tools/describe
PR_AGENT_AUTO_IMPROVE: "true"   # Auto-run the improve tool: https://docs.pr-agent.ai/tools/improve
```

These control which PR-Agent tools run automatically. Adjust them as needed for your repository.

For more information on PR-Agent tools, see <https://docs.pr-agent.ai/tools/>.

The workflow also defines `PR_AGENT_PR_ACTIONS`, which controls which pull request events trigger PR-Agent's automatic actions. Keep this value in sync with `on.pull_request.types` in the workflow.

## Updating PR-Agent

To upgrade PR-Agent, update the `PR_AGENT_IMAGE` value in the workflow to the desired `github_action` image version and SHA-256 digest:

```yaml
PR_AGENT_IMAGE: pragent/pr-agent:<version>-github_action@sha256:<digest>
```

The attestation verification step derives the digest from this value, so the image reference only needs to be updated in one place.
