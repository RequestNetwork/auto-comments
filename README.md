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
      token: ${{ secrets.GH_PAT_AUTO_COMMENTS }}
```

## How to Create a Personal Access Token (PAT)

Creating a Personal Access Token (PAT) for GitHub is a straightforward process:

1. **Log in to your GitHub account**
2. **Go to your Settings**:
   - Click on your profile photo in the top-right corner
   - Select "Settings" from the dropdown menu
3. **Navigate to Developer settings**:
   - Scroll down to the bottom of the sidebar
   - Click on "Developer settings"
4. **Select Personal access tokens**:
   - Click on "Personal access tokens"
   - Choose "Fine-grained tokens" or "Tokens (classic)" depending on your needs

### Required Permissions for Fine-grained Tokens

When creating a fine-grained personal access token, you'll need to configure the following permissions:

**Repository permissions:**
- **Contents**: Read (to access repository content)
- **Pull requests**: Read and Write (to read PR details and add comments)
- **Issues**: Read and Write (for commenting, as GitHub treats PR comments as issue comments)
- **Metadata**: Read (required for most API operations)

**Organization permissions:**
- **Members**: Read (to check if the PR author is an organization member)

5. **Generate a new token**:
   - Click "Generate new token"
6. **Configure token settings**:
   - Add a descriptive note to remember what this token is for (e.g., "Auto Comments Workflow")
   - Set an expiration date (consider security implications)
   - Select the repositories that will use this token
   - Select the permissions listed above
7. **Generate the token**:
   - Click "Generate token" at the bottom
8. **Copy your token**:
   - **IMPORTANT**: Copy the token immediately as you won't be able to see it again

After generating the token, you need to add it as a repository secret:

### Adding Repository or Organization Secrets

You can configure this workflow with either a repository-level secret or an organization-level secret:

#### For Repository-Level Secret:
1. Go to your repository
2. Click on "Settings"
3. In the left sidebar, click on "Secrets and variables" → "Actions"
4. Click "New repository secret"
5. Name the secret `GH_PAT_AUTO_COMMENTS`
6. Paste your token value
7. Click "Add secret"

#### For Organization-Level Secret:
1. Go to your organization's main page
2. Click on "Settings"
3. In the left sidebar, click on "Secrets and variables" → "Actions"
4. Click "New organization secret"
5. Name the secret `GH_PAT_AUTO_COMMENTS`
6. Choose your repository access policy (all repositories or select repositories)
7. Paste your token value
8. Click "Add secret"

Both repository-level and organization-level secrets are accessed the same way in workflows: `${{ secrets.GH_PAT_AUTO_COMMENTS }}`.

### References
For more detailed information about GitHub tokens and permissions, refer to the [GitHub documentation on fine-grained personal access tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token#creating-a-fine-grained-personal-access-token).

For more information about using secrets in GitHub Actions, see [GitHub's documentation on using secrets in GitHub Actions](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions).

## Workflow Updates and Versioning

This workflow uses a reference to the branch (`@main`) rather than a specific version tag. This means:

- **Automatic Updates**: When the workflow code is updated in the `main` branch, all repositories referencing it will automatically use the latest version without requiring any changes in those repositories.
- **Breaking Changes**: Be cautious when making changes to the workflow in the `main` branch, as they will immediately affect all dependent repositories. Test significant changes thoroughly before merging them into `main`.

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
| `token` | GitHub token with necessary permissions (see above) | No | `github.token` |

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

You can use most Markdown syntax in your comment messages, including:

- Headings (`# Heading`)
- Lists (`- Item`)
- Formatting (**bold**, *italic*)
- Links (`[text](url)`)
- Images
- Tables
- Emojis (`:tada:`)

> **Note:** Triple backtick code blocks are not supported due to technical limitations with the GitHub Actions script environment. If you need to include code examples, consider using inline code with single backticks or HTML alternatives.

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

For integration testing purposes, we maintain a separate [auto-comments-test](https://github.com/RequestNetwork/auto-comments-test) repository. This repository provides a real-world environment for testing the workflow.

If you're developing changes to this workflow, you should test them using the integration test repository before merging to the main branch.

## License

MIT
