# Build the Ladder in Claude Code: A Step-by-Step Guide

This guide turns the six-step AI skills ladder and the "folder is the agent" idea into a concrete setup in Claude Code.

**The core idea:** you don't program a separate agent for every job. You build a folder that holds your context, instructions, tools, and data. Claude Code opens that folder and *becomes* the agent for whatever you ask. Where you want more than one worker, you define subagents or run several sessions.

**Which Claude Code feature does what**

| Need | Feature | Where it lives |
|---|---|---|
| Facts Claude should always know | `CLAUDE.md` | project root |
| Repeatable procedures / how-tos | Skills | `.claude/skills/<name>/SKILL.md` |
| Delegated, isolated workers ("employees") | Subagents | `.claude/agents/<name>.md` |
| Reusable prompts you trigger by hand | Slash commands | `.claude/commands/<name>.md` |
| Deterministic rules and approval gates | Hooks + permissions | `.claude/settings.json` |
| Connections to outside tools and data | MCP servers | project or user settings |
| Running on a schedule | Scheduled tasks, `/loop`, `/schedule` | built-in / desktop / web |

Features change quickly. Check the official docs (https://code.claude.com/docs) when a command or file format doesn't behave as described here.

---

## Step 0: Install and open a project folder (30 minutes)

1. Install Claude Code by following the official quickstart at https://code.claude.com/docs.
2. Create a folder for your "business brain" and open a terminal in it:
   ```bash
   mkdir my-ai-team && cd my-ai-team
   git init
   claude
   ```
3. Put the folder under Git (and on GitHub if you like). Everything you build is plain text, so version history comes for free. **Keep secrets and client data out of any public repo.**

---

## Step 1: Learn the words (a few hours, then stop)

Learn these terms only well enough to recognize them:

- **Model:** the AI itself.
- **Harness:** the tool wrapped around a model (Claude Code is one).
- **Automation:** a fixed process that runs without you.
- **Agent:** a model that takes its own steps toward a goal.
- **Skill, subagent, hook, MCP:** the Claude Code building blocks in the table above.

Quick way to do it: run `claude` and ask it to explain each term, then explain how it works in the current folder. Then move on.

---

## Step 2: Automation with no AI (the foundation)

Pick one boring, daily, zero-judgment task, such as moving data between files, sending the same email, or updating a sheet.

1. Ask Claude Code to build it as a plain script:
   > "Write a Python script in `automations/` that does X. No AI calls. Make it safe to run repeatedly, log what it did, and fail loudly on errors."
2. Run it by hand until it works every time.
3. Schedule it. Options, from simplest up:
   - Your operating system's scheduler (cron on Mac/Linux, Task Scheduler on Windows) running the script directly.
   - Claude Code's scheduled-task features (desktop scheduled tasks, `/loop`, `/schedule`) if you want Claude to run it.
4. Done when you can forget about it for a week and it keeps working.

---

## Step 3: Put a brain in it

Keep the same automation and add one judgment step, such as reading, classifying, or drafting.

1. Create a skill for the judgment part. Make the folder `.claude/skills/triage-inbox/` and add `SKILL.md`:
   ```markdown
   ---
   name: triage-inbox
   description: Sorts incoming items into categories and drafts replies. Use when processing the daily inbox export.
   ---

   # Triage inbox

   1. Read the file in `data/inbox/` for today.
   2. Classify each item as: urgent, routine, or ignore.
   3. For routine items, draft a reply in my voice (see `context/voice.md`).
   4. Write results to `output/triage-YYYY-MM-DD.md`. Never send anything.
   ```
2. Have your scheduled automation call Claude Code with this skill as its job, so the old script handles the plumbing and the AI handles the judgment.
3. Start in "draft only" mode. You review the output for a couple of weeks before trusting it further.

---

## Step 4: Agents with a hand on the wheel

Now let Claude work toward a goal across multiple steps, and gate anything risky.

1. **Use plan mode for big tasks.** Press Shift+Tab to cycle modes, and let Claude propose a plan before it edits anything.
2. **Define subagents for specialist jobs.** Add `.claude/agents/researcher.md`:
   ```markdown
   ---
   name: researcher
   description: Gathers and summarizes information for a client brief. Use when a task needs background research.
   tools: Read, Grep, Glob, WebSearch
   ---

   You are a careful research assistant. Read the client folder first.
   Return a one-page summary with sources. Do not modify files.
   ```
   Limiting `tools` is how you keep a worker read-only.
3. **Set up approval gates.**
   - Use Claude Code's permission settings so risky actions (sending email, deleting files, running unknown shell commands) require your approval.
   - Use hooks in `.claude/settings.json` for rules that must *always* hold, for example blocking edits to a protected folder. Hooks run as real code, so they are more reliable than asking nicely in a prompt.
4. **Update `CLAUDE.md` after every mistake.** If the agent did something wrong, add the rule that prevents it.

---

## Step 5: Build tools and apps by talking to it

1. Describe the tool you want, in plain language, in plan mode first:
   > "I want a small web app that shows my daily triage results. Propose the front end, back end, and database before writing code."
2. Learn the three layers as you go:
   - **Front end:** what the user sees.
   - **Back end:** the logic and the server.
   - **Database:** where the data is stored.
3. Ask Claude to explain what it built and how to test it. Run it locally before deploying anywhere.
4. **Know when to slow down.** Anything touching logins, personal data, or payments needs real review of what the code does, not just "it seems to work." Ask Claude to review its own work for security problems, then check the important parts yourself.

---

## Step 6: Run a team ("AI employees" in folders you own)

Put it all together in one repository:

```
my-ai-team/
├── CLAUDE.md                  # always-on facts: who you are, how the folder is organized
├── context/                   # your voice, niche knowledge, client info, opinions
│   ├── voice.md
│   └── clients/
├── data/                      # inputs the agents read
├── output/                    # everything the agents produce
├── automations/               # plain scripts from Step 2
└── .claude/
    ├── settings.json          # permissions and hooks
    ├── skills/                # repeatable procedures (Step 3)
    ├── agents/                # specialist workers (Step 4)
    └── commands/              # shortcuts you trigger by hand
```

Treat each "employee" as three things in the folder:

- a **subagent file** (its role and allowed tools),
- one or more **skills** (what it knows how to do),
- a **memory/context folder** (what it knows about your business).

Then:

1. **Start with one narrow job** and get it reliable before adding another. Reliability drops as you chain more steps.
2. **Use a short `CLAUDE.md`** (aim for well under 200 lines) and push long procedures into skills so they load only when needed.
3. **Schedule the recurring jobs** and review the output regularly at first.
4. **Run more than one session** when you want parallel work. Separate Git worktrees keep sessions from editing the same files.
5. **Keep the structure portable.** Because it is just folders and markdown, other AI tools can read the same material. Some read an `AGENTS.md` file the way Claude Code reads `CLAUDE.md`.

---

## Suggested 12-week pacing

| Weeks | Focus |
|---|---|
| 1 | Install, vocabulary (Steps 0-1) |
| 2-3 | One scheduled, no-AI automation (Step 2) |
| 4-5 | Add the AI judgment step as a skill (Step 3) |
| 6-8 | Subagents, plan mode, permissions, hooks (Step 4) |
| 9-10 | Build one small tool or app (Step 5) |
| 11-12 | Assemble the team folder and schedule it (Step 6) |

Stretch it to four or five months if you need to. The pace matters less than finishing each rung with a real task.

---

## Common mistakes

- **Stuffing everything into `CLAUDE.md`.** Facts go there. Procedures belong in skills.
- **Giving agents broad permissions on day one.** Start read-only, and widen access as you gain trust.
- **Skipping Step 2.** If the plain automation isn't solid, the AI version will be flaky too.
- **Building features instead of structure.** The lasting value is your organized context, data, and procedures. Keep investing in `context/` and your skills.
- **Putting secrets or client data in a public repo.** Use a private repo, and keep credentials in environment variables, never in files Claude reads and commits.
