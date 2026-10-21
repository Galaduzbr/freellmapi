# Tech Advisor — FreeLLMAPI bridge (deployment checklist)

This fork is kept separate from `Galaduzbr/Tech-Advisor`. The goal is to expose an authenticated OpenAI-compatible router for a **backend-only** support agent, never for browsers.

## Security gates before a Render deployment

1. Deploy a **separate** Render web service from this fork, using the root Dockerfile (Docker runtime, port `3001` / `$PORT` compatibility must be verified).
2. Configure `ENCRYPTION_KEY` as a random 32-byte hex secret in Render environment variables. Do not commit it.
3. **Persist `/app/server/data`** across restarts. This application uses SQLite for admin, encrypted provider keys and routing state. A service without a persistent disk must not be treated as production-ready.
4. **Pre-provision the first administrator securely** before exposing the service publicly. Use a secret environment variable or secret file for the declarative config (`FREEAPI_CONFIG_JSON` or `FREEAPI_CONFIG_PATH`), and remove/rotate temporary bootstrap credentials after setup. Do not expose an unclaimed setup screen.
5. Review the Render service plan, disk support and costs before creating the service. Do not assume the free tier supports persistent disks.
6. Keep dashboard/admin routes access-controlled; configure a network/authenticated gateway if public exposure is unavoidable. A unified API key is not a substitute for securing the admin interface.
7. Configure provider keys **in the hosted instance**. Local `localhost:3001` keys and SQLite data do not automatically transfer to Render.
8. Set the Tech Advisor backend's secret `FREELLMAPI_API_KEY` and `FREELLMAPI_URL=https://<service-host>/v1/chat/completions` **only after** the hosted router is secured and healthy. Never ship either key to frontend JavaScript.
9. Set request/time/token budgets, restrict models/providers, and ensure a fail-closed behavior when the router is unavailable.
10. Start with **read-only** diagnostic reports. Do not execute model-generated shell commands, merge PRs or deploy automatically.

## Integration sequence

- Confirm secure, persistent hosted FreeLLMAPI deployment.
- Smoke-test authenticated `/v1/chat/completions` using a dummy prompt (never log the Authorization header).
- Point the Tech Advisor support agent to the hosted URL through backend environment variables.
- Run a real browser-bot report through the support agent; review the Markdown output.
- Only then consider automated, human-reviewed code suggestions.

## Rollback

The FreeLLMAPI service must remain independent of the Tech Advisor web and API services. Disable the bridge integration or remove its URL/key from the backend without interrupting the existing website.
