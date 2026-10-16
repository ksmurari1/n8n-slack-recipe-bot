# Slack Recipe Bot: setup and operations

Step-by-step setup of the bot, how to verify it, and how to operate it day to day.

## Prerequisites

- A Slack workspace where you can create and install an app.
- An n8n instance reachable from the internet over HTTPS (self-hosted or n8n Cloud), with its public base URL configured so webhook URLs are not `localhost`.
- An OpenAI API key with billing enabled (an API key is separate from a ChatGPT subscription).
- A public Slack channel for the bot (a dedicated one is recommended).

## 1. Create the Slack app

1. Open the Slack API site, choose **Create New App**, and create it from scratch in your workspace.
2. Go to **OAuth & Permissions** and add these **Bot Token Scopes**:

   | Scope | Purpose |
   |---|---|
   | `channels:history` | Read messages in public channels the bot has joined |
   | `channels:read` | Read basic channel information |
   | `chat:write` | Post replies as the bot |

   For private channels also add `groups:history` and `groups:read`.
3. Click **Install to Workspace** (or **Reinstall to Workspace** after changing scopes). Scope changes only take effect after reinstalling.
4. Copy the **Bot User OAuth Token** (it starts with `xoxb-`).
5. In Slack, invite the bot to the channel: `/invite @<bot name>`.
6. Copy the channel ID from the channel details (it starts with `C`).

## 2. Create the n8n credentials

1. Add a **Slack API** credential and paste the bot token.
2. Add an **OpenAI** credential and paste the API key. n8n should report a successful connection test.

Keep keys inside credentials. Do not paste them into node parameters or commit them anywhere.

## 3. Build the workflow

Import the sanitized template at `../flows/n8n/slack-recipe-bot.json` or create the nodes as described in [workflow design](08-slack-recipe-bot-workflow-design.md):

`Slack Trigger` → `If` → `Edit Fields` → `Basic LLM Chain` (with `OpenAI Chat Model`) → `Slack: Send a message`

Check that every expression field shows the **fx** badge. A field left in *Fixed* mode sends the literal text `{{ $json.recipe }}` to Slack instead of the value.

## 4. Connect Slack events to n8n

1. In the Slack app, open **Socket Mode** and make sure it is **off**.
2. Open **Event Subscriptions** and switch **Enable Events** on.
3. In the n8n Slack Trigger node, open **Webhook URLs** and copy the **Production URL**.
4. Paste it into **Request URL** in Slack. It must show **Verified**.
5. Under **Subscribe to bot events**, add `message.channels` (and `message.groups` for private channels).
6. Click **Save Changes**, and reinstall the app if Slack asks.

Slack can only verify the Production URL while the workflow is published. If verification fails, publish the workflow first, then click **Retry** in Slack.

## 5. Publish

Click **Save**, then **Publish** in n8n. From then on the workflow answers messages without anyone keeping the editor open.

Every later change needs **Publish** again before live traffic uses it.

## 6. Verify

Post these messages in the channel, one at a time:

| Message | Expected reply |
|---|---|
| `Paneer Butter Masala` | Bulleted ingredient list with quantities for 4 servings |
| `pizza` | Bulleted ingredient list |
| `hello` | "Could you please specify the recipe or dish you want the ingredient list for?" |
| `asdfgh` | "I couldn't understand that recipe name. Please type a dish name, for example: Paneer Butter Masala." |

Then open the **Executions** tab in n8n and confirm:

- each human message produced exactly one successful run, and
- the bot's own replies did not start extra runs that continued past the `If` node.

## Operations

### Monitoring

- Use the **Executions** tab to see every run, its input message and its output. A failed run shows the failing node in red.
- On a published workflow the canvas does not update live. Open a run from the Executions list to inspect its data.

### Changing the workflow

1. Edit and save.
2. Publish again.
3. Re-run the verification messages above.

### Debugging on the canvas

The Slack Trigger cannot listen for test events while the workflow is published. To use *Execute* on the canvas:

1. Unpublish the workflow.
2. Click **Execute** so the node waits for an event.
3. Switch Slack's Request URL to the **Test URL**, wait for **Verified**, and click **Execute** again.
4. Post one message. The Test URL handles a single event per execution.
5. When finished, switch Slack's Request URL back to the **Production URL**, publish, and wait for **Verified**.

For most checks, publishing and reading the Executions tab is simpler.

### Cost and noise control

- Use a dedicated channel so only recipe requests reach the LLM.
- Keep the `If` guard in place; it stops bot and system messages before any model call.
- Use the model configured and tested in your n8n instance; the exported workflow reviewed for this package specifies `gpt-4.1-mini`. Check availability and cost in your OpenAI account.

### Rotating credentials

1. Generate the new token or key in Slack or OpenAI.
2. Update the matching n8n credential.
3. Save, then re-run a verification message.

## Exporting the workflow for version control

1. In the workflow, open the **⋯** menu next to the workflow name and choose **Download**.
2. Open the JSON in a text editor and search for `xoxb`, `sk-` and any other key patterns. n8n exports credential names and IDs, not secret values, but always check.
3. Use a placeholder channel ID in the public template and configure your own channel after import. Remove unnecessary instance-specific metadata before publishing.
4. Save the sanitized file as `flows/n8n/slack-recipe-bot.json` and commit it. The package template has credentials removed and is not preconnected to your workspace.

## Related documents

- [Architecture](07-slack-recipe-bot-architecture.md)
- [Workflow design](08-slack-recipe-bot-workflow-design.md)
- [Troubleshooting](10-slack-recipe-bot-troubleshooting.md)
