# Billing & Credits

All billing commands are `faces billing:COMMAND`. Add `--json` (or `--toon`) for structured
output.

## Plans

| plan | price | what it includes |
|---|---|---|
| `free` | $0 | Everything is paid as it goes, from credits |
| `connect` | $17/month | The allowances below; with ChatGPT linked, interviews and style builds run on the ChatGPT plan |
| `pro` | $49/month | Larger allowances, and the same ChatGPT link |

A subscription is an **allowance**, not a fee per request. It covers compiling, choices and the
**steering** of chat and decisions (the face's part of `@alice:maestro-1/<model>`), up to the
limits below. The **downstream model** (the chat model or decision model after the `/`) is never
part of it: on every plan it is paid at the model's listed price plus 5%, so `faces decide` and
`faces chat:*` always need credit (a Codex-eligible chat model on a linked ChatGPT plan aside;
see [OAUTH.md](OAUTH.md)).

| allowance | Connect | Pro |
|---|---|---|
| compile, per month | 500k source tokens | 1.5M source tokens |
| choices and steering, per month | 25M tokens | 75M tokens |
| choices and steering, per week | 6.25M | 18.75M |
| choices and steering, per 5 hours (rolling) | 1.6M | 4.8M |

- **Compile** counts a source's tokens **each time** it is compiled; a recompile counts again.
- **Choices** counts the call's `usage.input_tokens`, the input tokens it processed; on
  `maestro-1-flash` with a question, 10 times that.
- **Steering** (chat and decisions) counts the input tokens it processed plus 1,000 a call
  (the $0.00042 fee, counted as the tokens it buys); on `maestro-1-flash`, 10 times that.
- The month is the subscription's 30-day window, weeks are 7-day blocks of it, and the 5-hour
  window rolls.

**Past an allowance**, the `api_fallback` preference decides:

- **on:** the work carries on, paid from credits at the pay-as-you-go rates. With no credit it
  is a `402`.
- **off** (the default): refused with a `429`, and nothing is charged. The message names the
  allowance and when it resets. Show it as it is, and offer
  `faces account:preferences api_fallback true` only if the user agrees to pay from credits.

**Pay-as-you-go rates** (Free, and past an allowance with fallback on):

| | rate |
|---|---|
| Choices, `maestro-1` | $0.42 per million input tokens, $1.00 per million output tokens, as `usage` reports them |
| Choices, `maestro-1-flash` with a question | $4.20 and $10.00 per million; without a question, as `maestro-1` |
| Steering chat and decisions, `maestro-1` | $0.42 per million input tokens plus $0.00042 a call; no output charge |
| Steering, `maestro-1-flash` | $4.20 per million input tokens plus $0.0042 a call |
| Compile | $16 per million source tokens |
| Rush compile (`--rush`) | $21 per million source tokens |
| A downstream model (chat or decision) | the model's listed price plus 5% |

The live figures are public at `GET /v1/rates` (no key needed), and per model in the model
catalog (`faces models:list`, or `GET /v1/models`): the maestros are `maestro-1` and
`maestro-1-flash` (`faces models:list --kind upstream`, with their steering prices) and the
compile model is `faces-compile-latest`. Quote them from there, not from memory.

## Check the plan and its allowances

```bash
faces billing:subscription
faces billing:subscription --json
```

Shows the plan, the renewal date and, on a paid plan, a meter of each allowance:

```
Connect allowances (tokens):
  choices and steering, rolling 5 hours  3018 of 1.6M (0%)  rolls
  choices and steering, this week        3018 of 6.25M (0%)  resets 2026-10-26 09:44 UTC
  choices and steering, this month       32.5k of 25M (0%)  resets 2026-10-28 09:44 UTC
  compile, this month                    4072 of 500k (0%)  resets 2026-10-28 09:44 UTC
```

In `--json`, `allowances` holds `cause_5h`, `cause_week`, `cause_month` and `compile_month`,
each `{limit, used, remaining, resets_at}`; the `cause_*` ones are the choices and steering
allowance (the key names are older than the product's). It is `null` on Free. `cause_5h` rolls, so its
`resets_at` is `null`. `rates` carries the pay-as-you-go rates.

## Subscribe

```bash
faces billing:subscribe --plan connect      # alias of billing:subscription:activate
faces billing:subscribe --plan pro
```

From Free, prints a Stripe Checkout URL. Only the human can complete it, in a browser; the page
takes a **promotion code**, which can make a plan free for a while or add credits. If the
subscription is set to cancel and is still in its period, this keeps it instead.

Checkout is for a **first** subscription only: already subscribed, it is refused (`409`).
Switch instead.

## Switch between Connect and Pro

```bash
faces billing:subscription:plan pro         # asks to confirm; --yes skips that
faces billing:subscription:plan connect
```

Switches in place, prorated on the default card, and the new allowances apply at once. Confirm
with the user first: it charges. A `409` means there is no subscription to switch (subscribe
instead). While the plan is set to cancel, the switch is refused with
`409 cancellation_pending` before anything is charged: keep the plan first with
`faces billing:subscription:activate`.

## Cancel

```bash
faces billing:subscription:cancel
```

Keeps the plan until the current period ends, then the account returns to Free. To keep it,
run `faces billing:subscription:activate` before then.

## Credits

```bash
faces billing:balance --json                 # balance_usd, has_payment_method (a default card exists)
faces billing:topup --amount 5               # charges the default card; minimum $1, whole cents
faces billing:topup --amount 5 --card pm_…   # charges another saved card
```

The same amount again within two minutes is a `409 duplicate_topup`, and nothing is charged: it
usually means the first went through. Check `faces billing:balance` before trying again, and
repeat it only if the user confirms, with `--allow-duplicate` (`allow_duplicate: true` on the
API). `--yes` does not imply it.

Credits apply at once. A `402` anywhere means there is no credit for that request, including
`faces decide` and `faces chat:*` on Connect and Pro, whose downstream model is always paid. Top up and retry.

## Cards

An account can save several cards. One is the **default**: the subscription renews on it, and
a top-up charges it unless `--card` names another. The card used at checkout becomes the
default.

```bash
faces billing:cards                          # newest first: brand, last 4, expiry, id, (default)
faces billing:card-setup                     # alias billing:card:add: a Stripe link to save a card
faces billing:card-setup --no-default        # save it without making it the default
faces billing:card:default pm_…              # the next renewal charges this one
faces billing:card:remove pm_…  [--yes]
```

- Saving a card charges nothing. The new card becomes the default unless `--no-default`; the
  first card always does. It is listed a few seconds after the user finishes: if
  `billing:cards` does not show it yet, wait and look again.
- Removing the default makes the newest other card the default. The **last** card cannot be
  removed while a paid subscription renews on it (`409`): add another first, or cancel.
- `404` means the card is not on this account: list them with `billing:cards`.

## Compile usage

```bash
faces compile:stats --json
```

```
Plan: connect
Compiled this month: 4,072 of 250,000 source tokens (2%), resets 2026-10-28
Pay-as-you-go compile rate: $25 per million source tokens (past the allowance, with api_fallback on)
```

A last line gives the rush rate. `--json` is `{plan, compile_this_month: {limit, used,
remaining, resets_at}, pay_as_you_go: {usd_per_million_source_tokens}, rush:
{usd_per_million_source_tokens}}`; `compile_this_month` is `null` on a plan with no compile
allowance, such as Free. Per-face numbers are in `faces face:stats`.

Compile, upload and thread responses carry `tokens_used` and `tokens_limit`: the source tokens
compiled in the current subscription window, against the plan's monthly compile allowance.
`tokens_limit` is `0` on a plan with no allowance (compiles are paid per source token) and `-1`
when unlimited.

## Rush compiles

A compile may have a scheduled start: `/make` answers with `starts_at`, or `null` when it starts
at once. `--rush` starts it now at the rush rate. Quote both rates from `faces compile:stats` or
`GET /v1/rates`, not from memory; the compile model `faces-compile-latest` also lists
`rush_input_cost_per_million_tokens`.

- **Paid as you go** (Free, or past an allowance with `api_fallback` on): a rush costs the rush
  rate instead of the standard one.
- **Within a plan's compile allowance:** the allowance covers the standard rate, and the rush
  adds the difference, paid from credit. Without enough credit, `/make` refuses with a `402`,
  `error_code: "insufficient_credit_for_rush"`, and `rush_premium_usd` (the amount), before
  anything is queued. A compile without `--rush` is never refused for this.
- Where a compile starts at once anyway, `--rush` changes nothing and costs nothing extra; the
  answer says `rush: false`.
- A rush's extra charge appears in usage under `faces-compile-latest`, like the compile itself.
- **A queued compile can be rushed.** `--rush` on `compile:doc:make` or `compile:thread:make`
  for a compile already waiting for its scheduled start makes it start now, billed exactly as a
  rush asked for up front. Without enough credit it is the same `402`, and the compile stays
  queued, unchanged. Asking again on a compile that is already a rush never charges twice.
- While a compile is queued, its status carries `rush_premium_usd`: what a rush would cost now,
  from credit. It is `null` once the compile has started or is already a rush.
- **Once a compile has started it is too late:** `409`, `error_code: "compile_already_started"`.
  Nothing changes and nothing is charged.

## Style builds

On Connect or Pro with a linked ChatGPT account, a style build runs there at no charge.
Anywhere else (Free, or Connect or Pro with no ChatGPT linked) it is charged to credits at the
build model's listed price plus the markup, and appears in usage under that model. The CLI asks
for `--allow-paid` before any build that would be charged; without it such a build is refused
with `409 chatgpt_not_linked`. A build credit cannot cover is refused up front with `402
insufficient_credit_for_style_build`, carrying `estimated_usd` and `balance_usd`; nothing is
started or charged. Automatic builds follow the same rule. See
[REFERENCE.md → Style](REFERENCE.md#style-how-a-face-writes-style).

## Model prices

```bash
faces billing:llm-costs --json
faces billing:llm-costs --lab openai
```

Lists the models with their per-token prices, one row per model under the id its lab gives it.
`--lab` filters by that **lab** (openai, anthropic, xai, qwen, z-ai, moonshot, and so on).
Prices are listed before the markup; each row carries `markup_rate` (`0.05` is 5%). A model's
price is the same whichever way it is served, so read it here rather than quoting one from
memory. Transcription is priced per hour, in the same list.

## Usage

```bash
faces billing:usage --json
faces billing:usage --group-by model --from 2026-03-01 --to 2026-03-31
```

Group by `api_key`, `model`, `llm` or `date`. Compiling appears as `faces-compile-latest`. COST is what you were charged.

## What to do on

**402 insufficient_credit_for_rush:** a `--rush` compile within the allowance, without the
credit for its premium. Show `rush_premium_usd`, then top up, or compile without `--rush` to
start at the scheduled time.

**402 insufficient_credit_for_style_build:** a style build charged to credits, without the
credit for it. Show `estimated_usd` and `balance_usd`, then top up, or on Connect or Pro link
ChatGPT (`faces auth:connect openai`) to build there at no charge.

**409 compile_already_started:** `--rush` on a compile that has already started. It carries on
as it is; nothing was charged.

**402:** no credit for the request. `faces billing:balance --json` to confirm, then
`faces billing:topup --amount <USD>` (with no card saved, `faces billing:card-setup` first),
and retry.

**429 naming an allowance:** a plan's allowance is used up and `api_fallback` is off. Wait for
the reset the message names, or, if the user agrees to pay from credits,
`faces account:preferences api_fallback true`.

**429 Rate limit:** every route that runs a model or a compile counts against a per-account,
per-minute limit set by plan (`limits.requests_per_minute` in `GET /v1/rates`). Nothing is
charged for the refused request. The CLI waits the `Retry-After` once and asks again; after a
second refusal, wait the seconds the message names. Never retry in a loop: parallel agents on
one account share the limit.

**422 oauth_only_unavailable:** `--oauth-only` where the ChatGPT route cannot serve the request:
always on Free, and on Connect or Pro when the model or the link cannot. Nothing is charged.
Drop the flag to pay from credits, or, on Free, subscribe to Connect to use a linked ChatGPT
account.

**422 oauth_rejected:** ChatGPT refused and fallback is off. `faces account:preferences
api_fallback true` to allow credits (top up first if there are none). See [OAUTH.md](OAUTH.md).
