# Slack Recipe Bot: troubleshooting

Use this guide to diagnose common issues in the Slack → n8n → OpenAI → Slack workflow.

## Quick reference

| Symptom | Likely cause | Fix |
|---|---|---|
| Slack Trigger channel list is empty | Missing scopes, app not reinstalled, or credential not refreshed | Verify scopes, reinstall the app if scopes changed, refresh the n8n credential, or configure the channel by ID |
| Slack says the Request URL did not respond to the challenge | Socket Mode is enabled, the wrong URL is configured, or the workflow/listener is not active | Turn Socket Mode off; use the Test URL only while the test listener is waiting, or publish and use the Production URL |
| Messages do not trigger a run | Event subscription or channel membership is missing | Enable Event Subscriptions, subscribe to `message.channels`, save changes, and invite the bot to the channel |
| Workflow works in test but not after leaving the editor | Slack still points to the Test URL or workflow is unpublished | Publish the workflow and set Slack's Request URL to the Production URL |
| Reply contains `{{ $json.recipe }}` literally | Field is in Fixed mode rather than Expression mode | Switch the field to Expression mode and confirm the `fx` indicator |
| Bot repeatedly answers itself | The If guard is missing, incorrectly configured, or its false branch continues | Check that both `bot_id` empty and `subtype` empty conditions use AND, and only the true output continues |
| Reply is empty or sent to the wrong channel | Channel mapping was lost after the LLM chain, or an expression is incorrect | Preserve `channel` in Edit Fields and use `{{ $('Edit Fields').item.json.channel }}` in the Slack destination |
| OpenAI node fails with authentication/quota error | Credential, billing, or model access issue | Verify the OpenAI credential, account billing, and model availability |
| Bot invents a recipe for a greeting or gibberish | Prompt-based classification is imperfect | Keep the ordered prompt rules; add deterministic validation before the LLM if stronger handling is needed |
| Markdown appears with literal double asterisks | Slack formatting differs from standard Markdown | Follow the prompt's Slack formatting instruction: single asterisks for bold and bullet characters |
| Duplicate replies appear | Slack event retry or duplicate delivery | Add event-level idempotency/deduplication; this is not implemented in the basic workflow |

## Event URL verification

Slack verifies a Request URL by sending a challenge. Check that the URL is complete and corresponds to a live n8n endpoint. For production, publish the workflow and use the Production URL. For testing, start the n8n test listener first and use the Test URL. Restore the Production URL after debugging.

## Inspect an execution

1. Open the n8n **Executions** view.
2. Open the failed or unexpected run.
3. Find the first node with an error or unexpected output.
4. Check the Slack Trigger payload, If condition result, Edit Fields output, LLM response, and Slack send node input.
5. Avoid sharing screenshots containing tokens, private messages, or sensitive workspace information.

## Related documents

- [Architecture](07-slack-recipe-bot-architecture.md)
- [Workflow design](08-slack-recipe-bot-workflow-design.md)
- [Setup and operations](09-slack-recipe-bot-setup-and-operations.md)
