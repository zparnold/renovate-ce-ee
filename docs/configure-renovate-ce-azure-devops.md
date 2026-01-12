# Configure Renovate for Azure DevOps

This guide explains how to configure Renovate to work with Azure DevOps repositories.

## Prerequisites

Before you begin, ensure you have:
- An Azure DevOps organization with repositories you want to manage
- A dedicated bot user account in Azure DevOps
- A Personal Access Token (PAT) with appropriate permissions

## Quick Start

1. **Set up the Bot Account**: Create a dedicated user account in Azure DevOps for Renovate
2. **Generate a PAT**: Create a Personal Access Token with Code (Read & Write) and Pull Request Threads (Read & Write) permissions
3. **Configure Renovate**: Set the required environment variables
4. **Set up Webhooks**: Configure service hooks to trigger Renovate on code changes

## Configuration Variables

### Required Variables

**`MEND_RNV_PLATFORM`**
- Set to: `azure-devops`

**`MEND_RNV_ENDPOINT`**
- For Azure DevOps Services: `https://dev.azure.com/{organization}/`
- For Azure DevOps Server: `https://azuredevops.company.com/{organization}/`
- Include the trailing slash

**`MEND_RNV_AZURE_DEVOPS_PAT`**
- Personal Access Token for the Renovate bot account
- Required permissions:
  - Code: Read & Write
  - Pull Request Threads: Read & Write
  - User Profile: Read

### Optional Variables

**`MEND_RNV_WEBHOOK_SECRET`**
- Secret used to authenticate webhook requests
- Default: `renovate`

**`MEND_RNV_WEBHOOK_URL`**
- URL where webhooks should be sent
- Format: `http://renovate.yourcompany.com:8080/webhook`
- When set, enables automatic webhook creation

**`MEND_RNV_ADMIN_TOKEN`**
- PAT with permissions to manage service hooks
- Used for automatic webhook creation
- Defaults to `MEND_RNV_AZURE_DEVOPS_PAT` if not provided

## Webhook Configuration

### Automatic Webhook Creation

Set `MEND_RNV_WEBHOOK_URL` to enable automatic webhook creation for newly enabled repositories.

### Manual Webhook Creation

To manually create webhooks:

1. Navigate to **Project Settings** → **Service Hooks**
2. Click **Create subscription**
3. Select **Web Hooks**
4. Configure triggers for:
   - Code pushed
   - Pull request created
   - Pull request updated
   - Pull request merge attempted
5. Set the URL to your Renovate server's webhook endpoint
6. Set authentication password to match `MEND_RNV_WEBHOOK_SECRET`

## Permissions

### Bot Account Permissions

The Renovate bot account needs:
- **Contributor** access to repositories it manages
- Ability to create and update pull requests
- Ability to read repository contents

### Admin Account Permissions (for automatic webhook creation)

The admin account (specified by `MEND_RNV_ADMIN_TOKEN`) needs:
- Permissions to manage service hooks at the project or organization level

## Example Configuration

```bash
# Basic Azure DevOps Configuration
MEND_RNV_PLATFORM=azure-devops
MEND_RNV_ENDPOINT=https://dev.azure.com/myorganization/
MEND_RNV_AZURE_DEVOPS_PAT=your_personal_access_token_here
MEND_RNV_WEBHOOK_SECRET=renovate

# Optional: Enable automatic webhook creation
MEND_RNV_WEBHOOK_URL=http://renovate.mycompany.com:8080/webhook
MEND_RNV_ADMIN_TOKEN=admin_user_pat_here

# Required for license
MEND_RNV_ACCEPT_TOS=y
MEND_RNV_LICENSE_KEY=your_license_key_here
```

## Troubleshooting

### Authentication Issues

If Renovate cannot authenticate with Azure DevOps:
- Verify the PAT is valid and not expired
- Ensure the PAT has the correct permissions (Code: Read & Write, Pull Request Threads: Read & Write)
- Check that the bot account has access to the repositories

### Webhook Issues

If webhooks are not triggering Renovate jobs:
- Verify the webhook URL is accessible from Azure DevOps
- Check that the webhook secret matches `MEND_RNV_WEBHOOK_SECRET`
- Ensure the service hooks are configured for the correct events
- Review Renovate server logs for webhook receipt

### Repository Discovery Issues

If Renovate is not discovering repositories:
- Verify the bot account has **Contributor** access to the repositories
- Check that the `MEND_RNV_ENDPOINT` is correctly formatted with the trailing slash
- Review the App Sync logs to see which repositories are being discovered

## Additional Resources

- [Azure DevOps REST API Documentation](https://learn.microsoft.com/en-us/rest/api/azure/devops/)
- [Azure DevOps Personal Access Tokens](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate)
- [Mend Renovate Configuration Options](configuration-options.md)
