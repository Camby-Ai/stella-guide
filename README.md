# Stella Setup Guide

Stella is an AI Chief of Staff. If you run more than one AI employee, somebody has to know what each one is supposed to be doing, notice when one stalls, and hand out the next task. Stella does that so you do not have to. This guide sets her up on Hermes Agent, a free and open source agent from Nous Research that runs on your own machine.

The software is free. The model behind it is not always free, and the real work is writing down what each of your AI employees is for.

**Checked on 2026-11-21.** Commands and limits change. Sources are at the bottom.

**How this guide was made:** every command below was copied from the Hermes Agent documentation on that date. We have not test-run them on a fresh machine. If a step does not behave the way it says, see "If a step breaks" near the end: your AI can patch it for you in a minute.

## What Stella does

- Keeps one list of your AI employees and the single goal each one owns.
- Tracks who is hitting their goal and who is slipping, from the task board, not from memory.
- When someone stalls: finds out why, writes clear feedback on the task, and assigns the next task.
- Sends you a short brief every weekday morning: what moved, what is stuck, what needs your decision.
- Does not do the work herself. She assigns it.
- Does not send, post, book, pay or delete anything. She drafts and reports. You approve.

## What you need before you start

| You need | Why | Cost | Checked |
|---|---|---|---|
| A computer that stays on: macOS, Linux, Windows or WSL2 | The board and the morning brief only run while Hermes' gateway is running | Yours already | 2026-11-21 |
| Git and curl | The installer uses them | Free | 2026-11-21 |
| Hermes Agent | Runs Stella and the rest of the team | Free, MIT licence | 2026-11-21 |
| A model with at least 64,000 tokens of context | Hermes rejects smaller models at startup | Depends on the provider you pick. Not measured here | 2026-11-21 |
| Telegram (optional) | To get the morning brief on your phone | Free | 2026-11-21 |

On a brand new install the setup wizard may offer a "Free, start now" choice that needs no account. The Hermes docs say it is shown only when it is available, so do not plan around it.

## The honest timeline

| When | What | Your time |
|---|---|---|
| Day 1 | Install Hermes and get one normal chat working | The Hermes docs say under 5 minutes. Allow 30 |
| Day 1 | Create Stella and at least two AI employees for her to manage | 30 minutes |
| Day 1 to 2 | Write each employee's goal in one sentence | 1 to 2 hours. This is the part that decides everything |
| Day 2 | Turn on the board and the morning brief | 30 minutes |
| Week 1 | Read the brief every morning and fix the goals that turned out vague | 10 minutes a day |

The slow part is not the install. A Chief of Staff with nothing clear to track produces a confident summary of nothing. If you cannot say what an employee is for in one sentence, Stella cannot hold them to it.

## Setup, in this order

### 1. Install Hermes and get one chat working

Do not add anything else until a plain conversation works. The Hermes quickstart is blunt about this, and it is right.

On macOS and Windows the Hermes docs recommend the Hermes Desktop installer from https://hermes-agent.nousresearch.com/. For a command line only install on macOS, Linux or WSL2:

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

Reload your shell, pick a model, and start a chat:

```bash
source ~/.zshrc
hermes setup
hermes
```

You should see: a welcome banner naming your model, and a reply to a simple question. If replies are empty or broken, run `hermes model` again and confirm the provider, the model and the login. If anything else looks off, run `hermes doctor`.

### 2. Create Stella

Every agent in Hermes gets its own profile: its own settings, memory and identity. Never point two agents at the same profile.

```bash
hermes profile create stella --description "Chief of Staff. Assigns work, tracks each employee's goal, gives feedback, reports to the owner. Does not do the work herself."
```

Creating the profile also creates a `stella` command. Check it:

```bash
stella chat
```

You should see: the same kind of chat as step 1, now running as the `stella` profile.

### 3. Create the employees she manages

Stella needs somebody to manage. Create one profile per AI employee, and give each a description. Hermes uses that description to decide who gets which task, so write what the employee is good at, not a job title.

```bash
hermes profile create researcher --description "Reads source material and outside pages, writes findings with sources."
hermes profile create writer --description "Turns findings into finished drafts in the owner's voice."
```

Swap these two for the employees your business needs. Start with two or three. You can add more later.

You should see: all of them listed by `hermes profile list`.

### 4. Give Stella her identity

In Hermes, who an agent is lives in a file called `SOUL.md` inside that agent's home. For a profile named `stella` that is:

```text
~/.hermes/profiles/stella/SOUL.md
```

Hermes creates a starter file there. Replace its contents with the text in "Stella's instructions" below. Keep it about who she is and how she talks. The team list and the goals go in a different file, in the next step.

You should see: after starting a new `stella chat`, she introduces herself as your Chief of Staff when asked who she is.

### 5. Write the team and the goals

This is the step everybody rushes, and it is the one that makes Stella useful.

Make a folder for her to work from and put one file in it:

```bash
mkdir -p ~/stella-desk
```

Create `~/stella-desk/AGENTS.md` with your real team. Hermes loads `AGENTS.md` from the folder you start a chat in.

```text
# The team

Owner: {your name}. Everything goes to the owner for approval before it is sent, posted, booked or paid.

| Employee (profile name) | What they are for | Their one goal | How we know it is on track |
|---|---|---|---|
| researcher | {one sentence} | {one goal, with a deadline or a rhythm} | {what you would look at to check} |
| writer | {one sentence} | {one goal, with a deadline or a rhythm} | {what you would look at to check} |

# How Stella runs this team

- Every task goes on the board, assigned to exactly one employee from the table above.
- A task with no movement since the last brief counts as stalled.
- When a task stalls: read its comments, write one comment that says what is wrong and what to do next, then assign the next task.
- Never assign to a name that is not in the table.
```

A goal that works: "Publish two finished posts a week, drafts ready by Thursday." A goal that does not: "Help with content."

You should see: when you run `stella chat` from inside `~/stella-desk` and ask who is on the team, she answers from this table.

### 6. Turn on the task board

Hermes has a shared task board, called Kanban, that every profile can read and write. It is how Stella assigns work and how she can tell who is slipping.

Create the board and give Stella permission to manage it:

```bash
hermes kanban init
hermes -p stella tools enable kanban
```

Then tell Hermes that Stella owns the team's work. Add this to `~/.hermes/config.yaml`:

```yaml
kanban:
  orchestrator_profile: stella
```

Start the gateway. It is the background process that hands tasks to employees, and nothing on the board moves without it:

```bash
hermes gateway start
```

On macOS you can make it start on its own with `hermes gateway install`.

Start a new `stella chat` after changing her tools. An old conversation keeps its old permissions.

You should see: `hermes kanban list` runs without error, and a new task you ask Stella to create shows up there with one of your employees as the assignee. Tasks are picked up about once a minute.

### 7. Schedule the morning brief

Hermes has a built-in scheduler. Give the job Stella's folder so she reads the team file when she runs. The folder must be written as a full path.

```bash
hermes -p stella cron create "weekdays at 7am" \
  "Morning brief. For each employee in the team table: their goal, what moved on the board since the last brief, and whether they are on track or slipping. Then list every stalled task with why it stalled and the next task you assigned. End with the decisions that need the owner. Keep it under 200 words." \
  --workdir /Users/you/stella-desk \
  --name "Morning brief"
```

Schedules run from a gateway, and each profile has its own. Start Stella's, and check that her scheduler is alive:

```bash
stella gateway start
hermes -p stella cron status
```

`stella gateway install` keeps it running after a restart. If you would rather run one gateway for every profile, the Hermes docs describe that under "Running Many Gateways at Once".

By default the brief is saved on your machine. To get it on your phone, connect Telegram first with `hermes gateway setup`, then create the job with `--deliver telegram` added.

Two things to check on your machine:

- Whether a scheduled run can see the board with the default settings. Run `hermes -p stella tools` to see which toolsets are on for each place Stella runs, and turn `kanban` on wherever the brief runs.
- Whether your times come out in your own timezone. Hermes reads the schedule in the profile's `timezone:` setting. Add `--timezone America/Los_Angeles` (or your zone) to pin it.

You should see: a brief waiting for you the next weekday morning.

## Stella's instructions

Paste this into `~/.hermes/profiles/stella/SOUL.md`. Change the parts in braces to your business.

```text
# Who you are

You are Stella, Chief of Staff for {business name}. You run a team of AI employees on behalf of {owner name}.

Your job is to make sure every employee has one clear goal, to know who is hitting it and who is slipping, and to keep work moving without the owner having to manage anyone.

# How you work

- You do not do the work yourself. You assign it to the right employee and step back.
- Every task you create has one owner, a clear finish line, and everything that employee needs to start.
- When someone stalls, you find out why before you say anything. Then you give feedback that names the problem and the next action, and you assign the next task.
- You judge progress by what is on the board, not by what anyone says they did.
- You never invent a result, a number or a status. If you do not know, you say so.
- Nothing is sent, posted, booked, paid or deleted without the owner's approval. You draft and you report.

# How you talk

- Direct, calm and short. No filler, no cheerleading.
- Lead with what needs a decision.
- When you report on the team, name the employee, the goal, and whether they are on track. One line each.
- If two things conflict, say which one you would pick and why, in one sentence.
```

This persona was written from Stella's job description for this guide. It is not a copy of the instructions she runs on in our own team.

## First test

Run `stella chat` from inside `~/stella-desk` and send her this:

```text
Who is on the team, what is each employee's goal, and who is slipping? Then create one task for each employee that moves them toward their goal.
```

A good answer lists your employees from the team table with their goals, says plainly that nothing is on the board yet, and creates one task per employee with that employee as the assignee. Check with `hermes kanban list`.

If she starts doing the tasks herself, tighten the "You do not do the work yourself" line and consider limiting her tools to the board, the gateway and memory. The Hermes docs recommend exactly that for an orchestrator.

## When it goes wrong

| What you see | Why | Fix |
|---|---|---|
| Empty or broken replies | Model or login is wrong | `hermes model`, then a plain chat before anything else |
| Stella says she cannot create tasks | The kanban toolset is off for her, or the chat is older than the change | `hermes -p stella tools enable kanban`, then a new chat |
| Tasks sit in "ready" forever | The gateway is not running | `hermes gateway start`, check with `hermes gateway status` |
| A task is never picked up | It is assigned to a name that is not a real profile | `hermes profile list`, then `hermes kanban assign <id> <profile>` |
| She does not know your team | The chat was not started in `~/stella-desk` | Start `stella chat` from that folder |
| No morning brief | Stella's gateway is not running, or the job is missing its folder | `hermes -p stella cron status`, `stella gateway start`, and recreate the job with `--workdir` |
| Anything else | | `hermes doctor` |

## If a step breaks

Hermes changes fast, so a command here can drift from what your version expects. You do not need us to fix it. Paste this into Hermes itself, or into Claude or ChatGPT, with the error underneath:

```text
I am following a setup guide for a Chief of Staff agent on Hermes Agent. This step failed. Here is the step, the exact command I ran, and the full error. Check the current Hermes Agent docs at https://hermes-agent.nousresearch.com/docs, tell me what changed, and give me the corrected command.
```

`hermes doctor` is worth running first. It catches most setup problems on its own.

## What it costs to run

- Hermes Agent: free, MIT licence.
- The model: billed by whichever provider you choose in `hermes model`. Every task an employee runs and every morning brief is model usage. Not measured here, so no number is given.
- A cheaper setup the Hermes docs describe: a strong model for Stella and less expensive models for the employees.

## Want it set up for you?

CAMBY Ai sets Stella up and runs her for you: https://mikecamby.com

## Credit

Stella runs on [Hermes Agent](https://github.com/NousResearch/hermes-agent), built by Nous Research and released under the MIT licence. Hermes is not ours. This guide is a walkthrough for using it as a Chief of Staff. Not affiliated with or endorsed by Nous Research or Telegram.

## Sources

All read on 2026-11-21 from the Hermes Agent repository and its documentation.

- https://github.com/NousResearch/hermes-agent : install command, first-run commands, licence.
- https://hermes-agent.nousresearch.com/docs/getting-started/quickstart : install paths, `hermes setup`, the 64,000 token minimum, the "Free, start now" option, `hermes doctor`.
- https://hermes-agent.nousresearch.com/docs/user-guide/profiles : `hermes profile create`, `--description`, one profile per agent, the per-profile command.
- https://hermes-agent.nousresearch.com/docs/user-guide/features/personality : `SOUL.md`, where it lives and what belongs in it.
- https://hermes-agent.nousresearch.com/docs/user-guide/which-file-does-what : `SOUL.md` versus `AGENTS.md`.
- https://hermes-agent.nousresearch.com/docs/user-guide/features/kanban : `hermes kanban init`, enabling the kanban toolset, `kanban.orchestrator_profile`, the gateway dispatcher, how an orchestrator should behave.
- https://hermes-agent.nousresearch.com/docs/user-guide/features/cron : `cron create`, "weekdays at 7am", `--workdir`, `--deliver`, `--name`, `--timezone`.
- https://hermes-agent.nousresearch.com/docs/user-guide/messaging : `hermes gateway setup`, `start`, `install`, `status`.
- https://hermes-agent.nousresearch.com/docs/user-guide/multi-profile-gateways : one gateway per profile, `<profile> gateway start`, `hermes -p <profile> cron status`.

<!-- Before this leaves the folder:
[x] commands come from the Hermes docs dated above; not test-run, by Michael's decision (2026-11-21: readers ask their AI to patch)
[x] the guide says so once at the top and has an "If a step breaks" section
[x] every price and limit has a source and a date (no prices are stated)
[x] no private names, accounts, keys, clients or other bots
[x] no em dashes, brand written CAMBY Ai
[ ] Michael has read it. Nothing is pushed to GitHub without his OK. -->
