# Full command reference

```
faces auth:login        --email  --password
faces auth:logout
faces auth:register     --email  --password  --username  [--plan free|connect|pro]
                                                       # username: 1-32 chars, a lowercase letter, then lowercase
                                                       # letters, numbers or underscores (no dashes)
faces auth:whoami
faces auth:refresh
faces auth:connect      openai   # device-code flow: prints a code and a URL, polls until approved
faces auth:disconnect   <provider>
faces auth:connections

faces face:create       --name  --alias  [--default-model MODEL]  [--description TEXT]  [--specialty TEXT]  [--tag TAG...]  [--formula EXPR | --attr KEY=VALUE... --tool NAME...]  [--profile-addendum TEXT | --profile-addendum-file PATH]
faces face:list         [--has-style]  [--tag TAG...]  [--group GROUP_ID...]  [--include tags,groups,profile,sharing,avatar,usage,owner_follow]  [--public]  [--shared]  [--system]  [--from-users USER...]  [--not-from-users USER...]
                        [--search TEXT]  [--sort created|alias|relevance|usage]  [--order asc|desc]
                                                       # --search matches alias, name and description.
                                                       # OMIT --order to keep each field's own direction: newest first for
                                                       # created, most used first for usage, best match first for relevance,
                                                       # A to Z for alias. --sort relevance needs --search.
                                                       # --public = open to everybody; --shared = shared with YOU. They are
                                                       # independent and compose. Faces you do not own print as owner:alias.
faces face:get          <alias | owner:alias>  [--include tags,groups,profile,sharing,avatar,usage]  [--full]
                                                       # prints `styles:` with the media the face has a built style for.
                                                       # A face you do not own prints an `access:` line saying so.
faces face:attributes
faces face:edit         <alias>  [--name]  [--default-model MODEL]  [--description TEXT]  [--specialty TEXT]  [--tag TAG...]  [--formula EXPR]  [--attr KEY=VALUE]...  [--tool NAME...]  [--profile-addendum TEXT | --profile-addendum-file PATH | --clear-profile-addendum]
                                                       # --formula on a face with sources of its own, or one another
                                                       # composite uses, is a 409: make a new composite with
                                                       # face:create --formula. --formula "" makes it an ordinary face again.
faces face:delete       <alias>  [--yes]
faces face:lock         <alias>              # freeze the face (read-only)
faces face:unlock       <alias>              # make it writable again
faces face:share        <alias>  (--list | --add ACCOUNT... | --with ACCOUNT... | --none)  [--yes]
faces face:unshare      <alias>  (--from ACCOUNT... | --all)  [--yes]
                                                       # --add keeps the current list; --with REPLACES it, and asks first
                                                       # (or needs --yes) when that would drop someone. Accounts are
                                                       # usernames or emails; the stored list is usernames, so revoke by
                                                       # username. Sharing lets them ask it and read its card: not
                                                       # its documents, compiling, re-sharing, or its profile addendum.
faces face:publish      <alias>  --yes        # every account can ask it
faces face:unpublish    <alias>
faces face:sources      <alias>  [--type doc|thread]
                                                       # every source on the face, documents and threads in one table:
                                                       # label, tokens, status, medium, updated. STATUS reads `finishing`
                                                       # first: a source still in hand (retries included) needs NO action.
                                                       # Exits 4 for a face that does not exist (a face with no sources
                                                       # exits 0). Never prints content. --json has a stable id per row.
faces face:stats
faces face:diff         --face ALIAS  --face ALIAS  [--face ALIAS]...
                                                       # one overall similarity score per pair, -1 to 1. Every face named
                                                       # needs something compiled.
faces face:neighbors    <alias | owner:alias>  [--k N]  [--direction nearest|furthest]
                                                       # nearest (default) = most SIMILAR; furthest = most DIFFERENT.
                                                       # A face with nothing compiled is refused. Similarity is -1 to
                                                       # 1, rounded to one decimal place when either face is not yours.

faces face:avatar:list    <alias | owner:alias>          # versions, which one is served, which are stale
faces face:avatar:get     <alias | owner:alias>  [--size sm|md|lg]  [--format png|svg]  [--width PX]  [--version N]  [--prefer organic|uploaded]  [--out PATH]
                                             # FREE. Writes the image and says which version arrived,
                                             # generated or uploaded, and whether it is stale.
faces face:avatar:make    <alias>  --allow-paid  [--seed N]
                                             # COSTS MONEY on Free. Refuses without --allow-paid and sends
                                             # nothing. Prints the amount charged.
faces face:avatar:upload  <alias>  --file PATH          # your own picture: png, jpeg, webp, gif or bmp. Free; replaces any earlier upload.
faces face:avatar:delete  <alias>  [--yes]              # removes only the UPLOAD; generated versions stay.

faces user:get           <username>           # name, follower counts, and whether you follow them
faces user:follow        <username>
faces user:unfollow      <username>
faces user:followers     <username>
faces user:following     <username>

faces face:tag:list     <alias>
faces face:tag:add      <alias>  --tag TAG  [--tag TAG...]
faces face:tag:set      <alias>  --tag TAG  [--tag TAG...]
faces face:tag:remove   <alias>  <tag>
faces face:tag:all
faces face:groups       <alias>  --group GROUP_ID  [--group GROUP_ID...]

faces choices          <@alias:maestro-1 | @owner~alias:maestro-1 | "(@a | @b):maestro-1">
                        (--state TEXT | --state-file PATH | --state-id ID)  [--question TEXT]  [--max-choices N]
                                                       # what the face would do (POST /v1/choices): its options,
                                                       # strongest first. No --question: the question it would ask.
                                                       # --state is prose, at most 3,000 tokens; longer goes in a state
                                                       # (--state-id). With both, the prose is an addendum for that
                                                       # call only, never saved: persist facts with state:add --premise.
                                                       # :maestro-1-flash answers sooner, at 10x.

faces chat:chat         <@alias:maestro-1/MODEL>  (-m MSG... | --file PATH)  [--system TEXT]  [--max-tokens N]  [--temperature T]
                        [--no-stream]  [--responses]  [--conversation ID [--premise K=V ...]]  [--compaction ID]  [--oauth-only]
                                                       # chat with a face: it steers, MODEL writes the reply. Routed to
                                                       # chat completions, responses or messages by the catalog.
faces chat:messages     <@alias:maestro-1/MODEL>  ...  # the same, always /v1/messages
faces chat:responses    <@alias:maestro-1/MODEL>  ...  # the same, always /v1/responses
faces chat:thread       [<@alias:maestro-1/MODEL>]  [--id THREAD_ID]  -m MSG  [--conversation ID]  [--list | --show | --delete]
                                                       # a multi-turn chat kept under ~/.faces/threads

faces state:create      [<@alias | @owner~alias | "(@a | @b)">]  [--premise K=V ...]  [--no-causes-visible]
faces state:add         <state_id>  [--premise K=V ...]  [--premise-description K="..."]  [--premise-status K=STATUS]
                        [--premise-depends-on K=a,b]  [-m MSG]
                                                       # prints the status changes and the decisions resting on a value
                                                       # you changed. STATUS: stated|verified|assumed|unresolved|invalidated
faces state:get         <state_id>
faces state:items       <state_id>                     # premises with their status, decisions, messages
faces state:compact     <state_id>  [<@face>]
faces state:close       <state_id>  [--yes]
                                                       # conversation:* and session:* are older names for state:*

faces models:list       [--kind upstream|chat|decision|compile|image|transcription[,...] | --kind all]  [--provider LAB]  [--chatgpt]
                                                       # the model catalog, once each, with kind, endpoint and price.
                                                       # upstream: the maestros, with their steering prices.
                                                       # CODEX marks Codex-eligible models. No sign-in needed.

faces decide            <@alias:maestro-1/DECISION_MODEL>  (--state TEXT | --state-file PATH)
                      [--question TEXT [--type noul|choice|score] [--option NAME[=DESCRIPTION]... | --level DESCRIPTION...] [--id KEY] | --questions-file PATH]
                      [--format systemone|openai]
                      [--conversation ID [--depends-on a,b] [--premise K=V ...]]  [--session ID]  [--compaction ID]  [--no-retry]
                                                       # typed answers, as this face. DECISION_MODEL: jev-latest, clef,
                                                       # clef-flash or gpt-6-luna. See Decisions.

faces benchmark:start   <alias>  [--model MODEL]  [--reference-model MODEL]  [--rounds N]  [--seed N]  [--max-spend USD]  [--dry-run]  [--wait]
faces benchmark:get     <run_id>  [--wait]
faces benchmark:list    [--face ALIAS]  [--limit N]

faces compile:import       <alias>  --url YOUTUBE_URL  [--type document|thread]  [--medium MEDIUM]  [--perspective first-person|third-person]  [--face-speaker LABEL]  [--no-wait]
                                                       # --medium is required with --type document and refused with thread.
                                                       # --perspective defaults to third-person here.
faces compile:upload       <alias>  --file PATH  [--kind document|thread]  [--medium MEDIUM]  [--perspective first-person|third-person]  [--face-speaker NAME]  [--no-wait]
                                                       # --kind document is the face's own writing; an interview, transcript
                                                       # or dialogue is --kind thread with --face-speaker, so only the
                                                       # face's turns are used.
                                                       # text, PDF, Word (.docx), audio and video. --medium is required with
                                                       # --kind document and refused with thread. --perspective defaults to
                                                       # third-person here. A legacy .doc is a 400 (save it as .docx); a
                                                       # .docx with no text is a 422 (images and text boxes are not read).

faces compile:doc          <alias>  (--content TEXT | --file PATH...)  --medium MEDIUM  [--label]  [--perspective first-person|third-person]  [--timeout N]  [--rush]  [--no-wait]
                                                       # --file is repeatable: each file becomes its OWN document.
                                                       # Returns {"documents":[{file, document_id}...]} in input order; a
                                                       # bad file is reported in place and the rest still compile.
                                                       # Text only: for PDF, Word, audio or video use compile:upload.
                                                       # A source that would cost $1 or more to compile on credits is noted,
                                                       # with its price, before the compile starts (compile:doc:make too).
faces compile:doc:create   <alias>  [--label]  (--content TEXT | --file PATH)  --medium MEDIUM  [--perspective first-person|third-person]
faces compile:doc:make     <doc_id>  [--timeout N]  [--rush]  [--no-wait]
                                                       # A compile may have a scheduled start: the answer's starts_at,
                                                       # null when it starts at once. Without --no-wait the CLI waits
                                                       # for it, and --timeout counts from the start. --rush starts it
                                                       # now at the rush rate (rush: true); see compile:stats. --rush
                                                       # on a compile already queued starts it now; once it has
                                                       # started: 409 compile_already_started, nothing changes.
faces compile:doc:pause    <doc_id>  [--no-wait]  [--timeout N]
faces compile:doc:reset    <doc_id>  [--yes]       # withdraw it from the face, keep the document
faces compile:doc:list     <alias>  [--verbose]    # text left out unless --verbose (--json always has it)
faces compile:doc:get      <doc_id>  [--status-only]   # poll with --status-only: it leaves the text out
faces compile:doc:edit     <doc_id>  [--label]  [--content TEXT | --file PATH]  [--medium MEDIUM]  [--perspective first-person|third-person]
                                                       # a document's medium can be changed, never cleared
faces compile:doc:delete   <doc_id>  [--yes]

faces compile:thread:create   <alias>  [--label]  [--oauth-only]     # a built-in interview; see INTERVIEWS.md
faces compile:thread:list     <alias>
faces compile:thread:get      <thread_id>  [--status-only]           # poll with --status-only: it leaves the messages out
faces compile:thread:edit     <thread_id>  [--label TEXT]  [--medium MEDIUM | --clear-medium]  [--face-speaker NAME]
                                                       # --medium sets what the thread is read as, for style builds.
                                                       # --clear-medium removes it. --face-speaker remaps whose lines are
                                                       # the face's.
faces compile:thread:replace  <thread_id>  --file PATH  [--face-speaker NAME]
                                                       # REPLACE every message, keeping the thread's id: how a growing
                                                       # conversation stays ONE thread. Takes a "SPEAKER: text" transcript
                                                       # or a JSON array of {role, content, name}. Writes content only:
                                                       # nothing generated, nothing billed. Then compile:thread:make.
faces compile:thread:message  <thread_id>  -m MSG  [--oauth-only]
                                                       # NOT the same thing: this continues a built-in INTERVIEW. It
                                                       # stores your answer AND writes the next question, and it bills.
faces compile:thread:make     <thread_id>  [--timeout N]  [--rush]  [--no-wait]    # starts_at and --rush as compile:doc:make
faces compile:thread:pause    <thread_id>  [--no-wait]  [--timeout N]
faces compile:thread:reset    <thread_id>  [--yes]    # withdraw it from the face, keep the thread
faces compile:thread:delete   <thread_id>  [--yes]

faces compile:all         [--timeout N]  [--rush]
                                                       # a compile with a scheduled start is left to start on its own and
                                                       # listed as scheduled, with its start time; --rush starts them now
faces compile:status      [--watch]  [--interval SECONDS]
                                                       # what is compiling on the ACCOUNT, grouped by face, in one small
                                                       # call. Branch on sources_active. --watch polls until nothing is in
                                                       # flight. Paused sources are listed apart, under `paused`.
faces compile:stats                                    # this month's compiling against the allowance, and the rates,
                                                       # standard and rush

faces style:upload      <alias>  <file>  [--type jsonl|email_jsonl|thread_json]  [--messages-per-thread N]  [--strict]
                                                       # load a corpus of the person's OWN writing. Stores only: nothing
                                                       # compiled, nothing billed. ONE MEDIUM PER FILE.
faces style:make        <alias>  (--all | --source ID[:MEDIUM]...)  [--medium MEDIUM]  [--model MODEL]  [--allow-paid]  [--no-compile]  [--timeout N]  [--no-wait]
                                                       # build a style per medium and install it. Chat applies it when a
                                                       # request is in that medium. Charged to credits unless it runs
                                                       # on a linked ChatGPT account (Connect or Pro): --allow-paid.
faces style:status      [JOB_ID]  [--face ALIAS]  [--limit N]  [--wait]  [--timeout N]
                                                       # one build, or a face's recent builds (how to find a lost job id)
faces style:versions    <alias>                        # the styles built, per medium, and which is in use
faces style:revert      <alias>  [--medium MEDIUM]  --yes   # back to the previous version of one medium's style (a toggle)
faces style:delete      <alias>  [--medium MEDIUM | --all]  [--yes]
                                                       # delete a style build. Sources, and what the face learned
                                                       # from them, are kept.

faces catalog:doctor      [--fix]
faces catalog:list
faces catalog:backup
faces catalog:restore     [FILE]  [--compile]
faces catalog:manyfaced   [--skill NAME]  [--install NAME --skills-dir PATH]  [--refresh]

faces keys:create   --name  [--expires-days N]  [--budget F]  [--face ALIAS]...  [--model NAME]...  [--no-save]
faces keys:list
faces keys:revoke   <key_id>  [--yes]
faces keys:update   <key_id>  [--name]  [--budget F]  [--reset-spent]

faces billing:balance
faces billing:subscription                             # plan, and on Connect or Pro a meter of each allowance
faces billing:subscription:activate  [--plan connect|pro]    # alias billing:subscribe. First subscription, or keep one set to cancel.
faces billing:subscription:plan      <connect|pro>  [--yes]  # already subscribed: switch in place, prorated
faces billing:subscription:cancel    [--yes]
faces billing:usage      [--group-by api_key|model|llm|date]  [--from DATE]  [--to DATE]
faces billing:topup      --amount F  [--card PM_ID]  [--yes]  [--allow-duplicate]
                                                       # a repeat of a recent top-up is a 409 duplicate_topup unless
                                                       # --allow-duplicate is given. --yes does not imply it.
faces billing:cards
faces billing:card-setup [--no-default]                # alias billing:card:add: a Stripe link to save a card
faces billing:card:default  <PM_ID>
faces billing:card:remove   <PM_ID>  [--yes]
faces billing:llm-costs  [--lab LAB]                   # one row per model, under its lab's id

faces group:create      --name  [--description TEXT]  [--protocol TEXT]  [--protocol-file PATH]  [--tag TAG...]
faces group:list
faces group:get         <group_id>
faces group:edit        <group_id>  [--name]  [--description TEXT]  [--protocol TEXT]  [--protocol-file PATH]
faces group:delete      <group_id>  [--yes]
faces group:members     <group_id>                     # a member whose owner unpublished, unshared or deleted it stays
                                                       # LISTED, marked unreachable, with the reason
faces group:add         <group_id>  --face ALIAS | owner:alias  [--face ...]
                                                       # your own faces, published ones, and ones shared with you. Name
                                                       # someone else's as owner:alias. Never copy a face to add it.
faces group:remove      <group_id>  <alias>

faces group:tag:list    <group_id>
faces group:tag:add     <group_id>  --tag TAG  [--tag TAG...]
faces group:tag:set     <group_id>  --tag TAG  [--tag TAG...]
faces group:tag:remove  <group_id>  <tag>

faces account:state
faces account:preferences [KEY] [VALUE]  [--allow-paid]
                                                       # KEY: api_fallback, default_model, auto_style, auto_style_thresholds

faces config:set    <key> <value>
faces config:show
faces config:clear  [--yes]                            # never run this: see SKILL.md
```

## Fidelity (`fidelity:*`)

Fidelity runs compared chat answers; starting or advancing one is retired, a `410`. Runs already made can still be read and saved:

```bash
faces fidelity:get <run>                          # progress, every turn, the score
faces fidelity:save-thread <run>                  # save a finished run as a thread you can compile
```

## Default model (`--default-model`)

`--default-model` on `face:create` and `face:edit` stores the face's preferred model for
clients to read: `/facechat` chats on it, as `@alias:maestro-1/<default model>`. The API never
fills it in: `model` always names the downstream model.

## The face's cause

A face's **cause** is the principal *why* behind its response in this context. It steers the
downstream model on chat and decisions, and stays on the server: no response carries it, and
`POST /v1/causes` is removed (`422`). To bring one face's view into another, ask it with `faces
choices` and put its options in the other's situation, or ask a composite.

## States and compaction

A **state** is the situation your agent builds, kept on the server: premises (named facts, each
with a status and what it depends on), the decisions faces made, and your messages. Over HTTP it
is `/v1/states` (`/v1/conversations` is a deprecated alias). Hand it to a face with
`faces choices --state-id`; join `chat:*` and `decide` to it with `--conversation`.

```bash
faces state:create @alice
faces state:add fs_... --premise rent_rises='"12% in March"' --premise-status rent_rises=assumed \
  --premise-description rent_rises="the landlord says rent goes up 12% in March"
faces state:add fs_... --premise can_afford=false --premise-depends-on can_afford=rent_rises \
  --premise-description can_afford="the shop can carry the higher rent"
faces decide @alice:maestro-1/jev-latest --conversation fs_... --depends-on can_afford --state "Decide about the lease." \
  --question "Move the shop to a cheaper street before March?"
# ...
# decision id (answer): dec_...
faces state:add fs_... --premise rent_rises='"no rise this year"' --premise-status rent_rises=invalidated
# 3 statuses changed:
#   premise rent_rises: assumed -> invalidated
#   premise can_afford: stated -> invalidated (because rent_rises)
#   decision dec_...: decided -> invalidated (because can_afford)
faces choices @alice:maestro-1 --state-id fs_... --question "What should the shop do?"
```

**Premises.** `--premise key=value` (the key is 1 to 64 letters, digits, `_` or `-`; the value
is JSON when it is JSON, else text), with `--premise-description` the first time a key is used.
`--premise-status key=status`: `stated` (default), `verified`, `assumed`, `unresolved`,
`invalidated`. `--premise-depends-on key=a,b`: premise keys and decision ids already in the
state or set in the same request, at most 200; `key=` clears it, and a newer value without it
keeps the earlier one. A premise cannot depend on itself.

**Status.** Each premise has the status the agent set; each decision is `decided`. Read back,
`status` is the **effective** one: `invalidated` when its own is, or when anything it depends on
is, directly or through others. Only `invalidated` spreads. Setting an invalidated premise to
another status restores what depended on it, unless something else it depends on is still
invalidated. `asserted_status` is the agent's own when it differs; `because` names what
invalidated it. A write's response lists `status_changes`: `[{kind, key | id, from, to,
because}]`.

**Decisions in a state.** `decide --depends-on a,b` says what a decision rests on; it is shown
those and invalidated with them. Without it, it rests on every premise in force. The response
carries `decision_ids` (printed as `decision id (<question>): dec_...`) to name in a later
`--depends-on`. When a premise's value changes, `decisions_made_under_old_values` lists the
decisions that rested on the old value, with `via` naming a decision that carried it. Nothing is
decided again for you.

**Compaction.** `faces state:compact fs_... [@face]` keeps one face's position over the state on
the server under a compaction id (`fcmp_...`); `--compaction fcmp_...` on `chat:*` or `decide`
reads it. The response carries only the id. Billed on the tokens it reads and writes.

Opening, reading and adding to a state are free. A state expires 30 days after its last use. A
`404` on a state means unknown, expired, closed, another account's, or opened with another face.

## The `me` face

Every account has one face aliased **`me`**: the account's own face, made with the account.
Another account's is `<username>:me` (in a model, `@<username>~me`), readable only if that account publishes it or shares it.

- **It starts with the account's name only:** no description, attributes or default model, not
  published, and nothing compiled. Until a source is compiled into it, asking it is a `409`.
  Chat names the model: `@me:maestro-1/gpt-5.4-mini`.
- **Edit it like any face:** `faces face:edit me --name ... --description ... --attr key=value`.
- **It cannot be made or removed:** creating another face aliased `me` is a `400`, and
  `face:delete me` is a `409`. Publishing, sharing, locking, tags and groups work as on any
  face.
- **Nothing goes to it by itself:** uploads, compiles and style builds go to the face named. To
  build the user's own style: `faces style:upload me <file>`, then `faces style:make me --all`.

## System & published faces

Besides your own faces, every account can use a curated set of **system faces** under the
`head` account, and any face another account has published or shared with you:

| Command | Shows |
|---|---|
| `faces face:list` | your own faces |
| `faces face:list --public` | your faces **plus** every published face |
| `faces face:list --shared` | your faces plus those shared with you |
| `faces face:list --system` | only the system faces |
| `faces face:list --from-users alice,bob` | only these owners' faces |
| `faces face:list --not-from-users head` | everyone's but these owners' |

`--from-users` and `--not-from-users` are repeatable or comma-separated, and cannot be combined.
In `--json` a published face carries `published: true` and `owned_by`; build the handle
`@owned_by~alias` from those rather than assuming `head`, since the set and its owner can change.

**Asking one** uses `@owner~alias` in the model:

```bash
faces choices @head~socrates:maestro-1 --state "A chain bakery opened across the street." --question "What now?"
faces chat:chat @head~socrates:maestro-1/gpt-5.4 -m "What is virtue?"
faces choices @logician:maestro-1 --state "..."      # your own faces need no owner
```

The same works on `decide` and `state:create`. `face:get`, `face:neighbors`, `face:avatar:*` and
`group:add` take the face as `owner:alias`, with no `@`. The **caller** pays at normal rates; the owner is never charged.

## Run-time composite faces

Ask a **composite composed on the fly**, without persisting it, by putting a formula where
the face goes:

```bash
faces choices "(@socrates | @nietzsche):maestro-1" --state "..." --question "What is virtue?"
faces chat:chat "(@socrates | @nietzsche):maestro-1/gpt-5.4" -m "What is virtue?"
faces decide "(@alice | @bob):maestro-1/jev-latest" --state "..." --question "..."
```

Operators: `|` union, `&` intersection, `-` difference, `^` symmetric difference. Parentheses
group and nest: `((@a | @b) & @c)`. The maestro follows the whole formula, once. Works on
`faces choices`, `faces chat:*` and `faces decide`, priced like one face. Quote it: the shell
reads `|` and `&`. A saved composite (`face:create --formula`) works the same way everywhere.

Rules:

- **An empty result is a `422 empty_composite`**:
  `{"detail": {"error_code": "empty_composite", "message": "the intersection between these two faces is null"}}`.
  Branch on `error_code` and show `message` as it is. A composite with nothing compiled is a
  `409`: each face in it needs a source.
- **Operands are concrete faces**, not composites (no nesting: `422`). A bare alias is one of
  your own; `@owner~alias` works in a run-time formula for a face you can read
  (`(@alice | @head~socrates)`). A face you cannot read is a `404`. A persisted formula
  takes only your own faces.
- **Difference:** a `-` right before `@` or `(` is the difference, with or without spaces
  (`@a-@b`, `@a - @b`). A dash inside a name (`@news-bot`) is part of the alias; no alias starts
  or ends with one.

Billing is unchanged: the caller pays. A scoped API key may use a composite only if it may use
*every* operand; otherwise it is a `403` naming the one it may not.

## OAuth-only mode (`--oauth-only`): Connect and Pro

`--oauth-only` and the `api_fallback` preference matter on **Connect and Pro** with a linked
ChatGPT account. `--oauth-only` guarantees an interview's model is not paid from credits: a
request the ChatGPT route cannot serve is refused with `422 oauth_only_unavailable`, and
nothing is charged. Available on `compile:thread:create` and `compile:thread:message`.

`api_fallback` is the account-level setting: `false` by default on Connect and Pro, so a
ChatGPT failure or a used-up allowance is refused rather than billed. On Free it is always
`true`, and setting it to `false`, or passing `--oauth-only`, is a `422`.

```bash
faces account:preferences api_fallback false    # the default on Connect and Pro
faces account:preferences api_fallback true     # allow paying from credits when ChatGPT refuses
```

## Account preferences

| Key | Values | Effect |
|---|---|---|
| `api_fallback` | `true` / `false` | When false, a ChatGPT failure or a used-up allowance is refused instead of paid from credits |
| `default_model` | any chat model | The model new faces store as their default when `--default-model` is not given |
| `auto_style` | `on` / `off` / `default` | Build each face's style automatically once it has enough of its own writing in a medium. `default` follows the plan: on for Connect and Pro, off for Free |
| `auto_style_thresholds` | `medium=words,...` | Words of the face's own writing before an automatic build, per medium. At least 500; `default` resets one. Merged into what is stored |

```bash
faces account:preferences                                # view
faces account:preferences auto_style off                 # stop future automatic builds
faces account:preferences auto_style default             # back to the plan's default
faces account:preferences auto_style on --allow-paid     # needed unless ChatGPT is linked on Connect or Pro
faces account:preferences auto_style_thresholds "email=25000,essay=default"
```

Read back, `auto_style` is the **effective** answer (is it running now) and
`auto_style_setting` is what was stored (`null` means the plan's default).
`auto_style_thresholds` lists every medium with defaults filled in. **Leave `auto_style` alone
unless the user asks.**

## Decisions (`decide`)

`faces choices` gives you the options a face would weigh. `faces decide` gives you what it would **decide**:
describe a situation, ask typed questions about it, and get back numbers, as this face. The
face steers a **decision model**, named after the `/`: `@alice:maestro-1/jev-latest`.

```bash
faces decide @alice:maestro-1/jev-latest --state "A colleague asks you to put your name on a grant proposal you have not read, due in an hour. They say it is a formality." \
  --question "Does the speaker sign the proposal?" \
  --option yes="signs it as asked" --option no="declines to sign" --option later="asks for time to read it first"
# answer: no, confidence 0.90
#   no      96.6%  ###################
#   later    2.6%  #
#   yes      0.9%
```

Three question types. With `--question`, the type follows from the flags, or say it with
`--type`:

| Type | Flags | Answer (`--json`) |
|---|---|---|
| `noul` | neither `--option` nor `--level` | `{type, noul}`: the probability of yes, 0 to 1 |
| `choice` | `--option name="description"` (2 to 255; a bare name sends no description) | `{type, choice, probabilities, confidence}` |
| `score` | `--level "description"` (2 to 10, lowest first) | `{type, score, legend, probabilities, confidence}`: the expected level, 0 to n-1 |

For several questions, write them to a file in the API's shape and pass `--questions-file`:

```json
{"sign":  {"type": "choice", "instructions": "Does the speaker sign?", "criteria": {"yes": "signs it as asked", "no": "declines to sign", "later": null}},
 "irked": {"type": "noul", "instructions": "Is the speaker irritated?"},
 "firm":  {"type": "score", "instructions": "How firm is the reply?", "criteria": ["yields", "hedges", "firm"]}}
```

At most **10 questions** per request. A noul's `criteria` are optional,
`{"true": "...", "false": "..."}`.

- **`--state`** is the situation, written the way the face would hear it. `--state-file` reads
  it from disk; a `.json` file is sent as the JSON it holds (an object or an array).
- **Criteria do the work.** `no` alone is ambiguous; `declines to sign` is not. Keep options
  mutually exclusive.
- **The decision model** goes after the `/`: `jev-latest`, `clef` or `clef-flash`
  (`faces models:list --kind decision`). Call them "decision models" to the user. The face can
  be a saved composite or a formula (`"(@a | @b):maestro-1/jev-latest"`), priced like one face.
  A decision model with no face is not served.
- **`gpt-6-luna`** is OpenAI's Decisions model: the CLI sends it in OpenAI's format on its own
  (it isn't served in System One). It's paid from credits only.
- **Images** aren't supported in decisions or with a face (`422`).
- **`--format`** picks the request shape: `systemone` (the default; TypeSafe's Jev System One,
  `POST /v1/systemone`) or `openai` (OpenAI's Decisions format, `POST /v1/decisions`). The
  answers are the same; `--json` returns that format's own shape.
- **`--conversation` / `--compaction`** attach the decision to a state: it reads the state so
  far and is stored with the premises in force. `--depends-on` says what it rests on; see
  States and compaction. (`--session` is the old name for `--conversation`.)
- **History:** outside a state, decisions keep none between requests. To chain them, put
  the earlier situations and what was decided into the state.

With `--json` (systemone) the result is `{"model", "answers", "answered_by", "usage"}`,
`answers` keyed by question id (`--id` renames the default `answer`), `model` as sent
(`@alice:maestro-1/jev-latest`), and `answered_by` the decision model's versioned id
(`jev-1.14.0`).

**Timing.** A face's answer takes about 10 to 20 seconds. Allow at least
300 seconds. Past about 40 seconds the server answers `200` with `X-Faces-Held: 1`, sends spaces
to keep the connection open, then the JSON. A held call that fails still answers `200`, with
`{"detail": ..., "error": {"code": ..., "status": ...}}` and no `answers`; the CLI reports the real status and code.
A request still working after 300 seconds is stopped with a `504`; retry. A `503` means the
decision model failed: retry, or name another decision model. Do not add a retry loop.

**Cost.** The steering costs $0.42 per million input tokens it processes plus $0.00042 a call
(10 times that on `maestro-1-flash`), with no output charge, against a Connect or Pro allowance
or paid on Free. The decision model is paid from
credits at the model's listed price plus 5% **on every plan**, so a decision needs credit even on
Connect and Pro: with none it is a `402` before any work. Nothing is charged for a request
refused before work starts (`400`, `402`, `403`, `409`, `422`, `429`).

| Code | Meaning | What to do |
|---|---|---|
| `400` | The decision model after the `/` is not one we serve | Name one from `faces models:list --kind decision` |
| `402` | No credit for the decision model | `faces billing:topup` |
| `404` | No such face, or not yours to use | Check the alias |
| `409` | The face has nothing compiled | Compile a source into it |
| `422` | Not `@face:maestro-1/<decision model>`, the wrong shape, the other format's body, or over a size limit | Read the message: it shows the form to use |
| `422 empty_composite` | The formula's result is empty | Show `message`; use a formula whose result is not empty |
| `429` naming an allowance | The plan's allowance is used up and `api_fallback` is off | Wait for the reset, or turn on `api_fallback` |
| `429 Rate limit` | The account's per-minute limit | The CLI waits `Retry-After` once; then wait the seconds named |
| `504` | The face did not finish within 300 seconds | Retry |
| `503` | The steering failed, or the decision model is unavailable | Retry, or another decision model |

## Benchmark (`benchmark:*`)

A benchmark run puts a face you own against a reference model, round after round, on the face's
own writing, and reports which of the two sounds more like the person. The face needs something
compiled **and** a built style.

```bash
faces benchmark:start alice --dry-run                       # price estimate; starts and charges nothing
faces benchmark:start alice --rounds 50 --max-spend 5 --wait
faces benchmark:get RUN_ID --wait
faces benchmark:list --face alice
```

- `--model` is what the face answers with and `--reference-model` what it is compared against;
  both default to a model the server chooses, and the run reports which. `--rounds` is 1 to 300
  (default 100).
- **Price:** run `--dry-run` first. `--max-spend USD` is a hard ceiling: the run stops there and
  reports what it scored. Paid from credits; on Connect or Pro with ChatGPT linked, calls on
  Codex-eligible models run on the ChatGPT plan, except the reference model's answers, which are
  always paid from credits. A balance below what it needs is a `402` before anything runs.
- **Reading it:** `winner` is `face`, `reference`, or `null` when the run did not resolve a
  difference. `gap` is the face's score minus the reference's, and `ci95.resolved` says whether
  the 95% interval excludes zero: report a winner only when it does. `by_length` and
  `by_medium` give the same figures per slice; a slice with fewer than 20 rounds never resolves.
- **Repeat a run** with `--seed` (every result reports its seed): the same face, rounds and seed
  draw the same rounds, so the two compare.
- Refused up front: `422 benchmark_no_compiled_corpus` (nothing compiled), `422
  benchmark_no_style` (no built style), `402 benchmark_insufficient_credit`.

## Face description (`--description`)

A plain-text bio, up to 1,500 characters, shown in listings. It is metadata: it does not change
what the face does. Synced to the local FACE.md.

```bash
faces face:create --name "Ada" --alias ada --description "Theoretical physicist and first-principles thinker"
faces face:edit ada --description "Updated bio"
```

`catalog:doctor --fix` pulls descriptions from the server.

## Face specialty (`--specialty`)

What the face is known for, in a few words: the one line a card shows under the name. At most
80 characters. Metadata only.

```bash
faces face:create --name "Ada" --alias ada --specialty "first-principles physics"
faces face:edit ada --specialty ""                   # clears it
```

## Per-face system prompt (`profile_addendum`)

A face can store a system prompt fragment. It is sent to the model on every chat request with
that face; choices and decisions do not use it. It is returned to the owner.

```bash
faces face:create --name "Support Bot" --alias support-bot --default-model claude-sonnet-4-6 \
  --profile-addendum "Always answer in under 3 sentences. Never promise refunds."
faces face:create --name "Support Bot" --alias support-bot --profile-addendum-file ./prompt.txt
faces face:edit support-bot --profile-addendum "New instructions."
faces face:edit support-bot --clear-profile-addendum
faces face:get support-bot            # a preview and its length
faces face:get support-bot --full     # the whole prompt
```

- At most 100,000 characters (`422` over it). Formatting is kept; control characters are
  stripped.
- `--profile-addendum`, `--profile-addendum-file` and `--clear-profile-addendum` are mutually
  exclusive; leaving all three out leaves it unchanged.
- **Private:** returned only to the face's owner, never to anyone it is shared or published
  with.
- **Composites:** a persisted composite uses its own; a run-time composite borrows its **first
  operand's**.

## Face tags (`--tag`)

Lowercase labels, at most 64 characters, at most 32 per face.

```bash
faces face:create --name "Ada" --alias ada --tag research --tag physics
faces face:tag:set ada --tag research --tag updated      # replace all
faces face:tag:add ada --tag new-tag                     # add (idempotent)
faces face:tag:remove ada old-tag
faces face:tag:list ada
faces face:tag:all                                       # every tag across your faces
faces face:list --tag research --tag physics             # filter (AND)
faces face:list --group GROUP_ID                         # filter by group (OR), composes with --tag
faces face:list --include tags,groups,profile            # extra fields
faces face:groups ada --group GROUP_ID_1 --group GROUP_ID_2  # set a face's groups (replaces all)
```

## Groups

Named groups of faces with an optional description, protocol (usually a mermaid diagram) and
tags.

```bash
faces group:create --name "Review Panel" --description "Research critique panel" --tag review
faces group:add GROUP_ID --face alice --face bob
faces group:add GROUP_ID --face head:socrates       # a face you do not own, by owner:alias
faces group:list                                    # your personal group is left out
faces group:edit GROUP_ID --protocol-file workflow.mmd
faces group:members GROUP_ID
faces group:remove GROUP_ID alice
faces group:tag:add GROUP_ID --tag urgent
```

Each group also has a local `~/.faces/teams/<slug>/TEAM.md`, with YAML frontmatter (id, name,
description, tags, members) and the protocol as the body. See the TEAM.md schema in SKILL.md.

## Compiling documents (`compile:doc`)

`compile:doc` creates a document and compiles it in one step, with progress.

**A document is the face's own writing** (an essay, letter, paper, book, poem), and every word
in it is read as the face's. An interview, transcript, dialogue or any other exchange between
speakers is a **thread**: upload it with `compile:upload --kind thread --face-speaker NAME`, so
only the face's turns are used. A labelled dialogue uploaded as a document is read as entirely
the face's own words, other speakers included.

```bash
faces compile:doc alice --file essay.txt --medium essay --perspective first-person
```

```
Creating document... done (abc123)
Compiling:
  compiling, 43%
  compiled
Done.
```

A source waiting behind the face's other sources prints `queued` until its turn. For a document
that already exists, use `compile:doc:make <doc_id>`. `--timeout` is the polling timeout in
seconds (default 600).

**`--medium` is required.** Every document says what kind of writing it is, and the server will
not save one without it (a `422` naming the eleven). Eleven values, spelled exactly:
`academic paper`, `blog post`, `conversation`, `email`, `essay`, `lecture`, `legal document`,
`other`, `social post`, `text message`, `thread reply`. Only case and surrounding spaces are
forgiven; any other word is a `422` naming all eleven. Never guess one: ask.

- `conversation` is the face speaking informally on their own: a voice memo, dictation. A
  conversation with anyone else is a thread, not a document. `lecture` is prepared speech
  nobody interrupts: a talk, a sermon, a keynote.
- **`other`** is for writing in none of the named forms, read as prose with no form-specific
  conventions. It is not a shrug: a letter, a diary entry or a memo has rules of its own, and
  filing it under `other` is the same mistake as filing it under `essay`.
- A document's medium can be changed with `compile:doc:edit <id> --medium <medium>`, from the
  next compile, but never cleared.

`--medium` is required on `compile:doc`, `compile:doc:create`, `compile:upload --kind document`
and `compile:import --type document`: it says what the source is.

**`--perspective`: by them or about them?** `first-person` for the person's own words (essays,
letters, interview answers, talks); `third-person` for material *about* them (a biography, an
encyclopedia article, a news profile, anyone else's writing about them). **The default differs
by command, so always pass it:**

| command | default |
|---|---|
| `compile:doc`, `compile:doc:create` | `first-person` |
| `compile:upload --kind document`, `compile:import --type document` | `third-person` |
| `compile:doc:edit` | unchanged unless passed |

Third-person material compiles, but never goes into a style. Fix either mistake afterwards with
`compile:doc:edit ID --perspective ...`.

## Style: how a face writes (`style:*`)

Every source has a medium. A **style build** reads a face's own writing in one medium and
installs a style for that medium, so the face writes better in it. A face holds a separate style
per medium. Compiling and building a style are separate acts: most faces only need compiling. A
style is worth building when the face is a real person and you have their own writing: mail,
messages, essays. Writing *about* someone never goes into a style.

```bash
# 1. Load material. Stores only: nothing compiled, nothing billed.
faces style:upload alice ./mail.jsonl

# 2. Build from the threads that upload made, named by id.
faces face:sources alice --type thread --json | jq -r '.threads[] | "\(.id)  \(.medium)  \(.label)"'
faces style:make alice --source THREAD_ID_1 --source THREAD_ID_2

# 3. Check it
faces style:versions alice
```

### Using a built style

Chat applies a built style on its own: the server reads what each request is (an email, an
essay, a text message, or none of these) and serves the face's style for that medium when it has
one. Nothing is sent to ask for one; `style`, `medium` and `kind` on a request are refused (`422`).

- Every face carries `styles`: the media it has a built style for. `face:get` prints them,
  `face:list --has-style` filters to faces with any, and in `--json` it is a list:

```bash
faces face:list --json | jq '.data[] | {alias, styles}'
```


### The corpus file

`style:upload` takes one JSON object per line, one message each (`--type jsonl`, the default),
or a JSON array that becomes one thread (`--type thread_json`). Each message needs `author`
(`self` when the face's subject wrote it, `other` when someone else did), `sender`, `date`
(ISO-8601) and its text in `plaintextBody`, `htmlBody` or `snippet` (`body` and `text` are read
as `plaintextBody`). `kind`, if present, must be one of the eleven media.

```json
{"author": "self", "sender": "alice@example.com", "date": "2026-03-14T09:12:00Z", "kind": "email", "snippet": "Friday works. I'll bring the draft and we can cut it down together."}
{"author": "other", "sender": "bob@example.com", "date": "2026-03-14T08:40:00Z", "kind": "email", "snippet": "Can we meet Friday to go over the proposal?"}
```

- Invalid lines are skipped and reported, each with every problem it has; `--strict` refuses
  the whole file instead.
- **Upload one medium per file.** The output says what medium the new threads carry. A file
  that mixes media, or whose messages declare none, makes threads with **no medium**, which no
  style can be built from until one is set: `faces compile:thread:edit THREAD_ID --medium
  lecture` (`--clear-medium` removes it). Check that line rather than assuming.
- `--messages-per-thread` (default 50) sets how many messages go in each thread. Set it above
  the length of a conversation you want kept whole.

### Building

- **Name the sources** with `--source ID` (repeatable) or `--source "ID:MEDIUM"`, or take
  everything ready with `--all`. `--all` takes every document and thread on the face,
  interview threads included, so use it only when everything on the face is the subject's own
  writing in one medium. List the sources with `faces face:sources alice`.
- **Declare the medium.** `--medium` fills in for sources that declare none, and never overrides
  one that does. A source with no medium is refused, not guessed. To change what a document
  is, edit the document (`compile:doc:edit DOC_ID --medium essay`).
- **Quote `"ID:MEDIUM"` when the id is a shell variable.** In zsh, `$DOC_ID:essay` is read as a
  history modifier and silently becomes `ssay`.
- A build whose sources are **all** third-person is refused (`422
  style_sources_all_third_person`). A mixed build runs and reports what it left out.
- By default, uncompiled sources are compiled first, billed as ordinary compiles;
  `--no-compile` builds from what is already compiled.

**Cost.** On Connect or Pro with a linked ChatGPT account, a build runs there at **no charge**.
Anywhere else (Free, or Connect or Pro with no ChatGPT linked) it is **charged to credits** at
the build model's listed price plus the published markup, and appears in `billing:usage` under
that model.

- **Without `--allow-paid`** the CLI asks for the linked ChatGPT account or nothing. A build that
  cannot run there is refused up front with `409 chatgpt_not_linked` (on Free,
  `422 oauth_only_unavailable`), and nothing starts. Link ChatGPT with `faces auth:connect
  openai`, or pass `--allow-paid` if the user agrees to pay.
- **With `--allow-paid`** a build credit cannot cover is refused up front with
  `402 insufficient_credit_for_style_build`: `estimated_usd` is the most it is expected to cost,
  `balance_usd` the balance. Nothing is started or charged. Top up with `faces billing:topup
  --amount <USD>`.
- **`--model`** is used only on the ChatGPT link, and only for a model that can run there
  (`codex_eligible`). A build charged to credits runs on the default build model, whatever was
  named; the CLI refuses a named model that cannot run on ChatGPT unless `--allow-paid` is given.
- If a build fails with `CHATGPT_AUTH`, the ChatGPT link has expired: `faces auth:connect
  openai`, then build again. A link that fails part-way under `--allow-paid` carries on charged
  to credits for the rest.

**Long builds.** `style:make` waits and prints progress; `--no-wait` returns a job id. To find a
lost one, list the face's builds:

```bash
faces style:make alice --all --medium email --no-wait
faces style:status --face alice        # recover the job id
faces style:status JOB_ID --wait
```

**One build per face and medium at a time.** Starting another is a `409 build_in_progress`
naming the running job: follow it with `faces style:status JOB_ID --wait`. Do not retry.

**What a finished build reports** (printed by `style:make` and `style:status JOB_ID`, and under
`report` in `--json`):

- `skipped_third_person`: documents left out as writing *about* the subject. If one is really
  their own, fix it with `faces compile:doc:edit ID --perspective first-person` and build again.
- `thin_corpus`: `{medium: {words, threshold}}` for each medium built from less writing than the
  automatic threshold. It installed anyway; a manual build has no minimum.
- `served_by`: where it ran (`route` `chatgpt`, at no charge, or `faces`, charged to credits;
  `model`; `fell_back`).

### Versions, revert, delete

A face holds one style **per medium**, so `style:versions` can show several in use, and version
numbers count per medium:

```bash
faces style:versions alice
# MEDIUM  VERSION  IN USE  REVERTABLE  MODEL          BUILT
# essay   2        yes     yes         gpt-5.6-terra  2026-08-27 20:21:54
# email   1        yes     -           gpt-5.6-terra  2026-08-27 20:23:26

faces style:revert alice --medium essay --yes   # back to the previous version: a toggle
faces style:delete alice --medium email --yes   # delete one medium's style build
faces style:delete alice --all --yes            # delete every style build
```

- `style:revert` toggles between the two versions kept for a medium; name the medium once the
  face has more than one style. A revert with no earlier version is a `409`, a normal answer. A
  build after a revert keeps the version you reverted to as its previous one.
- An unscoped `style:delete` on a face with more than one style is **refused**, naming them.
  Deleting a style build never deletes documents, threads or uploaded material, so it can be
  built again without uploading. What the face learned about how it writes from its documents
  and threads stays too, until they are deleted. **To remove a style completely**, delete that
  medium's documents and threads, then the build. To withdraw a **source** from a face while
  keeping it, use `compile:doc:reset` / `compile:thread:reset`.

### Automatic builds

Once a face has enough of its **own** writing in a medium, a build runs for that medium without
being asked: 10,000 words for essay, academic paper, blog post and legal document; 15,000 for
the rest. It runs again at twice the threshold, then three times, and so on. Only the face's own
messages in uploaded threads and first-person documents count; email and text message count
only from uploaded message threads; `other` never triggers one. It installs at once, like a
manual build, and `style:revert` undoes it. `style:status --face` marks these rows `BY auto`.

They are on by default on Connect and Pro, and off by default on Free. Each is paid like a manual
build: at no charge on a linked ChatGPT account, otherwise charged to credits. One that credit
cannot cover is not started: it shows in `style:status --face` as a failed build with
`insufficient_credit_for_style_build` and is tried again later; top up to let it run. Turning
them on with `account:preferences auto_style on` needs `--allow-paid` unless ChatGPT is linked
on Connect or Pro. Control them with the `auto_style` preferences
([Account preferences](#account-preferences)); turning it off removes nothing already built.

## Compile state: read `finishing` first

A source carries `finishing`, `synced`, `status` and `progress`. **Branch on `finishing` first;
read `status` only when it is false.**

| `finishing` | means | what to do |
|---|---|---|
| `true` | compiling, or queued to resume by itself | show progress, **offer no action**, keep polling |
| `false` | compiled, failed, paused, or never compiled | read `status` to tell them apart |

**Never offer to compile a source that is finishing.** One being retried can report `failed` on
the way; telling someone to re-run it asks them to repeat work in flight. `faces compile:all`
skips them and says how many.

| `status` | meaning |
|---|---|
| `not_compiled` | saved, not compiled yet (or reset); call `/make` |
| `transcribing` | an audio or video source is being transcribed; a transcription or import that ends `failed` is uploaded again |
| `queued` | waiting behind the face's other sources, or about to resume by itself |
| `compiling` | running; `progress` says how far |
| `compiled` | done. For a document, its writing style catches up within about a minute; treat `compiled` as finished |
| `paused` | paused; nothing moves until `compile:*:make` resumes it |
| `failed` | final; `error_code` or `stall_reason` says why (`nothing_extracted`: the source gave nothing to compile; add more of the author's own writing) |

Treat an unknown value as `compiling`. `progress` runs from 0 to 1: 1 only when `compiled`, and
0 while `queued` or `not_compiled`. Show queued work as "queued", never "0%". A temporary
failure goes back to `queued` and resumes by itself.

`/make` is the only compile call. A second `/make` of an unchanged, compiled source is a `409
already_compiled`, and nothing is charged: edit the source or reset it to compile it again.

## Is anything compiling? (`compile:status`)

`finishing` answers it for one source; `faces compile:status` answers for the whole account in
one small call. Use it rather than walking the lists.

```bash
faces compile:status                 # grouped by face
faces compile:status --json          # branch on sources_active
faces compile:status --watch         # poll until quiet, then exit
```

Idle, it is exactly:

```json
{"object":"compile_activity","faces":[],"faces_active":0,"sources_active":0,
 "progress":null,"measurable":false,"paused":[]}
```

With work in hand, each entry in `faces` has `alias`, `sources_active`, `progress`,
`measurable`, and `sources` (`{id, kind, label, status, progress, stall_reason}`). Paused
sources are not in `faces`; each is an entry in the top-level `paused` list, and the CLI prints
the command that resumes it:

```
Nothing is compiling.

1 paused source(s), waiting for you to resume:
  alice  document Letters
         paused, 14%
         resume: faces compile:doc:make 549e…
```

- **A compile started in an earlier turn, or on another machine, is otherwise invisible.** Check
  before reporting work finished.
- **`sources_active: 0` means nothing is RUNNING, not nothing is outstanding.** Queued and
  retrying sources are counted; paused ones are not, and `--watch` does not wait for them. Read
  `paused` before reporting that the work completed.
- **Branch on `measurable` before reporting a percentage.** On a face and on the account,
  `progress` is `null` while it cannot be measured (during a transcription, for one): say
  "working" and give no number. Read `sources_active` first, since `measurable` is false on an
  idle account.
- `faces` is activity, not a roster: a face appears only while it has work.

`--status-only` (on `compile:doc:get` and `compile:thread:get`) is per item; the account-wide
lists do not take it.

## Compile state: is the face still current?

`synced` says a compile ran; these say *what it ran on*:

| | field | meaning |
|---|---|---|
| document, thread | `version` / `synced_version` | the current revision, and the one the compile read |
| thread | `synced_at` | when that compile ran |
| each message | `timestamp` | when that message was last written |

- **A document is stale** when `synced_version < version`, by that many revisions.
- **A thread is stale per message**: a message whose `timestamp` is later than the thread's
  `synced_at` is one the compile never read.
- **A reorder moves `version` but restamps no message.** If the pair says stale and no message
  looks new, the order changed.
- A `null` `timestamp`, or `synced_at` set with `synced_version` null, is **unknown**, not
  stale. Do not send someone to pay for a recompile on that alone.

What moves a thread's `version`: editing, adding, removing or reordering messages. Renaming it,
changing its medium, or sending the messages back unchanged does not. `face:sources` folds this
into its STATUS cell (`stale (2 behind)`), and `compile:doc:get`, `compile:thread:get` and the
lists say what moved.

## `compilable`: whether the action exists at all

Every source carries:

```json
{ "compilable": false, "compile_blocked_reason": "too_short" }
```

| `compile_blocked_reason` | meaning |
|---|---|
| `empty` | a document with no text |
| `too_short` | a document under `compile_min_words` (50) words; it carries `word_count`. Add text or merge it into a longer source |
| `no_user_messages` | a thread with nothing the subject wrote. A fresh interview is this until the first answer |
| `corpus_thread` | uploaded material for building a style rather than a thread to compile. Build a style from it with `style:make`, or upload the same material with `--kind thread` to compile it |

`/make` refuses these with a `400` carrying the same code in `error_code`. **Treat an unknown
code as "cannot be compiled"**: the set grows.

The two booleans compose:

| `finishing` | `compilable` | what to do |
|---|---|---|
| `true` | either | show progress, offer nothing, keep polling |
| `false` | `true` | a compile is available; offer it if the source is stale or never compiled |
| `false` | `false` | say why from `compile_blocked_reason`; never offer a compile |

`compilable` does not say a compile would change anything (that is `synced` and the revision
pair), and does not cover transient refusals: a running compile, a locked face or an empty
balance are refused by other means (`409`, `409`, `402`). `faces compile:all` skips what cannot
compile and reports the counts by reason.

## Deleting sources

`compile:doc:delete` and `compile:thread:delete` remove the source and what it contributed to
the face. Nothing else is needed. To withdraw a source but keep it for later, use
`compile:doc:reset` / `compile:thread:reset`.

## Read-only sources

Documents and threads carry `read_only`. When it is `true`, `compile:doc:list`/`:get` and
`compile:thread:list`/`:get` print a bare `read only` line (in `--json` the field is always
there). A read-only item can be read and deleted, but a write to its **content**
(`compile:thread:message`, `compile:thread:replace`, a content edit) is a `409`:

```
Error (409): This document is read-only and cannot be edited.
```

Do not retry: say it is read-only. Its label and medium are not content, so
`compile:thread:edit ID --label`, `--medium` and `--clear-medium` still work. Threads made by
`style:upload` are read-only. To freeze or thaw a whole face, use `face:lock` / `face:unlock`.

## Face locking

`face:lock <alias>` freezes a face; `face:unlock <alias>` thaws it. A locked face can still be read
and asked, but any write to it or anything it owns (editing it, its documents and
threads, uploads, imports, style builds, deleting it) is a `409` mentioning `read-only`. Do
not retry; unlock first. `face:get` shows a `read only` line and `face:list` a `[read only]`
marker.

## Catalog

The CLI keeps a local catalog at `~/.faces/catalog/`, a `FACE.md` per face (YAML frontmatter
plus your notes), and an index at `~/.faces/catalog.json`. `face:create`, `face:edit` and
`face:delete` keep it up to date.

- `faces catalog:doctor`: missing, stale or orphaned entries.
- `faces catalog:doctor --fix`: rebuild from the server. It rewrites the frontmatter of **every**
  FACE.md and the whole of every TEAM.md, but `Fixed N` counts only entries that were missing or
  renamed, so `Fixed 0` does not mean nothing changed. A FACE.md body is kept.
- `faces catalog:doctor --generate`: `--fix`, plus missing descriptions written by a model and
  saved to the server.
- `faces catalog:backup`: every face (descriptions, tags), group, document (with its medium) and
  thread, to `~/.faces/backups/<timestamp>.json`.
- `faces catalog:restore [FILE]`: restore from a backup (default: the newest). `--compile` runs
  `compile:all` afterwards. A document the backup records no medium for is refused by the
  server; restore names it, and it needs creating again with `compile:doc:create --medium`.
  Never guess the medium.
- `faces catalog:list`: print the catalog.
- `faces config:set catalog false`: turn catalog management off.
- `faces config:set catalog_model MODEL`: the model `--generate` uses.

## Attributes (`--attr`)

`--attr KEY=VALUE` on `face:create` and `face:edit` sets basic facts about the person directly,
without compiling. Repeat it for each fact. A face with nothing compiled answers from them.

**Twelve keys, and no others.** Three are fields with a plain value; nine take a short sentence.

| Key | Format | Holds | Example value |
|---|---|---|---|
| `age` | value | their age | `45` |
| `gender` | value | their gender | `woman` |
| `location` | value | where they live now | `Lisbon, Portugal` |
| `identity` | sentence | name, sexuality, where they are from | `Born in Seoul, raised in Los Angeles.` |
| `education` | sentence | schooling and study | `Holds a master's degree in nursing.` |
| `occupation` | sentence | work and career | `Works as a nurse practitioner at a county clinic.` |
| `religion` | sentence | faith, or its absence | `A practising Buddhist.` |
| `family` | sentence | parents, siblings, children | `Has two teenage sons.` |
| `relationships` | sentence | partner, marriage, close ties | `Married for twelve years.` |
| `health` | sentence | lasting health conditions | `Manages type 1 diabetes.` |
| `languages` | sentence | languages spoken | `Speaks English and Korean.` |
| `background` | sentence | formative history no other key covers | `Spent her twenties in the Peace Corps.` |

Anything else goes in one of these as a sentence (nationality in `identity`, marital status in
`relationships`). **Any other key is a `422` that names it, and nothing in that request is
written**: fix the key and send it all again. `faces face:attributes` lists the keys.

The API calls them `basic_facts`; the CLI shows them as `attributes`. Compiling can update them;
a value you set replaces the current one, and a later compile can still refine it, so check
them after a big upload.

```bash
faces face:create --name "Maria Chen" --alias maria \
  --attr gender=woman --attr age=34 \
  --attr location="Portland, OR" \
  --attr occupation="Works as a nurse practitioner at a county clinic."
faces face:edit maria --attr religion="A practising Buddhist."
```

`--attr` cannot be combined with `--formula`: a composite has nothing of its own.

## Output format: `--json` and `--toon`

Every command that takes `--json` also takes `--toon`: the **same data and field names** in
[TOON](https://toonformat.dev/), in noticeably fewer tokens. Prefer it when the output goes into
a model's context.

```bash
faces face:list --toon      # same fields as --json, fewer tokens
faces face:list --json
faces face:list             # human output
```

The two are mutually exclusive. Errors keep the same structured shape in either.

## Paged listings

`GET /v1/faces` is paged with a cursor. `face:list`, `compile:all`, `catalog:backup` and
`catalog:doctor` follow it to the end, and say so when a list came back short;
`catalog:backup` and `catalog:doctor` refuse outright on a short list. `user:followers` and
`user:following` are followed the same way.

## Errors

A client's mistake is a `422`, not a `500`. Every error has a `detail`; one that ends
`Reference <12 hex characters>` names a logged request: quote it when reporting a failure.
**Branch on the status and `error_code`, never on the message text**, which is written for
people and changes. `503` is the status worth retrying; a `409` means busy or frozen and a
`402` means no credit, and neither is fixed by retrying.

**`may be out of date`:** the CLI ends a refusal with `Your Faces CLI (X) may be out of date.
Update with npm install -g @faces-sh/cli@latest and try again.` when the API looks to have moved under it.
Update and retry; do not work around it.

The CLI prints a refusal as `Error: HTTP <status>: <message>`, followed by a line saying what
to do when there is one.

Refusals that can come from many routes:

- **`429` naming an allowance.** A Connect or Pro allowance (choices and steering over 5 hours, the week or
  the month, or compiling over the month) is used up and `api_fallback` is off. The message
  says which, and when it resets: show it as it is. Wait, or turn on `api_fallback` to pay from
  credits.
- **`429 Rate limit`.** Every route that runs a model or a compile counts against a
  per-account, per-minute limit set by plan (`limits.requests_per_minute` in `GET /v1/rates`).
  Nothing is charged. The CLI waits the `Retry-After` once and asks again; a second refusal is
  final. Parallel agents share the account's limit.
- **`402`.** No credit. On Connect and Pro the answering model and decision models are still
  paid from credits, so this can happen on any plan. Top up.
- **`503 oauth_incomplete`.** ChatGPT stopped its reply partway, twice. Nothing was charged;
  retry.

## Global flags

Any command takes these, **after** the command name (before it they are read as the command):

```
faces COMMAND [ARGS] [--base-url URL] [--token JWT] [--api-key KEY] [--json | --toon]
```

## Environment variables

| Variable | Purpose |
|---|---|
| `FACES_BASE_URL` | API base URL (default `https://api.faces.sh`) |
| `FACES_TOKEN` | JWT |
| `FACES_API_KEY` | API key |
