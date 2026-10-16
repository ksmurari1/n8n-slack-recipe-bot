# Slack Recipe Bot: troubleshooting

Problems that can occur when connecting Slack to n8n, with the cause and the fix for each.

## Quick reference

| Symptom | Likely cause | Fix |
|---|---|---|
| Channel dropdown in the Slack Trigger is empty, red warning on the node | The credential cannot list channels yet (missing scopes, app not reinstalled, or token not refreshed) | Add the scopes, reinstall the app, update the token in the n8n credential. Or switch **From list** to **By ID** and paste the channel ID |
| Orange warning "Set up a webhook in your Slack app" | Slack has not been told where to send events | Configure **Event Subscriptions** with the n8n webhook URL |
| Slack says the URL "didn't respond with the challenge parameter" | Socket Mode is on, the node is not listening (Test URL), or the workflow is not published (Production URL) | Turn Socket Mode off. For the Test URL, click Execute first and then Retry. For the Production URL, publish first and then Retry |
| "HTTP error" next to the Request URL after pasting the Production URL | The workflow is not published, so the production webhook does not exist | Publish the workflow, then click **Retry** in Slack |
| Messages posted in Slack get no reaction while using *Execute* | The Test URL listens for one event only, and verification used it up | Click **Execute** again before each message, or test on the published workflow |
| Same, but Slack's Request URL is the Production URL and the workflow is unpublished | Slack sends events to the Production URL, which is down while unpublished | Publish and test there, or switch the Request URL to the Test URL while debugging |
| Reply contains `{{ $json.recipe }}` literally | The field is in **Fixed** mode, not **Expression** | Hover the field and choose **Expression** (the **fx** badge appears) |
| "Problem running workflow: n8n can't listen for test executions at the same time as production" | *Execute* was clicked while the workflow is published | Do not use Execute while published. Test from Slack and read the Executions tab, or unpublish first |
| The bot answers itself repeatedly | The loop guard is missing or not connected | Make sure the `If` node (`bot_id` empty AND `subtype` empty) sits right after the trigger and only its **true** output continues |
| Reply shows `**bold**` with literal asterisks | Slack uses single-asterisk markup | Keep the Slack formatting line in the prompt (single asterisks, "•" bullets, no headings) |
| Reply is empty or the Slack node fails on the channel | The channel expression is wrong, or the chain output was used for the channel | Use `{{ $('Edit Fields').item.json.channel }}` for the channel and `{{ $json.text }}` for the message |
| LLM node fails with an authentication or quota error | Missing key or no billing credit on the OpenAI account | Check the OpenAI credential and the account's billing status |
| Greetings or gibberish produce an invented ingredient list | The prompt has no rule for non-recipe input | Add the ordered response rules from the [workflow design](08-slack-recipe-bot-workflow-design.md) |

## Detailed notes

### Event URL verification

Slack sends a request containing a `challenge` value as soon as a Request URL is entered, and expects it echoed back. That only works when:

1. Socket Mode is off;
2. the URL is complete (these URLs are long and are easy to truncate when pasting);
3. the matching n8n webhook is live: the Test URL while the editor is listening, or the Production URL while the workflow is published.

If verification fails, fix the cause and click **Retry**. No change to the URL is needed.

### One event per test run

The Test URL answers a single request. The verification request itself counts, so after Slack verifies the Test URL, the listener stops. Click **Execute** again before posting the real test message.

### Test versus production

Slack can only point at one Request URL at a time. The Production URL is the one to leave configured. Switch to the Test URL only for short debugging sessions, and switch back afterwards.

### Seeing what happened

On a published workflow, open the **Executions** tab and select the run. Each node shows its input and output. Look for a red node first: Slack Trigger and `If` issues are usually about the event, and Basic LLM Chain issues are usually about the OpenAI credential or the prompt.

## Related documents

- [Architecture](07-slack-recipe-bot-architecture.md)
- [Workflow design](08-slack-recipe-bot-workflow-design.md)
- [Setup and operations](09-slack-recipe-bot-setup-and-operations.md)
