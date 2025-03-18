# Automated PR Comments for GitHub Actions

This reusable workflow adds automated comments on Pull Requests from external contributors. It identifies external contributors as users who are not members of your GitHub organization.

## Features

The workflow can leave comments in these situations:

- On a contributor's **first** Pull Request to your repository
- When a Pull Request is marked as **ready for review**
- When a Pull Request is **merged**

Each comment type can be individually enabled or disabled.

## Setup

To use this workflow in your repository, create a new workflow file in your `.github/workflows` directory:

```yaml
name: PR Comments

on:
  pull_request_target:
    types: [opened, ready_for_review, closed]

jobs:
  pr-comments:
    uses: your-org/auto-comments/.github/workflows/pr-auto-comments.yml@main
    with:
      org_name: "your-organization-name"
      # Enable only the comment types you want by providing content
      first_pr_comment: |
        # Welcome to our project!

        Thanks for your first contribution. We're glad you're here.
      # To disable a comment type, either remove it or leave it empty
      merged_pr_comment: |
        # PR Merged

        Thank you for your contribution!
    secrets:
      token: ${{ secrets.GITHUB_TOKEN }}
```

## Configuration

### Required Inputs

| Input | Description |
|-------|-------------|
| `org_name` | GitHub organization name to identify internal team members |

### Optional Inputs

| Input | Description | Default | Behavior if not provided |
|-------|-------------|---------|--------------------------|
| `additional_internal_users` | Additional comma-separated list of usernames to consider as internal | `''` | No additional users excluded |
| `first_pr_comment` | Message for first PR comments | `''` | First PR comments disabled |
| `ready_for_review_comment` | Message for ready for review comments | `''` | Ready for review comments disabled |
| `merged_pr_comment` | Message for merged PR comments | `''` | Merged PR comments disabled |

### Secrets

| Secret | Description | Required | Default |
|--------|-------------|----------|---------|
| `token` | GitHub token with org:read permission | No | `github.token` |

## Enabling and Disabling Comment Types

You can selectively enable comment types by providing content for the corresponding input:

- To **enable** a comment type: provide the message text
- To **disable** a comment type: omit the input or provide an empty string

For example, to enable only first PR and merged PR comments:

```yaml
first_pr_comment: |
  # Welcome!
  Thanks for your contribution.
# ready_for_review_comment is omitted to disable it
merged_pr_comment: |
  # Merged
  Your PR has been merged.
```

## Comment Formatting

You can use full Markdown syntax in your comment messages, including:

- Headings (`# Heading`)
- Lists (`- Item`)
- Formatting (**bold**, *italic*)
- Links (`[text](url)`)
- Code blocks
- Emojis (`:tada:`)

## How It Works

1. The workflow first checks the PR author in a central job:
   - Determines if they are an external contributor (not in your org or additional users list)
   - For first PR comments, checks if this is their first contribution

2. Based on the event type (opened/ready for review/merged) and author status:
   - Runs only the appropriate comment job
   - Only if the corresponding comment text is provided

3. Each enabled job posts its specific comment to the PR

This architecture ensures contributor checks run only once per workflow execution, making the process more efficient.

## Security Considerations

This workflow uses `pull_request_target` to ensure it has the necessary permissions to comment on PRs. Since this event has access to secrets, the workflow is designed to only perform safe operations (commenting) and does not check out code from external PRs.

The workflow requires a token with `org:read` permission to check organization membership.

## License

MIT
