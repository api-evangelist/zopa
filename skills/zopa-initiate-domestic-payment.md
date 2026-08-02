---
name: Initiate a Zopa domestic payment (PIS)
description: Create a domestic-payment consent, get the PSU to authorise it, confirm funds, then initiate the payment idempotently on Zopa's Open Banking Payment Initiation API.
api: openapi/zopa-payment-initiation-openapi-original.yml
operations:
  - CreateDomesticPaymentConsents
  - GetDomesticPaymentConsentsConsentId
  - GetDomesticPaymentConsentsConsentIdFundsConfirmation
  - CreateDomesticPayments
  - GetDomesticPaymentsDomesticPaymentId
---

# Initiate a Zopa domestic payment (PIS)

Payment initiation runs over the `payments` OAuth2 scope on base path
`/open-banking/v4.0/pisp`, with mTLS, FAPI headers and JWS request signing
(`x-jws-signature`). Writes are idempotent via `x-idempotency-key`.

## Steps

1. **Create the payment consent.** Call `CreateDomesticPaymentConsents` with the
   `Initiation` block (creditor account, `InstructedAmount`, reference). Send a fresh
   `x-idempotency-key`. Store the `ConsentId`.
2. **PSU authorises.** Redirect the customer through the `PSUOAuth2Security`
   authorization-code flow to authenticate and authorise the consent.
3. **Check consent status** with `GetDomesticPaymentConsentsConsentId` — proceed only
   when `Status` is `Authorised`.
4. **Confirm funds** (optional) with `GetDomesticPaymentConsentsConsentIdFundsConfirmation`.
5. **Initiate the payment** with `CreateDomesticPayments`, referencing the authorised
   `ConsentId` and re-sending the **same** `x-idempotency-key` on any retry.
6. **Track status** with `GetDomesticPaymentsDomesticPaymentId`.

## Rules

- Reuse the identical `x-idempotency-key` + body to retry safely; a mismatch returns
  `409 Conflict`. See `conventions/zopa-conventions.yml`.
- Sign the request payload with `x-jws-signature`.
- The `Initiation` block sent at consent creation must match the one at payment
  creation, or the ASPSP rejects with a `400` OBError.
