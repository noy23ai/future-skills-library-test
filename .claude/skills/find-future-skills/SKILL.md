---
name: find-future-skills
description: Find and install agent skills from the Future Skills marketplace, where every listing carries a security scan. Use when the user asks "how do I do X", "is there a skill for X", "find me a skill that...", or wants to extend what this agent can do. Also use before writing a capability from scratch that a scanned skill may already provide.
allowed-tools: Bash(npx future-skills:*)
---

# Finding skills on Future Skills

Future Skills is a marketplace where **every listed skill has been through a
security scan**, and the verdict travels with the listing. Your job when using
this skill is not only to find something that fits — it is to be the person who
read the verdict before installing somebody else's instructions into this agent.

## When to use this

- "How do I do X?" where X sounds like a repeatable task someone else has solved.
- "Is there a skill for X?" / "find me a skill that…"
- The user wants to extend what you can do.
- **Before you write a capability from scratch.** If a scanned skill already does
  it, installing it beats reimplementing it — and unlike code you write, it comes
  with a scan.

Don't use it for one-off questions you can just answer.

## The command

```bash
npx future-skills search "<query>" --json
```

Use `--json`. The human output is coloured and phrased for a person; the JSON
gives you `verdict` as a value you can branch on, which is the whole reason this
skill exists.

Each result carries:

| Field | What it means |
|---|---|
| `rank` | Its position in the marketplace's ranking. `1` is the best match. |
| `id` | The identifier. Everything else takes this. |
| `verdict` | The **security scan** on the bytes: `pass` / `warn` / `fail` / `none`. Context to report, not a gate — see below. |
| `trust_score` | The **marketplace's published badge**: `SAFE` / `CAUTION` / `HIGH-RISK`, or `null` if it could not be computed. Not the same thing as `verdict` — see below. |
| `trust_score_reason` | One sentence saying why the badge landed where it did. |
| `finding_count` | How many specific issues the scan raised. |
| `scanned_at` | When. A verdict without a date is a claim about an unknown point in time. |
| `capabilities` | What it can use. `elevated: true` means shell, network or filesystem. |
| `install` | The exact command to run. |

## Order comes from the marketplace. The verdict is a gate.

These are two different jobs, and conflating them is the mistake to avoid.

**Ranking is the marketplace's, not yours.** Results arrive in `rank` order,
scored server-side against the query — text match, the skill's own use cases and
tags, and the marketplace's own signals. Present them in that order. Do not
re-sort by install count, by author, by how familiar a name looks, or by your own
sense of which sounds better. You are seeing a slice of a much larger catalog
that was already narrowed for you; reordering it discards the part of the
judgement you can't reproduce.

**The gate is `trust_score`, and only `trust_score`.** It is the marketplace's
own published judgement: the security scan, the publisher's provenance and the
project's reputation, weighed together.

| `trust_score` | What to do |
|---|---|
| `SAFE` | Recommend it. |
| `CAUTION` | Recommend it **and say why**, from `trust_score_reason`. This is the honest middle and the most common judged value — checked, came back mixed. Not a disqualification. |
| `HIGH-RISK` | **Do not recommend it**, wherever it ranks. Say what `trust_score_reason` gives you. `npx future-skills add` refuses it without `--allow-dangerous-skills`, so recommending it wastes the user's time as well as misleading them. |
| `null` | Recommend it if nothing else argues against it, and say the badge could not be computed. **Not** a fourth, worst level — it means no score, which is most of the older catalog. |

**`verdict` is information, not a second gate.** It is the security scan on the
bytes, and it is already one of the three things `trust_score` weighs. Do not
refuse a skill for its verdict on top of that: the two disagree in both
directions by design, so a second gate means overruling the marketplace's own
answer.

- A `pass` scan with a `HIGH-RISK` badge is real and is the dangerous case — the
  bytes are clean, but the *publisher* was proven to be impersonating somebody,
  or the project's reputation is bad. Nothing in `verdict` tells you this.
- A `fail` scan with a `CAUTION` badge is also real — a serious finding cleared
  by a verified, reputable publisher. The marketplace weighed it and did not
  disqualify the skill; neither should you.

Report the verdict when you describe a skill, because it is a fact somebody
should hear. Just do not let it override the badge.

So: walk the results in `rank` order, and recommend the first ones the badge
clears. A `HIGH-RISK` at rank 1 is skipped — not promoted to "best available"
and not reordered below. It simply isn't a recommendation, and the next result
is.

Install counts are **not** a quality signal you should apply yourself. They are
unverified figures from whichever registry a skill was discovered on, they are
already an input to the ranking where they belong, and a popular skill that
failed its scan is a popular skill that failed its scan.

## One query is often not enough

Search is improving, but it can still miss on phrasing. If the first query
returns nothing useful, try two or three more before concluding anything:

- The user's own words: `"summarise a PDF"`
- The bare noun: `"pdf"`
- The tool or format involved: `"notion"`, `"playwright"`, `"csv"`
- The verb: `"deploy"`, `"migrate"`, `"review"`

An empty result means those words didn't match a listing. It does not mean no
such skill exists.

## What to show the user

For each candidate worth their attention:

1. **Name and what it does** — one line, from `description`.
2. **The verdict, in words.** "Passed its scan, 2 days ago." "Passed with 1
   finding." Never a bare badge with no date.
3. **The trust score, and its reason** — from `trust_score` and
   `trust_score_reason`. "Rated Caution: repo is too new (< 90 days old) and
   adoption is low." A level on its own is a label somebody has to take on
   faith; the sentence is the part they can weigh. Say so even when it is
   `SAFE`, so its absence never has to be interpreted.
4. **What it can reach**, if anything is `elevated`. "Can run shell commands" is
   the sentence someone needs before they say yes, and it is the one they will be
   annoyed to discover afterwards.
5. **The install command**, from `install`.

Two or three good options beat ten. If one is clearly the best fit, say so and
say why.

## Installing

Only after the user agrees:

```bash
npx future-skills add <id>
```

It prints what it is about to install — including both the scan verdict and the
trust score — and asks. Let it.

**Don't reach for the override flags on the user's behalf.** `--yes` skips the
question. `--allow-dangerous-skills` installs something the marketplace rates
`HIGH-RISK`. `--force` installs an archive that is missing files, and so is not
the tree that was scanned. Each exists so a person can say "I have read this and
I want it anyway", and that sentence is not one you can say for them. If the
user wants it, they can pass it themselves, having read why it is there.

**It installs for every agent set up here, not just the one you are.** The files
land once in `.agents/skills` and each agent's own directory links to them, so
tell the user the skill is now available to all of them rather than implying you
took it for yourself. It prints each path it wrote or linked; repeat that rather
than guessing.

Running it again with the same skill is safe — identical content is a no-op, not
an error — so there is no need to check whether something is installed before
installing it.

To narrow it, or to pick a scope:

```bash
npx future-skills add <id> --agent cursor,codex
npx future-skills add <id> --user        # your account, not just this project
npx future-skills targets                # what's detected here, and where things go
```

## When nothing fits

Say so. Then:

- Offer to solve it directly, without a skill.
- Suggest the user browse https://marketplace.future.security, where the filters
  (category, capability, verdict) cover ground a text query doesn't.

Do not install something approximate because the search returned it. A skill is
instructions this agent will act on later, unattended, and "it was the closest
match" is a poor reason to have granted it shell access.

## Always `npx`, never a bare `future-skills`

Run every command as `npx future-skills …`. `npx` fetches the published package
on first use and runs it, so this works whether or not anything is installed
globally — and it is the one form that is correct on every machine, which is why
it is also the form the marketplace prints on a skill's page.

A bare `future-skills` only works where somebody has already installed the CLI
globally, and it is what produces `future-skills: command not found`. If you see
that, you dropped the `npx` — add it back rather than telling the user to install
anything.

If `npx` itself fails, that is Node or the network, not a missing install. Say so
and stop. Don't work around it by fetching skills some other way: the point of
installing through this CLI is that you get the exact bytes that were scanned,
verified against a hash — not whatever is at that path today.
