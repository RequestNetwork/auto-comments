# Automated PR Comments for GitHub Actions

This reusable workflow adds automated comments on Pull Requests from external contributors. It identifies external contributors as users who are not members of your GitHub organization.

## Overview

The `pr-auto-comments` workflow automatically posts customizable comments on Pull Requests submitted by external contributors (those outside your organization). It helps maintain consistent communication with contributors while reducing manual effort from maintainers.

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
    uses: RequestNetwork/auto-comments/.github/workflows/pr-auto-comments.yml@main
    with:
      org_name: "your-organization-name"
      # Optional: override the default comments
      first_pr_comment: |
        # Welcome to our project!

        Thanks for your first contribution, @{{username}}. We're glad you're here.
    secrets:
      token: ${{ secrets.GITHUB_TOKEN }}
```

## Workflow Updates and Versioning

This workflow uses a reference to the branch (`@main`) rather than a specific version tag. This means:

- **Automatic Updates**: When the workflow code is updated in the `main` branch, all repositories referencing it will automatically use the latest version without requiring any changes in those repositories.
- **Breaking Changes**: Be cautious when making changes to the workflow in the `main` branch, as they will immediately affect all dependent repositories. Test significant changes thoroughly before merging them into `main`.

### Recommendations for Stability

If you need more stability and control over updates, consider:
1. Using version tags (e.g., `@v1`, `@v2`) instead of `@main`.
2. Having repositories explicitly opt-in to new versions by updating their workflow reference.

For example, to use a specific version tag:
```yaml
uses: your-org/auto-comments/.github/workflows/pr-auto-comments.yml@v1
```

## Configuration

### Required Inputs

| Input | Description |
|-------|-------------|
| `org_name` | GitHub organization name to identify internal team members |

### Optional Inputs

| Input | Description | Default | Behavior if empty string |
|-------|-------------|---------|--------------------------|
| `additional_internal_users` | Additional comma-separated list of usernames to consider as internal | `''` | No additional users excluded |
| `first_pr_comment` | Message for first PR comments | Default welcome message | First PR comments disabled |
| `ready_for_review_comment` | Message for ready for review comments | Default guidelines message | Ready for review comments disabled |
| `merged_pr_comment` | Message for merged PR comments | Default thank you message | Merged PR comments disabled |

### Secrets

| Secret | Description | Required | Default |
|--------|-------------|----------|---------|
| `token` | GitHub token with org:read permission | No | `github.token` |

## Default Messages

The workflow includes default messages for each comment type:

### First PR Comment
A welcome message that mentions the contributor by username, introduces them to the project, and highlights the Best PR Initiative with its $500 quarterly prize.

### Ready for Review Comment
A reminder about contribution guidelines and the Best PR Initiative, encouraging clear PR descriptions to expedite the review process.

### Merged PR Comment
A congratulatory message that thanks the contributor, reminds them about the Best PR Initiative, and promotes the Request Network API for crypto payments and invoicing features.

## Enabling and Disabling Comment Types

By default, all comment types are enabled with predefined messages. You can:

- **Override** a default comment by providing your own message text
- **Disable** a comment type by providing an empty string `''`

For example, to disable ready for review comments:

```yaml
ready_for_review_comment: ''  # Explicitly set to empty string to disable
```

## Dynamic Content with Variable Placeholders

You can include dynamic content in your messages using placeholders with the format `{{variable}}`. The following variables are available:

| Variable | Description | Example |
|----------|-------------|---------|
| `{{username}}` | The PR author's username | `octocat` |
| `{{repository}}` | The repository name | `auto-comments` |
| `{{org}}` | The organization/owner name | `your-org` |

Example usage in a custom message:

```yaml
first_pr_comment: |
  # Welcome @{{username}}!

  Thank you for your first contribution to the {{repository}} repository.
  We at {{org}} appreciate your interest in our project.
```

## Comment Formatting

You can use full Markdown syntax in your comment messages, including:

- Headings (`# Heading`)
- Lists (`- Item`)
- Formatting (**bold**, *italic*)
- Links (`[text](url)`)
- Code blocks
- Emojis (`:tada:`)

## Special Placeholders

The first PR comment supports the `@<username>` placeholder, which will be automatically replaced with the PR author's username.

## How It Works

1. The workflow first checks the PR author in a central job:
   - Determines if they are an external contributor (not in your org or additional users list)
   - For first PR comments, checks if this is their first contribution

2. Based on the event type (opened/ready for review/merged) and author status:
   - Runs only the appropriate comment job
   - Only if the corresponding comment text is not empty

3. Each enabled job posts its specific comment to the PR

This architecture ensures contributor checks run only once per workflow execution, making the process more efficient.

## Security Considerations

This workflow uses `pull_request_target` to ensure it has the necessary permissions to comment on PRs. Since this event has access to secrets, the workflow is designed to only perform safe operations (commenting) and does not check out code from external PRs.

The workflow requires a token with `org:read` permission to check organization membership.

## Integration Testing

For integration testing purposes, we maintain a separate [auto-comments-test](https://github.com/RequestNetwork/auto-comments-test) repository. This repository contains workflows that:

1. Test actual PR events using the reusable workflow
2. Allow manual simulation of different PR events
3. Support testing different branches or versions of the workflow

If you're developing changes to this workflow, you can test them using the integration test repository before merging to the main branch.

## License

MIT
