---
name: facechat
description: >
  Chat with any face in your catalog. Invoke as /facechat <alias> <message>
  (e.g. /facechat socrates How are you?) to send a message directly, /facechat <alias>
  to start a conversation, or /facechat with no argument to browse your catalog and
  pick a face. The face answers in its own voice on any chat model, or proposes the
  options it would weigh. Trigger when the user says "chat with a face", "talk to
  <name>", "ask <name>", or wants a conversation with a compiled face. Also trigger
  on /facechat.
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
  - AskUserQuestion
  - WebSearch
  - WebFetch
---

# /facechat: Chat with a Face

## Preamble

```bash
faces --version 2>/dev/null || echo "NOT_INSTALLED"
LATEST=$(npm outdated -g @faces-sh/cli --json 2>/dev/null | jq -r '.["@faces-sh/cli"].latest // empty')
[ -n "$LATEST" ] && echo "UPDATE_AVAILABLE: $LATEST"
faces auth:whoami --json 2>/dev/null
echo "EXIT:$?"
[ -f ~/.faces/config.json ] && echo "HAS_CONFIG" || echo "NO_CONFIG"
```

If `NOT_INSTALLED` (the auth check then prints `EXIT:127`, command not found): run `npm install -g @faces-sh/cli@latest` and re-run the preamble.

This skill needs the CLI's `faces chat:thread` on the model names `@face:maestro-1/<model>`
(version 1.2.0 or later). If `faces chat:thread --help` says the command is not found, or a
command answers that `face@model` is gone, run `npm install -g @faces-sh/cli@latest`.

If `UPDATE_AVAILABLE`: run `npm install -g @faces-sh/cli@latest`. Once auth triage below reaches `EXIT:0`, run `faces catalog:doctor --fix` to migrate local catalog files. It needs a signed-in account, so before sign-in it fails with "Authentication required"; that is not a broken update.

**Auth triage:**

- `EXIT:0` -> authenticated. Proceed.
- `EXIT:127` -> the CLI is not installed. Install it as above and re-run, before anything else.
- `EXIT:1` + `HAS_CONFIG` -> returning user. Read the whoami output to
  understand what failed. Present the diagnosis and help them fix it. Do NOT
  walk through QUICKSTART or ask about plans.
- `EXIT:1` + `NO_CONFIG` -> new user. Ask the user:

  > You're not logged into Faces, and I don't see any prior config on this
  > machine. Do you already have an account?
  >
  > A) I have an account, and I'll log in now
  > B) I have an API key, and I'll paste it
  > C) No account: help me set one up here
  > D) No account: I'll register at faces.sh myself and come back

  If A: have the user run `faces auth:login --email YOUR_EMAIL --password 'YOUR_PASSWORD'`
  If B: have the user run `faces config:set api_key <key>`, verify with `faces auth:whoami`
  If C: walk through [QUICKSTART.md](../faces/references/QUICKSTART.md)
  If D: tell them to come back with login credentials or an API key

**Secret hygiene:** Never display API keys, tokens, or passwords from config
files. Always mask them (e.g. `sk-faces-...dN`).

---

## Parse arguments

The argument string after `/facechat` is parsed as: first word is the alias,
everything after it is the message. Examples:

- `/facechat socrates Should I take the job?`: alias=`socrates`, message=`Should I take the job?`
- `/facechat socrates`: alias=`socrates`, no message
- `/facechat`: no alias, no message (browse the catalog)

## Resolve the target

The alias can name a **face** or a **team** (several faces with a collaboration
protocol). Check both.

**If alias provided** (`/facechat socrates` or `/facechat socrates Should I take the job?`):

```bash
# Check face first, then team
faces face:get $ALIAS --json 2>/dev/null
echo "---TEAM_CHECK---"
faces group:list --json 2>/dev/null | jq -r '.[] | select(.name == "'$ALIAS'") | .id'
# Local teams live under a slug of the name ("Review Panel" -> review-panel)
SLUG=$(echo "$ALIAS" | tr '[:upper:]' '[:lower:]' | sed -E 's/[^a-z0-9]+/-/g; s/^-+|-+$//g')
ls ~/.faces/teams/$SLUG/TEAM.md 2>/dev/null
```

- A face found: ask the face (see **Ask** below).
- A team found (by name in `group:list`, or a local TEAM.md): ask the team (see **Team** below).
- Both found: the face wins (teams should have distinct names).
- If neither found, check for close matches:

```bash
cat ~/.faces/catalog.json | jq -r '.[].alias'
faces group:list --json 2>/dev/null | jq -r '.[].name'
ls ~/.faces/teams/ 2>/dev/null
```

Suggest the closest match or offer to create the face with `/facemake` or team
with `/faceteam`.

**If no alias provided** (`/facechat`):

List both faces and teams:

```bash
faces face:list --json 2>/dev/null
echo "---TEAMS---"
ls ~/.faces/teams/ 2>/dev/null
```

Present as a numbered list showing alias, name/description, and whether it's
a face or team. Ask the user:

> **Your catalog.** Who do you want to ask?
>
> **Faces:**
> [numbered list of faces]
>
> **Teams:**
> [numbered list of teams]
>
> Pick a number, or type a name/alias.

## Chat

A face is named in one `model`: the face and its maestro, then the chat model that writes the
reply after a slash: `@ALIAS:maestro-1/MODEL`. A face the user does not own is `@OWNER~ALIAS`.
Use the face's `default_model` (from `faces face:get ALIAS --json`) unless the user names
another model; `faces models:list` lists them. Use `maestro-1-flash` in place of `maestro-1` only
when the user asks for speed: it costs 10 times more on the face's part.

Keep the conversation in a thread, so each turn sends what came before:

```bash
faces chat:thread @ALIAS:maestro-1/MODEL -m "MESSAGE" --json     # first turn: keep thread_id
faces chat:thread --id THREAD_ID -m "MESSAGE" --json              # every turn after
```

Show the reply (`assistant` in `--json`) as the face's, under its name. If the user gave no
message, ask:

> **Chatting with NAME.** What do you want to say?

The face's style is chosen for each message by the server (an email reads as an email, a text
message as a text message): never pass a style or medium. Start a new thread when the user
switches faces or models.

### Ask what it would do

When the user wants the face's **options** rather than a reply ("what would X do", "what are my
options as X sees them"), ask with `faces choices`, which takes the face and its maestro alone:

```bash
faces choices @ALIAS:maestro-1 --state "SITUATION" --question "QUESTION" --json
faces choices @ALIAS:maestro-1 --state "SITUATION" --json        # no question: the face proposes one
```

Split what the user wrote into the **situation** (the facts) and the **question**. Show the
options as a numbered list, `name` in bold with its description, strongest first. With no
question, show the face's `question` and its `alternatives`, and offer to ask it. `--state` takes
at most 3,000 tokens of prose. For more, use a state.

### Follow-ups: keep a state

When the conversation builds up facts that should hold from turn to turn, keep them on the
server as a **state**:

```bash
faces state:create @ALIAS --json                         # once; keep the id
faces state:add STATE_ID --premise KEY=VALUE --premise-description KEY="what it means"
faces chat:thread --id THREAD_ID -m "MESSAGE" --conversation STATE_ID --json
faces choices @ALIAS:maestro-1 --state-id STATE_ID --question "QUESTION" --json
```

In a state, a chat turn sends only the new message: the state holds the rest. Write each fact
the user establishes as a premise, with a status when it matters (`--premise-status
KEY=assumed`, `verified`, `unresolved`, `invalidated`) and what it rests on
(`--premise-depends-on KEY=other_key`). When the user says something no longer holds, set it
`invalidated`: everything resting on it is invalidated too.

Prose sent with `--state` beside `--state-id` is an **addendum for that one call**: the face
reads it with the state, and it is **never saved**. Anything that should still hold on the next
question must be written with `faces state:add --premise`.

Start a fresh state when switching faces or when the user clearly begins a new topic.

### Bringing in another face

To bring another face in, chat with it on the same question, or ask a composite:
`faces chat:chat "(@ALIAS | @OTHER):maestro-1/MODEL" -m "MESSAGE"`.

## Team

When the target is a team, read the TEAM.md and execute the mermaid protocol
diagram mechanically.

### Load the protocol

```bash
cat ~/.faces/teams/$SLUG/TEAM.md
```

Parse the YAML frontmatter for the `members:` list (the faces on the team).
Then parse the mermaid flowchart: this is the entire execution spec, round
limits included.

If `members:` holds ids rather than aliases, read the aliases with
`faces group:members GROUP_ID`, or run `faces catalog:doctor --fix` to rewrite
the file.

### Walk the graph

The mermaid diagram uses three shapes. Each has one rule:

- `(alias)` **rounded rectangle**: ask the face:
  `faces chat:chat @alias:maestro-1/MODEL -m "INPUT" --no-stream`, where MODEL is the face's
  `default_model` (or the one TEAM.md names) and INPUT is the user's message and the concatenated
  output from all incoming edges. When the node asks for the face's options rather than a reply,
  use `faces choices @alias:maestro-1 --state "INPUT" --question "QUESTION"`. Format each input
  clearly:
  ```
  [from Query]: <the user's original message>
  [from face-a]: <face-a's options>
  ```
  A member you do not own is `@owner~alias`: `faces chat:chat @owner~alias:maestro-1/MODEL ...`.
  Keep INPUT short; summarize earlier rounds if it grows long (`faces choices` takes at most
  3,000 tokens of `--state`).
- `[text]` **sharp rectangle**: you (the orchestrating agent) carry out the
  instruction described in the text. The text IS the instruction.
- `{text}` **diamond**: you evaluate the condition described in the text
  and follow the matching outgoing edge. The text IS the condition.

**Edge rule:** an edge from A to B means B receives A's output. If B has
three incoming edges, B gets all three outputs as input. No implicit context:
if there is no edge, that data does not flow.

**Entry/exit:** the graph starts at the `[Query]` node (the user's message)
and ends at the `[Response]` node (what you show the user).

### Execution loop

1. Start at `[Query]`: seed it with the user's message.
2. Follow outgoing edges. For each node you reach:
   - `(face)`: call `faces chat:chat` (or `faces choices`) with the concatenated inputs.
   - `[instruction]`: carry out the instruction yourself.
   - `{condition}`: evaluate and branch.
3. If an edge loops back (e.g. round-robin), increment the round counter.
   Stop where the diamond's condition says to (e.g. "or this is round 3"). If
   the diagram names no limit, stop after 3 rounds.
4. When you reach `[Response]`, present the final output to the user.

### Presenting team responses

Show each face's contribution with attribution so the user sees who said what:

> **skeptic:** [reply]
>
> **builder:** [reply]
>
> **strategist:** [reply]
>
> **[Tally/Decision/Final node label]:** [final output]

For pipeline protocols where each face refines the previous output, show only
the final face's response unless the user asks to see the full chain.

### Follow-up messages

For follow-ups in a team conversation, re-run the full protocol with the new
message. Each run is independent: the protocol starts fresh each time.
The user can reference prior responses in their follow-up message naturally.

## Related skills

- `/facemake`: guided face creation
- `/faces`: the core CLI for every platform operation
- `/faceteam`: compose faces into teams
