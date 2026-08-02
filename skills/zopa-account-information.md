---
name: Retrieve Zopa account information (AIS)
description: Set up an account-access consent, get it authorised by the PSU, then read accounts, balances and transactions from Zopa's Open Banking Account & Transaction API.
api: openapi/zopa-account-info-openapi-original.yml
operations:
  - CreateAccountAccessConsents
  - GetAccountAccessConsentsConsentId
  - GetAccounts
  - GetAccountsAccountIdBalances
  - GetAccountsAccountIdTransactions
---

# Retrieve Zopa account information (AIS)

Zopa is an ASPSP on the UK Open Banking Read/Write standard v4.0.0. All calls are
over mutual TLS with FAPI headers (`x-fapi-interaction-id`, `x-fapi-auth-date`)
and require an OAuth2 token with the `accounts` scope. Base path:
`/open-banking/v4.0/aisp`.

## Steps

1. **Get a client token.** Use the `TPPOAuth2Security` client-credentials flow to
   obtain a token with scope `accounts`.
2. **Create the consent.** Call `CreateAccountAccessConsents` with the permissions
   the PSU is granting (e.g. `ReadAccountsDetail`, `ReadBalances`,
   `ReadTransactionsDetail`). Store the returned `ConsentId`.
3. **Redirect the PSU to authorise.** Use the `PSUOAuth2Security` authorization-code
   flow so the customer authenticates and authorises the consent; exchange the code
   for a PSU access token.
4. **Confirm consent status** with `GetAccountAccessConsentsConsentId` — proceed only
   when `Status` is `Authorised`.
5. **List accounts** with `GetAccounts`, then read `GetAccountsAccountIdBalances` and
   `GetAccountsAccountIdTransactions` per `AccountId`.

## Rules

- Always send `x-fapi-interaction-id`; echo it in logs for support/audit.
- Page transactions via the OBIE `Links.Next` / `Meta.TotalPages` fields — see
  `conventions/zopa-conventions.yml`.
- Handle errors from the `OBErrorResponse1` envelope — see `errors/zopa-problem-types.yml`
  (403 = missing `accounts` scope or revoked consent).
