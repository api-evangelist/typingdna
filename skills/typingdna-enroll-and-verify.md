---
generated: '2026-07-21'
method: generated
name: Enroll a user and verify their typing pattern
description: Capture typing patterns with a TypingDNA recorder, enroll them, then verify the user by how they type.
api: openapi/typingdna-authentication-api-openapi-original.json
operations: ['POST /auto/{id}', 'GET /user/{id}', 'POST /save/{id}', 'POST /verify/{id}']
source: >-
  Grounded in the published OpenAPI 3.0.3 (the spec ships no operationIds, so
  operations are referenced by method + path, all verified verbatim in the
  spec) and https://api.typingdna.com/docs/index.html.
---

# Enroll a user and verify their typing pattern

Recognize a user by how they type: enroll 3+ typing patterns, then verify new ones.

## Auth
- HTTP Basic: `apiKey` as username, `apiSecret` as password, over HTTPS. Get keys with a free Starter account (https://www.typingdna.com/clients/signup). See `authentication/typingdna-authentication.yml`.
- Base URL `https://api.typingdna.com` (or `https://eu-api.typingdna.com` / `https://us-api.typingdna.com`).

## Capture
- Record patterns client-side with the `typingdna.js` recorder (`getTypingPattern({type: 2, text: ...})` — type 2 "extended" is recommended, same 30-40 char text for enroll and verify). Never call the API from the browser; submit from your backend.

## Steps
1. **Check the user** — `GET /user/{id}`. If `count` (desktop) or `mobilecount` (mobile) is above 0, the user already has enrollments; skip to verification.
2. **Enroll** — `POST /save/{id}` with the recorded `tp` (typing pattern), repeated for 3 initial patterns. Mobile and desktop patterns are tracked separately.
3. **Verify** — `POST /verify/{id}` with a freshly recorded pattern. Read `result` (0/1), `score` (0-100) and `message_code`. A score >= 90 is a high-confidence match; tune the threshold to your risk.
4. **Or let the API decide** — `POST /auto/{id}` coordinates enroll-vs-verify automatically: it enrolls until enough patterns exist, then verifies (Auto-Enroll settings in the dashboard control thresholds).

## Errors
- Handle by `message_code`, not message text: `3` = new typing position (enroll it), `9`/`10` = not enough patterns yet, `2` = duplicate pattern (replay protection), `40` = mobile vs desktop patterns cannot be matched. Full registry: `errors/typingdna-problem-types.yml`.
- No idempotency-key contract exists; retries of `POST /save/{id}` with the identical pattern are rejected as duplicates by design.

## Notes
- Quality param (1 UX / 2 balanced / 3 security) can be passed on `POST /verify/{id}`. Rate limiting signals: HTTP 429 and message codes 37/43/52. See `conventions/typingdna-conventions.yml`.
