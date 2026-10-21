---
name: mentiondock
description: Query raw Instagram and TikTok creator data with MentionDock, find candidates by handle or content topic, and manage connections, credit packs, subscriptions, payment and query recovery, and CLI updates. The user's own AI organizes, compares, and analyzes the returned creator data.
compatibility: Node.js 22.20+, installed mentiondock CLI, local CLI execution, network access to MentionDock, and a browser for Google sign-in and Stripe payment.
---

# MentionDock

Use the available `mentiondock <command>` CLI to retrieve raw data, then organize the results around the user's business goal. Commands work from any directory. Credentials and local query, purchase, and subscription records are stored under the user's `~/.mentiondock` directory.

## Connect with Google

Start with `mentiondock account`. Existing credentials are reused for queries. To connect a new device or renew an expired login, explain that Google sign-in connects the user's independent MentionDock account, then continue when the user authorizes the connection. An explicit request to sign in is sufficient.

Run `mentiondock login`. New logins default to the production service at https://exapi.pietrastudio.com. It opens the Google authorization page, and returns `authorization_required`. Give the user `authorizationUrl` if a browser cannot be opened. Let the user authorize directly in the browser; their password and verification codes stay there.

Run the same `mentiondock login` command at the returned `intervalSeconds` spacing to check the saved login. `pending` means browser authorization is still incomplete; `connected` confirms the local token was saved. Continue the user's task once connected, without requiring another confirmation in chat. The CLI exchanges its private local verifier for the token and outputs only safe account information. An expired or consumed login requires running `login` again to start a new authorization.

On another computer, sign in with the same Google account to access the same MentionDock account and credits. Query and purchase recovery records remain on the computer that created them. To change accounts, run `mentiondock login` and choose the intended Google account in the browser. When the user specifies a test service, include its `--service` on login, queries, payments, and recovery commands. `MENTIONDOCK_URL` can also select a service; `--service` takes precedence. The CLI keeps one current login. To switch services, run `logout`, then sign in to the selected service; saved recovery records remain bound to their original account and service.

## Query data

Run `mentiondock help` for current commands and arguments. Common queries:

```sh
mentiondock query --json '{"platform":"instagram","handle":"nike","limit":1}'
mentiondock query --json '{"hashtag":"skincare","limit":10,"minFollowers":10000,"maxFollowers":100000}'
```

Queries use existing credits. Choose exactly one of `handle`, `hashtag`, or `keyword`. Set `platform` only when the user wants Instagram or TikTok; omitting it searches stored data from both platforms. Topic queries accept `limit` from 1 to 50 and default to 10. Handle queries default to and require 1. For larger requests, explain the 50-creator limit and direct the user to contact us at https://mentiondock.com.

Run the query once and wait for it to finish. The CLI checks the saved request internally, writes brief English progress to stderr, and returns final JSON on stdout. When your terminal tool yields a running session, continue waiting on that session. Use `result.items` as the creator data and `costCredits` as the actual charge.

Topic searches filter stored follower counts and leave existing profiles unchanged. If the results are insufficient, the backend collects once on the selected platform, or TikTok when platform is omitted. Instagram collection supports handles and hashtags; Instagram keyword searches use stored data only. A handle lookup refreshes that profile when needed and uses its stored platform, or TikTok if no stored account exists and no platform was selected. Results may still contain fewer creators than requested.

Each creator includes up to five `recentPosts` already stored by the service, with basic post fields. These are the latest stored posts, and may differ from the platform's latest posts. Use `profileObservedAt`, `followerCountObservedAt`, and missing values when assessing the data. Make claims about location, audience demographics, contact details, or willingness to collaborate only when the data supports them. Keep each returned platform account distinct, distinguish raw facts from your analysis, and produce the list or report the user needs.

## Credits and plans

`mentiondock balance` returns remaining, reserved, and available credits, with separate subscription and one-time balances. `topupBalances` lists each unexpired pack's remaining credits and `expiresAt`. If credits are insufficient, explain that the query has stopped. Run `mentiondock topup` to read the current products and usage `prices`. Use the `credits` in the `dataType=creator` entry as the cost per delivered creator profile, with the same rate across Instagram and TikTok. Each profile represents one social account; group accounts as one person only when the data proves that relationship. `amountBp` uses 10,000 Bp per US dollar.

Choose the product type from `billingInterval`: `one_time` is a credit pack; `month` or `year` is an automatically renewing subscription. For subscriptions, `amountBp` is the amount charged each month or year and catalog `credits` is the fixed monthly allowance. Subscription months start from the subscription date, unused allowance expires each month, and annual payment still resets the allowance monthly. Queries use the current subscription allowance first, then one-time credits. Use `subscriptionPeriodEnd` for the next allowance reset; `currentPeriodEnd` describes the subscription's paid billing period.

Match the user's authorization to the payment type. State the pack price, quantity, and six-month validity before a one-time purchase; an explicit instruction to buy that pack or within a stated one-time budget is sufficient. Packs expire six calendar months after payment, with month-end dates adjusted to the target month's last day. Queries consume packs that expire sooner first, after the subscription allowance. Before subscribing, state the amount per billing interval, monthly allowance, monthly expiry, and automatic renewal, and obtain authorization for those recurring charges. A query uses existing credits. A one-time purchase budget authorizes the pack purchase only.

### One-time credit packs

After the purchase is authorized, run `mentiondock buy <productId>`. The CLI saves a stable `purchaseId`, creates or resumes the order, and opens Stripe Checkout. Give the user `checkoutUrl` if `browserOpened` is false. The user confirms the amount on Stripe's page using an offered payment method; Link is optional on that page.

Use `mentiondock resume-purchase <purchaseId>` at spaced intervals to check the same purchase. It reopens the validated Checkout page while `action=checkout`; `waiting` means payment confirmation or page creation is pending. Only `action=complete` confirms that the server credited the purchase. Report its `periodEnd` as the expiry date, then read `balance` and resume the original data query. Browser confirmation does not require a separate reply in chat.

After a timeout or process exit, use `mentiondock purchases` to find the saved ID and resume it. Listed statuses are local snapshots; resuming checks the server. `buy` reuses an unfinished purchase for the same pack. If the server reports `expired`, explain this before starting a new purchase with renewed authorization. Uncertain payment results require checking the original order.

### Automatically renewing subscriptions

After the user authorizes the recurring plan, run `mentiondock subscribe <productId>`. The CLI saves a stable `id`, creates or resumes that application, and opens a validated Stripe Checkout page. Give the user `checkoutUrl` if `browserOpened` is false. The user confirms the first payment and recurring terms on Stripe's page, using Link when available or another offered payment method. Handle payment credentials in that page.

Read `mentiondock subscription` for the current or latest subscription; it returns `null` when none exists and never opens a payment page. Use the saved `id` with `mentiondock resume-subscription <id>` to resume the same application; this reopens its valid Checkout page when needed. After the user completes the page, or while waiting for payment confirmation, check `subscription` at spaced intervals. A separate reply in chat is unnecessary when the backend already confirms activation.

| Action | Next step |
| --- | --- |
| `checkout` | Let the user confirm on the Stripe page. Preserve this application ID. |
| `waiting` | Check the same subscription later; payment confirmation is still pending. |
| `active` | The backend confirms activation. Check `balance` and resume the user's query. Stripe handles subsequent renewal payments. |
| `ended` | This application or subscription has ended. A new subscription requires renewed authorization and `subscribe`, which creates a new ID. |

Opening or closing the browser page does not establish payment or cancellation. Unknown network results require checking the original application. An account can have one unfinished or active subscription at a time. On another computer, `subscription` retrieves the account's current state and records its ID locally for recovery.

When the user explicitly asks to stop renewal, read the current subscription and run `mentiondock cancel-subscription <id>`. Confirm `cancelAtPeriodEnd: true` and report `currentPeriodEnd`: access to the paid period continues, and renewal stops at its end. This command does not request an immediate refund or a plan change. Closing Checkout, logging out, or removing the Skill leaves automatic renewal unchanged.

Each data query reserves credits for its result limit. The backend settles the charge before returning creator data, charging only for creators actually delivered and releasing unused reserved credits. Recent stored posts are included in the creator price.

## Recover requests and results

The CLI saves the request ID before sending it and reuses unfinished queries with the same criteria. If the command times out or is interrupted, use the saved ID with `mentiondock resume <id>`; it waits for the same query. Use `mentiondock requests` to find the ID when needed.

Completed requests can also be retrieved by their original ID. The creator set stays fixed, while the response contains their latest stored profiles and posts. Retrieval does not trigger additional collection or another credit charge. `returnedRows` and `costCredits` describe the original delivery and charge. If a creator is no longer available, report the IDs in `missingCreatorIds`; the current result count may be lower.

A query with status `FAILED` has ended. Start a new query when another attempt is needed; a new query with the same criteria may incur another charge.

`mentiondock logout` deletes local MentionDock credentials and the pending Google login, and keeps query, purchase, and subscription records. Reconnect to the original MentionDock account to resume those requests. Existing MentionDock tokens retain their original expiry; this command does not revoke access on remote devices.

## Keep the CLI and Skill current

Successful commands may also write an `updateAvailable` notice to stderr, with `currentVersion`, `latestVersion`, and `command`. Continue processing the command's normal result, and tell the user which update is available. When the user asks to check for updates, run `mentiondock update --check`; this only checks versions.

Run `mentiondock update` after the user authorizes the update. It updates the CLI and synchronizes this Skill at its previously registered locations, preserving credentials and query, purchase, and subscription records. After a successful update, read the updated `SKILL.md` at the path used by this session, then continue the user's task with its current instructions. If the agent cannot reload the Skill in this session, ask the user to start a new session to load the updated instructions.
