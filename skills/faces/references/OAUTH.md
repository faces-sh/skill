# Connect ChatGPT (Connect and Pro plans)

On the **Connect** and **Pro** plans, a user can link a ChatGPT account. A **Codex-eligible**
model (`codex_eligible: true` in `GET /v1/models`) then runs on the linked ChatGPT plan at
`/v1/responses`: the model's part of the request costs no credits and carries no 5% markup.

**The steering is billed as usual**, from the plan's allowance and then from credits. Linking
ChatGPT changes who pays for the downstream model, not for the face.

Which models are Codex-eligible changes, so read `codex_eligible` from the catalog
(`faces models:list --chatgpt`, or `GET /v1/models`) rather than keeping a list.

**The human approves once in their browser.** After that the link is stored on the account and
renews itself. Linking uses a device code, so it works over SSH, in containers and headless: no
browser extension and no local server.

## How the flow works

`faces auth:connect openai` prints a short code and a URL, then polls until the human approves:

```
To connect ChatGPT:
  1. Open this URL in any browser (phone, laptop, anywhere):
       https://auth.openai.com/codex/device
  2. Enter this code:  QUHO-OSQNW
  3. Sign in to ChatGPT and click Continue.
```

The code expires after 15 minutes. When approved, the command prints the linked account and its
ChatGPT plan. Ctrl-C cancels.

## First-time setting

ChatGPT requires device codes to be enabled once per account, on ChatGPT's side. Tell the human
up front:

> If this is your first time, enable device-code authorization in ChatGPT first:
> 1. Open https://chatgpt.com/#settings/Security (the panel can take a few seconds to appear)
> 2. Turn on "Enable device code authorization for Codex"
> 3. Then enter the code

On Team, Business and Enterprise workspaces, a workspace admin must turn it on.

## Recommended agent flow

```bash
# 1. Already linked? If openai is listed, skip the rest.
faces auth:connections --json

# 2. If not, ask the human to run this in THEIR terminal: it prints a code and a URL and
#    blocks while it polls, so do not run it as a silent background call.
faces auth:connect openai

# 3. Confirm.
faces auth:connections --json   # openai, with their email and ChatGPT plan

# 4. Unlink, when asked.
faces auth:disconnect openai
```

**Asking the human:** if `auth:connections` returns `[]`, say: *"Run `faces auth:connect openai`
in your terminal. It shows a short code and a URL: open the URL in any browser (your phone
works), enter the code, and approve in ChatGPT. First time? Turn on 'Enable device code
authorization for Codex' at chatgpt.com, Settings, Security first."*

`auth:connections` reports the linked account, for example
`openai  connected as you@example.com (Plus)`. A ChatGPT plan of `free`, or none, cannot be used
this way: tell the human.

Once linked, nothing else is needed: interviews and style builds run on the ChatGPT plan where
the model allows it. Choices and the steering of chat and decisions are not affected: they use the plan's allowance,
then credits.

## If `auth:connect` errors

- **Connect plan required** (`403 connect_plan_required`): the account is on Free. Subscribe
  with `faces billing:subscribe --plan connect` (or `--plan pro`).
- **Device-code authorization is not enabled** (`422 device_auth_not_enabled`): the first-time
  setting above. Have the human turn it on, then run the command again.
- **Your Faces session is missing or expired**: `faces auth:login`, then retry.
- **The code expired**: more than 15 minutes passed. Run `faces auth:connect openai` again.

## When ChatGPT is unavailable

On Connect and Pro the `api_fallback` preference is `false` by default: if ChatGPT is
unavailable or refuses the request, it fails rather than being paid from credits.

```bash
faces account:preferences api_fallback true     # fall back to credits
```

`--oauth-only` on `compile:thread:create` and `compile:thread:message` refuses to fall back for
that interview, whatever the preference says.

On Free there is no ChatGPT route: `api_fallback` is always `true`, and setting it to `false`,
or passing `--oauth-only`, is a `422 oauth_only_unavailable`. Nothing is charged.

When ChatGPT refuses and fallback is off, the error is under `detail`:

| `error` | When | What to do |
|---|---|---|
| `oauth_rejected` | ChatGPT refused, or no account is linked | `fallback_available` says whether the balance could pay for a retry with `api_fallback` on |
| `model_not_supported` | The model is not available on this ChatGPT plan | Pick another model |
| `oauth_workspace_residency` | The workspace's data-residency setting does not allow it | No client-side fix |
| `oauth_rate_limited` | The ChatGPT plan's usage limit is reached | Try later; `Retry-After` says when, if ChatGPT said |
| `oauth_incomplete` (`503`) | ChatGPT stopped its reply before finishing, twice | Retry. Nothing was charged |

With `api_fallback` on, a reply that ChatGPT stops short twice is answered from credits instead.
The same rule covers interviews. Style builds do not follow `api_fallback`: off a linked ChatGPT
account they are charged to credits, which the CLI asks for with `--allow-paid` (see
[REFERENCE.md → Style](REFERENCE.md#style-how-a-face-writes-style)).

## Connect and Pro onboarding

A new Connect or Pro user with no ChatGPT linked and `api_fallback` off gets a `422` for an
interview on the ChatGPT route rather than a charge, until they either:

1. link ChatGPT: `faces auth:connect openai`, or
2. allow paid fallback: `faces account:preferences api_fallback true`.

Link first, then run interviews.
