# Bulbuly Bot Builder Pro — चरण 1: स्कीमा, API डिज़ाइन, फ़ोल्डर स्ट्रक्चर

## स्टैक (Bulbuly पर)
- वेब ऐप + PWA (Android पर इंस्टॉल योग्य; बाद में Play Store हेतु TWA पैकेज)
- TanStack Start (SSR) + सर्वर फ़ंक्शन; डेटाबेस/लॉगिन: Bulbuly Cloud (Postgres)
- जॉब क्यू: DB तालिका + शेड्यूल्ड हैंडलर (BullMQ की जगह)
- सैंडबॉक्स: बाहरी आइसोलेटेड रनर (Judge0 / अपना Firecracker सर्वर) — मुख्य सर्वर पर कभी नहीं
- AI: Bulbuly AI Gateway; पेमेंट: Razorpay

## डेटाबेस (बन चुका)
| तालिका | काम |
|---|---|
| profiles | यूज़र नाम, भाषा, फ़ोन |
| user_roles | admin/moderator/user (अलग तालिका, `has_role()`) |
| plans | Free/Pro/Premium लिमिट्स — एडमिन बदल सकता है |
| subscriptions | यूज़र का प्लान + एक्सपायरी; `effective_tier()` एक्सपायरी पर free लौटाता है |
| bots | बॉट मेटा, स्टेटस (live/paused), webhook secret (क्लाइंट को नहीं दिखता) |
| bot_secrets | एन्क्रिप्टेड टोकन (AES-GCM) — किसी क्लाइंट रोल को एक्सेस नहीं |
| commands | नाम, aliases, help, answer, मीडिया, keyboard JSON, scenario JSON, स्कोप, admin_only, next_command, code |
| bot_users | बॉट के Telegram यूज़र्स, blocked/opted_out, awaiting (scenario स्टेट) |
| bot_variables | bot/user/chat स्कोप वेरिएबल्स |
| bot_events | लॉग्स + एनालिटिक्स |
| usage_counters | मासिक कोटा (AI, code runs, broadcasts) |
| audit_logs | एडमिन ऑडिट |

सर्वर-साइड गेटिंग: `bots_limit` ट्रिगर (बॉट संख्या), `commands_code_gate` ट्रिगर (कस्टम कोड) — API से भी बायपास नहीं।

## API डिज़ाइन (आने वाले चरण)
- आंतरिक: `createServerFn` — `connectBot`, `toggleBot`, `saveCommand`, `reorderCommands`, `importCsv`, `exportBot`, `startBroadcast` …
- `POST /api/public/telegram/:botId` — Telegram webhook (secret header से वेरिफ़ाई)
- `POST /api/public/razorpay/webhook` — HMAC सिग्नेचर वेरिफ़िकेशन, idempotent
- `/api/v1/*` — पब्लिक REST (API key), OpenAPI डॉक्स `/docs/api`

## फ़ोल्डर
```text
src/routes/            पेज + API रूट
src/routes/_authenticated/  लॉगिन के बाद वाले पेज (bottom nav)
src/lib/*.functions.ts सर्वर फ़ंक्शन
src/lib/*.server.ts    सर्वर-ओनली हेल्पर (एन्क्रिप्शन, Telegram)
docs/                  आर्किटेक्चर, सेटअप
```
