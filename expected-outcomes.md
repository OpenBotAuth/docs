# Expected Outcomes

1. **Friction free crawling:** Large agent providers like OpenAI require friction-free authentication paths that allow their compliant bots to access sites without solving CAPTCHAs or circumventing security measures.
2. **Non-spoofable identity:** The specification must provide non-repudiable proof of bot identity and actions, creating a verifiable record that can serve as legal evidence when disputes arise.
3. **Intent declaration:** Agents must signal their purpose—whether data will be used for training language models, search indexing, or read-only queries—giving site owners informed control over how their content is used.
4. **No discrimination:** Authentication and pricing should not favor bots from major corporations over independent developers, preventing discriminatory treatment where Google or Microsoft bots receive preferential access or rates.
5. **Privacy-preserving anonymous lanes:** The protocol must accommodate anonymous bot traffic with reasonable rate limits, respecting the foundational internet principle that not all automated access requires full identification.
6. **Open verification standard:** No single entity like Cloudflare should monopolize bot authentication and blocking decisions, ensuring the system remains open and resistant to centralized control.
7. **Runtime Attenuation:** Origin can dynamically adjust bot permissions during an active session—restricting or expanding access in real-time based on observed behavior, declared intent, and current context without forcing re-authentication.
