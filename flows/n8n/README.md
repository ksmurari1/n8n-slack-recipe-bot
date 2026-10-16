# Workflow export notes

`slack-recipe-bot.json` is a sanitized **template** based on the repository's documented Slack Recipe Bot flow.

Before running it:

1. Import it into your n8n instance.
2. Select your own Slack credential on the Slack Trigger and Send a Message nodes.
3. Select your own OpenAI credential on the OpenAI Chat Model node.
4. Replace `REPLACE_WITH_YOUR_SLACK_CHANNEL_ID` with a channel ID available to your bot.
5. Review the Slack Trigger, expressions, model selection, and node compatibility for your n8n version.
6. Save, configure Slack Event Subscriptions, publish, and test recipe/greeting/unclear-input cases.

Credentials, account-specific IDs, private channel ID, and instance metadata are intentionally excluded. This template is not connected to the author's workspace and is not a drop-in production workflow. The original tested workflow may have different node versions or settings; preserve your own working export privately.
