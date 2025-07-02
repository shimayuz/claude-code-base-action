# Claude Max Code Base Action

> **🎉 Use Your Claude Max Subscription in GitHub Actions!**
>
> This is a fork of [anthropics/claude-code-base-action](https://github.com/anthropics/claude-code-base-action) that adds OAuth authentication support, enabling you to use your **Claude Max subscription** with GitHub Actions workflows.
>
> **Key Feature:** Authenticate with your Claude Max subscription credentials instead of requiring API keys, making it accessible to all Claude Max subscribers.

This GitHub Action allows you to run [Claude Code](https://www.anthropic.com/claude-code) within your GitHub Actions workflows. You can use this to build any custom workflow on top of Claude Code.

For simply tagging @claude in issues and PRs out of the box, [check out the Claude Code action and GitHub app](https://github.com/anthropics/claude-code-action).

## Usage

Add the following to your workflow file:

```yaml
# Using a direct prompt with OAuth
- name: Run Claude Code with direct prompt
  uses: shimayuz/claude-code-base-action@main
  with:
    prompt: "Your prompt here"
    allowed_tools: "Bash(git:*),View,GlobTool,GrepTool,BatchTool"
    use_oauth: "true"
    claude_access_token: ${{ secrets.CLAUDE_ACCESS_TOKEN }}
    claude_refresh_token: ${{ secrets.CLAUDE_REFRESH_TOKEN }}
    claude_expires_at: ${{ secrets.CLAUDE_EXPIRES_AT }}

# Using a prompt from a file with API Key
- name: Run Claude Code with prompt file
  uses: shimayuz/claude-code-base-action@main
  with:
    prompt_file: "/path/to/prompt.txt"
    allowed_tools: "Bash(git:*),View,GlobTool,GrepTool,BatchTool"
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}

# Using custom system prompts
- name: Run Claude Code with custom system prompt
  uses: shimayuz/claude-code-base-action@main
  with:
    prompt: "Build a REST API"
    system_prompt: "You are a senior backend engineer. Focus on security, performance, and maintainability."
    allowed_tools: "Bash(git:*),View,GlobTool,GrepTool,BatchTool"
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}

# Using fallback model for handling API errors
- name: Run Claude Code with fallback model
  uses: shimayuz/claude-code-base-action@main
  with:
    prompt: "Review and fix TypeScript errors"
    model: "claude-opus-4-20250514"
    fallback_model: "claude-sonnet-4-20250514"
    allowed_tools: "Bash(git:*),View,GlobTool,GrepTool,BatchTool"
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

## Inputs

| Input                  | Description                                                                                       | Required | Default                      |
| ---------------------- | ------------------------------------------------------------------------------------------------- | -------- | ---------------------------- |
| `prompt`               | The prompt to send to Claude Code                                                                 | No\*     | ''                           |
| `prompt_file`          | Path to a file containing the prompt to send to Claude Code                                       | No\*     | ''                           |
| `allowed_tools`        | Comma-separated list of allowed tools for Claude Code to use                                      | No       | ''                           |
| `disallowed_tools`     | Comma-separated list of disallowed tools that Claude Code cannot use                              | No       | ''                           |
| `max_turns`            | Maximum number of conversation turns (default: no limit)                                          | No       | ''                           |
| `mcp_config`           | Path to the MCP configuration JSON file                                                           | No       | ''                           |
| `system_prompt`        | Override system prompt                                                                            | No       | ''                           |
| `append_system_prompt` | Append to system prompt                                                                           | No       | ''                           |
| `claude_env`           | Custom environment variables to pass to Claude Code execution (YAML multiline format)             | No       | ''                           |
| `model`                | Model to use (provider-specific format required for Bedrock/Vertex)                               | No       | 'claude-4-0-sonnet-20250219' |
| `anthropic_model`      | DEPRECATED: Use 'model' instead                                                                   | No       | 'claude-4-0-sonnet-20250219' |
| `fallback_model`       | Enable automatic fallback to specified model when default model is overloaded                     | No       | ''                           |
| `timeout_minutes`      | Timeout in minutes for Claude Code execution                                                      | No       | '10'                         |
| `anthropic_api_key`    | Anthropic API key (required for direct Anthropic API)                                             | No       | ''                           |
| `use_bedrock`          | Use Amazon Bedrock with OIDC authentication instead of direct Anthropic API                       | No       | 'false'                      |
| `use_vertex`           | Use Google Vertex AI with OIDC authentication instead of direct Anthropic API                     | No       | 'false'                      |
| `use_oauth`            | Use Claude AI OAuth authentication instead of API key                                             | No       | 'false'                      |
| `claude_access_token`  | Claude AI OAuth access token (required when use_oauth is true)                                    | No       | ''                           |
| `claude_refresh_token` | Claude AI OAuth refresh token (required when use_oauth is true)                                   | No       | ''                           |
| `claude_expires_at`    | Claude AI OAuth token expiration timestamp (required when use_oauth is true)                      | No       | ''                           |
| `use_node_cache`       | Whether to use Node.js dependency caching (set to true only for Node.js projects with lock files) | No       | 'false'                      |

\*Either `prompt` or `prompt_file` must be provided, but not both.

## Outputs

| Output           | Description                                                |
| ---------------- | ---------------------------------------------------------- |
| `conclusion`     | Execution status of Claude Code ('success' or 'failure')   |
| `execution_file` | Path to the JSON file containing Claude Code execution log |

## Using Cloud Providers

You can authenticate with Claude using any of these four methods:

1. Direct Anthropic API (default) - requires API key
2. Claude AI OAuth authentication - requires OAuth tokens
3. Amazon Bedrock - requires OIDC authentication and automatically uses cross-region inference profiles
4. Google Vertex AI - requires OIDC authentication

## Security Best Practices

**⚠️ IMPORTANT: Never commit API keys or OAuth tokens directly to your repository! Always use GitHub Actions secrets.**

### OAuth Authentication Setup

To use OAuth authentication with your Claude Max Subscription Plan:

0. Login into Claude Code with your Claude Max Subscription with `/login`:

   - Lookup your access token, refresh token and expires at values: `cat ~/.claude/.credentials.json`

1. Add your OAuth credentials as repository secrets:

   - Go to your repository's Settings
   - Navigate to "Secrets and variables" → "Actions"
   - Add the following secrets:
     - `CLAUDE_ACCESS_TOKEN` - Your Claude AI OAuth access token
     - `CLAUDE_REFRESH_TOKEN` - Your Claude AI OAuth refresh token
     - `CLAUDE_EXPIRES_AT` - Token expiration timestamp (Unix timestamp in seconds)

2. Reference the secrets in your workflow:
   ```yaml
   use_oauth: "true"
   claude_access_token: ${{ secrets.CLAUDE_ACCESS_TOKEN }}
   claude_refresh_token: ${{ secrets.CLAUDE_REFRESH_TOKEN }}
   claude_expires_at: ${{ secrets.CLAUDE_EXPIRES_AT }}
   ```

### API Key Authentication Setup (Legacy)

To use API key authentication:

1. Add your API key as a repository secret:

   - Go to your repository's Settings
   - Navigate to "Secrets and variables" → "Actions"
   - Click "New repository secret"
   - Name it `ANTHROPIC_API_KEY`
   - Paste your API key as the value

2. Reference the secret in your workflow:
   ```yaml
   anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
   ```

## License

This project is licensed under the MIT License—see the LICENSE file for details.