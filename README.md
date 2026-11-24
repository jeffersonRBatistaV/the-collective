# The Collective

![Alegorías de Mármol en Luz Dorada](assets/alegorias-marmol-luz-dorada.png)

A deliberation system with four AI agents for hard decisions. It works as a Claude Code project: open this folder and use `/collective` (personal decisions) or `/collective-group` (decisions that affect a group). For lighter decisions, both skills accept `--lite`.

Every session follows the same process:

1. The **Regent** builds a dossier with the decision, the options, the facts and the verbatim quotes.
2. **Round 1:** the four members propose separately and in parallel, without seeing each other's answers.
3. **Round 2:** each member critiques the others' proposals (skipped with `--lite`).
4. The Regent saves the full verdict to `verdict.md` and shows only `answer.md`: a direct answer, the cost of each option or faction, and one first step. The complete debate stays in `round1.md` and `round2.md`.

## Structure

```
.claude/
  agents/                      Full personalities (one per file)
    husserl.md                 The Dawn: facts vs. assumptions. Searches with WebSearch, opens pages with WebFetch.
    derrida.md                 The Dusk: the framing of the question and the discarded option.
    lacan.md                   The Night: the desire behind the decision. Reads the history.
    berkeley.md                The Day: how others see it. Finds testimonies with WebSearch and WebFetch.
  skills/
    collective/SKILL.md        Individual orchestrator. Includes "The members of the
                               Collective", the summary of lenses and biases the Regent uses.
    collective-group/SKILL.md  Group orchestrator: leader/assembly modes, per-group archive,
                               pseudonymization, shareable verdict.
  collective/                  Session archive (local, outside git)
    sessions/                  /collective sessions
    groups/<group>/            /collective-group sessions and the factions' legend.md
    groups/_rehearsals/        Rehearsals, kept separate from real history
```

`collective-group` does not repeat the members: it points to the members section in `collective/SKILL.md` and to each agent's "Group mode" section. All six files are required. If any of the four agents is missing, the skill stops and names the missing file instead of improvising the member.

## Installation

1. Clone this repo inside your project: `git clone https://github.com/jeffersonRBatistaV/the-collective.git`, or copy the whole folder.
2. Check that the six files are there: the four in `.claude/agents/` and the two `SKILL.md` files in `.claude/skills/`. Without them the session will not start.
3. Open Claude Code at the project root and try `/collective` or `/collective-group`.

## Before you use it

- **Cost:** a full session launches 8 subagents (4 per round) plus one synthesis call, with up to 300 words per agent per round. `/collective --lite` and `/collective-group --lite` run only round 1: 4 subagents, no debate, and the answer says so.
- **Language:** the agent and skill prompts are in English; replies, dossiers and verdicts come out in the language of the person consulting.
- **Outside Claude Code:** the logic is portable markdown (see below), but here the skills depend on Claude Code tools: `Agent`, `AskUserQuestion`, `Read`, `Glob`, `Grep`, `WebSearch` and `WebFetch`. Another platform needs equivalents for them.

## Privacy

`.claude/collective/` is in `.gitignore`. Dossiers, transcripts with quotes, Lacan's readings of people and factions, full verdicts and internal documents are stored there. Nothing in it is pushed to a remote.

Group sessions have three outputs: `verdict.md` is the full internal verdict, `verdict_shareable.md` omits the readings about people so it can circulate, and `answer.md` is the short answer shown to the representative, built with the same privacy rule as `verdict_shareable.md`.

## Porting to another platform

The logic is in markdown, so it can be moved to an app using the Claude API:

| In Claude Code | In the platform |
|---|---|
| Body of each `agents/*.md` (without frontmatter) | *System prompt* for that member's call |
| Phases of `collective-group/SKILL.md` | Orchestrator in code: 4 parallel calls (round 1), 4 parallel calls (round 2, full mode only), 1 synthesis call |
| Verdict template and the Regent's rules | *System prompt* for the synthesis call |
| Agents' `Read`/`Glob`/`Grep` | Dossier and rounds injected into the message; library and history via RAG |
| `WebSearch` + `WebFetch` (Husserl, Berkeley) | Web search and page-reading tools from the API |
| Folders in `.claude/collective/` | Database tables with role-based access control |

Before loading real data, define data retention with the provider, encryption, and compliance with the applicable data protection law (in the Dominican Republic, Law 172-13).
