---
title: Credential Lifecycle and Troubleshooting
description: Lifetimes, reuse rules and renewal paths for every credential in the App Shop installation flow, and how to diagnose the errors they produce.
---

## Overview

The App Shop installation flow issues three credentials, and each one has different rules about how long it lasts, whether it can be used more than once, and what happens when it stops working. Most installation failures come from one of those rules rather than from a mistake in the flow itself.

This page is the reference for those rules. For the installation flow itself, see [Installation (OAuth)](./installation).

## Credential lifecycle at a glance

| Credential | Lifetime | Reuse | How to renew | When it stops working |
| --- | --- | --- | --- | --- |
| **Authorization code** | 300 seconds | Single use only | Return the customer to the Redirect step to issue a new code | Exchange fails; a new code must be issued |
| **Access token** | Varies. Read `expires_in` from the token response | Reusable until expiry, on any endpoint your app is scoped for | Exchange your refresh token | Requests fail; refresh to continue |
| **Refresh token** | Until first use, plus a one hour grace period | Single use, with retries permitted inside the grace period | Replaced automatically each time you exchange one | Customer must reauthorize the app |

## Authorization code

The authorization code arrives as the `code` query parameter when Rippling redirects the customer to your configured redirect URL. It is **valid for 300 seconds and can be used once.**

Two consequences worth designing around:

- **Exchange it immediately.** Do not queue it, hold it for a background job, or wait for the customer to finish anything in your application. Five minutes is not much once a customer is clicking through screens.
- **Extract it server-side.** Do not read the code out of the browser URL. Capture and handle it in server-side code outside Rippling.

If the window closes before you exchange the code, nothing is lost permanently. Return the customer to the installation in Rippling and click **Redirect** again on the Redirect installation flow screen. That issues a fresh code and the customer picks up where they left off.

## Access token

Access tokens do expire, but **not on a fixed schedule you can hard-code.** The token response includes an `expires_in` value in seconds, and that value is the authority:

```json
{
  "access_token": "REDACTED",
  "token_type": "Bearer",
  "expires_in": 129600,
  "refresh_token": "REDACTED",
  "scope": "employee:workEmail employee:name"
}
```

Read `expires_in` on every exchange and store the resulting expiry timestamp alongside the token. Rippling does not notify you when a token is about to expire, so tracking it is your integration's responsibility.

Within its lifetime an access token is fully reusable. You can call it as often as you need, against any endpoint your app is scoped for.

**One access token grants access to one Rippling company.** If twelve companies have installed your app, you are holding and refreshing twelve separate token pairs. Plan storage and refresh scheduling per company, not per app.

## Refresh tokens rotate

This is the rule most likely to break an integration in week two rather than on day one.

**Each time you exchange a refresh token, you receive a new access token and a new refresh token, and the refresh token you just used is revoked.** The credential is not a long-lived key you keep on file. It is replaced on every use.

An integration that stores the refresh token from the original grant and reuses it will succeed the first time and fail every time after that.

To handle this correctly:

1. Exchange the current refresh token for a new token pair.
2. Persist **both** new values, overwriting the previous refresh token.
3. Use the new refresh token for the next cycle.

### The grace period

A used refresh token remains valid for **one hour after its first use.** This exists so that an integration which loses the response to an exchange, through a timeout, a crash or a failed write, can retry without stranding the customer.

Treat it as a safety net, not as permission to reuse tokens. Outside that hour, the old refresh token is gone.

## Reauthorization

Reauthorization sends a customer back through the authorize step to issue a fresh authorization code, and from it a new token pair. Direct the admin to:

```
https://app.rippling.com/apps/PLATFORM/{APPNAME}/authorize
```

They can also uninstall and reinstall the app from **the app in Rippling > Settings > Uninstall**, which achieves the same thing with more disruption.

Two situations require it:

**Both tokens have been revoked.** There is no way to recover a token pair from your side. The customer has to reauthorize.

**You have added new scopes to your app.** Existing installations do **not** gain access to new scopes automatically. Every currently-installed customer must reauthorize and consent to the new scopes individually. If you have three customers installed, that is three separate reauthorizations, and you are responsible for prompting them.

Plan scope changes accordingly. Adding a scope is a customer-facing migration, not a configuration change.

## Detecting uninstalls

When a customer uninstalls your app, Rippling sends a `company.deleted` webhook event:

```
event_name=company.deleted&id=583fd8803bf207cab80269b9&company_id=583fd8803bf207cab80269b9&company_primary_email=huroncompany%2Bfake%40rippling.com
```

Independently of the webhook, any subsequent API call for that company returns `403`:

```json
{ "detail": "The token has been revoked" }
```

**Handle both signals.** The webhook tells you promptly and lets you clean up in an orderly way. The `403` is the backstop for a webhook that was missed, delayed, or never delivered because your endpoint was down. An integration that only watches for one of them will eventually hold stale credentials and keep retrying against a company that is gone.

## Troubleshooting

| Symptom | Likely cause | Resolution |
| --- | --- | --- |
| Token exchange fails immediately after a successful redirect | The authorization code has passed its 300 second window, or has already been used once | Return the customer to the Redirect step to issue a new code, and move the exchange earlier in your flow |
| Token exchange fails with a redirect URI error | The `redirect_uri` sent in the exchange does not match the one configured in your app listing | Send the exact configured value. Rippling will not accept a URI that has not been registered |
| Refresh succeeds once, then fails on every later attempt | Your integration stored the original refresh token instead of the rotated one | Persist the new refresh token returned by every exchange |
| Refresh fails with no prior successful call | The refresh token was already consumed and the one hour grace period has passed | The customer must reauthorize |
| All endpoints return `403 The token has been revoked` | The customer uninstalled the app, or the tokens were revoked | Check for a `company.deleted` event. If the customer intends to continue, have them reauthorize |
| New endpoints return permission errors after a release | The app's scopes changed, but existing installs are still consented to the old set | Prompt each installed customer to reauthorize |
| Access token expires sooner than expected | Your integration assumed a fixed lifetime rather than reading `expires_in` | Store the expiry from each token response and refresh against it |

## Error reference

| Response | Meaning | Action |
| --- | --- | --- |
| `403` with `{ "detail": "The token has been revoked" }` | The access token is no longer valid for this company, usually because the app was uninstalled or access was revoked | Stop retrying, mark the install inactive, and reauthorize only if the customer asks to reconnect |

## Need Support?

If you're a partner seeking answers or encountering difficulties while integrating with Rippling, please contact us at [partner.support@rippling.com](mailto:partner.support@rippling.com). Ensure our responses reach you by preventing emails from this address from being filtered into your spam folder.
