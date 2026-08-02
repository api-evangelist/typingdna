---
generated: '2026-07-21'
method: generated
name: Add Verify 2FA typing verification to a login
description: Replace SMS/email OTP with typing verification using the Verify 2FA backend client, then validate the resulting OTP.
api: openapi/typingdna-verify-2fa-openapi.json
operations: ['POST /otp/validate', 'POST /otp/send', 'POST /user/reset-profile']
source: >-
  Grounded in the published Verify 2FA OpenAPI 3.0.3 (no operationIds in the
  spec; operations referenced by method + path, verified verbatim) and
  https://verify.typingdna.com/docs.
---

# Add Verify 2FA typing verification to a login

Offer typing verification as the second factor; SMS/email OTP is only the Root-of-Trust fallback.

## Auth
- Create an Application in the Verify Dashboard to get `clientId`, `applicationId` and `secret` (Sandbox and Production credentials differ — see `sandbox/typingdna-sandbox.yml`).
- `POST /user/reset-profile` uses its own per-application `Authorization` header token from the dashboard.

## Steps
1. **Backend: prepare data attributes** — with the `typingdna-verify-client` npm package, call `getDataAttributes({email, phoneNumber, language: 'en', flow: 'standard'})` (or `disableRoT: true` with a custom `id`).
2. **Frontend: render the flow** — load `https://cdn.typingdna.com/verify/typingdna-verify.js` and render the Verify button or iframe with the `data-typingdna-*` attributes; the user types the generated 4-word text and your `callback-fn` receives `{success, otp}`.
3. **Validate the OTP** — `POST /otp/validate` with `clientId`, `applicationId`, `payload` and the `code`. `{"success": 1, "code": 19, "message": "OTP given matches"}` means the user is verified. This call is mandatory even with RoT disabled.
4. **Fallback (optional)** — `POST /otp/send` manually sends the OTP over SMS/email when the user cannot type (e.g. mobile device; RoT integrations only).
5. **Reset (optional)** — `POST /user/reset-profile` with `{user_id}` deletes the user's typing patterns (returns the deleted-pattern count).

## Errors
- Validation results come back as `code` values on HTTP 200: `10` invalid/expired OTP, `11` too many failed attempts, `12` max OTP attempts. Request errors use codes 101-135 (105 invalid client id, 108 request expired, 135 max active users). Full registry: `errors/typingdna-problem-types.yml`.

## Notes
- Sandbox environment includes the first 200 OTPs; after that connect Twilio (SMS) / Sendgrid (email) gateways in the dashboard.
- For IAM stacks (Okta, Ping, Entra) prefer the OIDC integration (issuer `https://verify.typingdna.com`, discovery in `well-known/typingdna-verify-openid-configuration.json`) — no coding required.
