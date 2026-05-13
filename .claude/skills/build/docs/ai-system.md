# SysForge AI-Native Guidelines

SysForge applications integrate AI not as a gimmick, but as a core capability.

## 1. Tenant-Aware Context
AI prompts MUST be injected with the current `tenant_id` context. The backend must enforce that any data fetched to provide context to the LLM is scoped to that tenant. 

## 2. Rate Limiting and Cost Monitoring
- AI endpoints are expensive. Implement strict rate limiting (e.g., Token Bucket algorithm via Redis).
- Audit all AI requests. Log `tenant_id`, `user_id`, `prompt_tokens`, `completion_tokens`, and `cost`.

## 3. Streaming Support
- The UI must support streaming responses for AI interactions to ensure low perceived latency.
- Use `ai-sdk` (Vercel) on the frontend for smooth streaming integration.

## 4. Resilience
- Implement retry logic with exponential backoff for AI API calls.
- Define fallback models (e.g., fallback from `claude-3-opus` to `claude-3-haiku`) if the primary model is unavailable or rate-limited.
