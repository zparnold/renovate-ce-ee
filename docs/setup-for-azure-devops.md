# Set up Mend Renovate Self-hosted App for Azure DevOps

## Configure Renovate Bot Account on Azure DevOps

Two main parts to Bot setup:

1. **Create the Renovate Bot user account** on Azure DevOps - and get the **PAT**

Personal Access Token (PAT) for the user account will be used by your Renovate server for interacting with your repos on Azure DevOps.

2. **Create Service Hooks (Webhooks)** to respond to activity on the repo.

Without webhooks, Renovate jobs will run only on the configured schedule, or by API call.
Jobs triggered by webhooks jump the queue, and are scheduled to run as soon as possible.

### Step 1a: Renovate Bot user Account

In Azure DevOps, the "Renovate Bot" is not an App or Plugin; it's an Azure DevOps user account that's been given the right permissions on the repository.

Create an Azure DevOps user account to act as the "Renovate Bot".

> [!NOTE]
>
> You should use a dedicated "bot account" for Renovate, instead of using someone's personal user account.
>
> Apart from reducing the chance of conflicts, it is better for teams if the actions they see from Renovate are clearly marked as coming from a dedicated bot account and not from a teammate's account, which could be confusing at times.
> e.g. Did the bot automerge that PR, or did a human do it?

#### Self-hosted Azure DevOps Server:
- If you are running your own instance of Azure DevOps Server, it's suggested to name the account "Renovate Bot" with username "renovate-bot".

#### Azure DevOps Services (Cloud):
- If your repos are on Azure DevOps Services (dev.azure.com), create a new user account in your Azure DevOps organization.
You will need a unique name for the bot, for instance "renovate-bot".

### Step 1b: Generate a Personal Access Token (PAT)

Once the account is created, [create a Personal Access Token](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate) for the new Renovate user account.

Assign the following permissions:
  * **Code**: Read & Write
  * **Pull Request Threads**: Read & Write
  * **User Profile**: Read (to read user information)

> [!NOTE]
>
> It is recommended to set the PAT expiration to the maximum allowed duration (typically 1 year) and set up a reminder to rotate it before expiration.

Keep the PAT handy for the Configuration of the Renovate Self-Hosted App to set `MEND_RNV_AZURE_DEVOPS_PAT`.

### Step 2: Add Service Hooks (Webhooks)

Service Hooks allow Azure DevOps to ping your Renovate server whenever an activity occurs on the designated repositories or projects.
Activities like a new commit, a merged PR, a change to a package file, etc will trigger Renovate to run a new job on that repo.
Renovate can also respond to checkbox activities in PRs and the Dependency Dashboard.

> [!NOTE]
>
> You can still run Renovate without webhooks.
> - Renovate Jobs will run on a schedule, which is highly configurable.
> - The Admin APIs can be used to trigger App Sync and to run a Renovate job on a single repo.

You can enable webhooks on your Azure DevOps repos manually, or with automation.

#### Option: Automatic webhook creation

When enabled, will automatically install webhooks for all new repos that are enabled with Renovate. Attempts are made to remove webhooks when repos are uninstalled.

> [!NOTE]
>
> Currently, if you add the configuration for Webhooks later to an existing setup, webhooks will not be added to repositories already registered in Renovates DB.
>
> As a workaround, you can uninstall and reinstall repos that you want to add webhooks to. Alternatively, if you delete the database, all repos will be freshly re-installed and webhooks will be created for them.

Webhook installation requires an admin user account that has appropriate permissions to manage service hooks on the repos.

> [!NOTE]
>
> This admin account used for webhooks can be the repo owner account, or it can be another account that has more limited access to the repos.
>
> The Renovate user account can be used if it has the necessary permissions to manage service hooks.

To enable automatic webhook creation:

Set `MEND_RNV_WEBHOOK_URL`:
- When set, webhooks will be installed on repos when Renovate is enabled.
- Set the webhook URL to point to the Renovate server url followed by `/webhook`. (e.g. `http://renovate.yourcompany.com:8080/webhook` or `https://1.2.3.4/webhook`)

Set `MEND_RNV_ADMIN_TOKEN`: [Optional]
- Could be repo owner account, or special high-privilege account.
- Defaults to the primary Renovate user PAT when not provided.
- Important: Webhooks will only be installed on repos that the account has appropriate permissions to.

#### Option: Manually add webhooks

Add a **Service Hook** to each Azure DevOps Project that you want webhooks triggered on.

You can add service hooks at the organization or project level to cover all repos in that scope.
- This is an easy way to cover webhooks for all repos in a project.
- Webhooks coming from Repositories that haven't enabled Renovate will be ignored.

**Set service hook properties as shown below:**

##### To create a service hook manually:

1. Navigate to your Azure DevOps project
2. Go to **Project Settings** → **Service Hooks**
3. Click **Create subscription**
4. Select **Web Hooks** as the service type
5. Configure the trigger:
   - **Trigger on**: Select the following events:
     * **Code pushed**
     * **Pull request created**
     * **Pull request updated**
     * **Pull request merge attempted**

6. Configure the action:
   - **URL**: Set the webhook URL to point to the Renovate server url followed by `/webhook`. (e.g. `http://renovate.yourcompany.com:8080/webhook` or `https://1.2.3.4/webhook`)
   - **HTTP headers**: Add a header for authorization if needed
   - **Basic authentication username**: (optional)
   - **Basic authentication password**: Set to the same value configured for `MEND_RNV_WEBHOOK_SECRET` (defaults to `renovate`)

> [!TIP]
>
> Renovate's webhook listener binds to port 8080 inside its container, but you can map it (using Docker) to whatever external port you require, including port 80.

## Run Mend Renovate Self-hosted App

You can run Mend Renovate Self-hosted App from a Docker command line prompt, or by using a Docker Compose file. Examples are provided in the links below.

**Example Docker Compose files:**

- [Mend Renovate Community Edition](../examples/docker-compose/docker-compose-renovate-community.yml)
- [Mend Renovate Enterprise Edition](../examples/docker-compose/docker-compose-renovate-enterprise.yml)

> [!NOTE]
>
> Some configuration of environment variables will be required inside the Docker Compose files.
>
> Essential configuration options are shown below. For a full list of configurable variables, see [Configuration Options](configuration-options.md).

## Configure Environment Variables

### Essential Configuration for Mend Renovate Server

**`MEND_RNV_ACCEPT_TOS`**: Set this environment variable to `y` to consent to [Mend's Terms of Service](https://www.mend.io/terms-of-service/).

**`MEND_RNV_LICENSE_KEY`**: Provide a valid license key for Renovate Community Edition or Enterprise Edition

> [!Note]
>
> To run Renovate Community Edition with **up to 10 repositories**, you can use this unregistered license key:
>
> `eyJsaW1pdCI6IjEwIn0=.30440220457941b71ea8eb345c729031718b692169f0ce2cf020095fd328812f4d7d5bc1022022648d1a29e71d486f89f27bdc8754dfd6df0ddda64a23155000a61a105da2a1`
>
> For a free license key for an **unrestricted number of repositories** on Renovate Community Edition, register with the form on the [Renovate Community Edition web page](https://www.mend.io/mend-renovate-community/).
>
> For an Enterprise license key, contact Mend at http://mend.io.

**`MEND_RNV_PLATFORM`**: Set this to `azure-devops`.

**`MEND_RNV_ENDPOINT`**: This is the API endpoint for your Azure DevOps organization. e.g. `https://dev.azure.com/{organization}/` for Azure DevOps Services, or `https://azuredevops.company.com/{organization}/` for Azure DevOps Server. Include the trailing slash.

**`MEND_RNV_SERVER_PORT`**: The port on which the server listens for webhooks and api requests. Defaults to 8080.

**`MEND_RNV_AZURE_DEVOPS_PAT`**: Personal Access Token (PAT) for the Azure DevOps bot account.

**`MEND_RNV_ADMIN_API_ENABLED`**: Set to 'true' to enable Admin APIs. Defaults to 'false'.

**`MEND_RNV_SERVER_API_SECRET`**: Required if Admin APIs are enabled, or if running Enterprise Edition.

**`MEND_RNV_WEBHOOK_SECRET`**: Must match the secret sent by the Azure DevOps webhooks. Defaults to 'renovate'.

**`MEND_RNV_WEBHOOK_URL`**: [Optional] Set to the URL of your webhook handler to enable automatic webhook creation. (eg. `http://renovate.yourcompany.com:8080/webhook`)

**`MEND_RNV_ADMIN_TOKEN`**: [Optional] Used when automatically adding webhooks. Provide a PAT for a user with appropriate permissions to manage service hooks.

**`GITHUB_COM_TOKEN`**: A Personal Access Token for a user account on github.com

**Additional Configuration options**

For further details and a list of all available options, see the [Configuration Options](configuration-options.md) page.

### Renovate CLI Configuration

Renovate CLI functionality can be configured using environment variables (e.g. `RENOVATE_XXXXXX`) or via a `config.js` file mounted to `/usr/src/app/config.js` inside the Mend Renovate container.

**npm Registry**

If using your own npm registry, you may find it easiest to update your Docker Compose file to include a volume that maps an `.npmrc` file to `/home/ubuntu/.npmrc`. The RC file should contain `registry=...` with the registry URL your company uses internally. This will allow Renovate to find shared configs and other internally published packages.
