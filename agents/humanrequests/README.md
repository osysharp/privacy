# Osysharp.Privacy.Agents.HumanRequests

Who handed an inbox item to an AI agent — under their privacy request.

```osy
use Osysharp.Privacy@0;
use Osysharp.Agents.HumanRequests@1;     // …and this bridge is there: nothing to wire.

// A person hands an item of their inbox to an agent. The handover says who.
await AgentDesk.Hand(claim, Agents.Of<Matcher>(), new MatchClaim(), claim.Described);
```

## Use case

A ferry line's lost-property desk. Clerks take passengers' claims from the inbox and hand some of them to an agent
that matches a claim against what was found aboard; the agent hands each back with what it found, or a clerk takes it
back. A clerk leaves and asks to be erased.

- **Their copy** lists every item they handed to an agent: which agent, when, and how it came back.
- **Erased, the handover stays and the person goes.** It no longer says who handed the item over; that it was handed
  over, to which agent, the agent's work on it and how it came back stay — the inbox's own record.
- **An item an agent still holds waits for the erasure**: it comes back to whoever handed it over, so it is let come
  back, or taken back, first.
- **Nobody takes a person off by hand** — only a privacy request's own run does (`ErasesAgentRecords`, nobody by
  default), through `ForgetWhoHandedWorkToAgents` in `Osysharp.Agents.HumanRequests`.

## What it adds

| | |
|---|---|
| The holder `AgentHandoverPersonalData`, category `agents.handovers` | discovered by `Osysharp.Privacy`; nothing to wire |

Tests: `tests/handovers.test.osy`.

## Source and package

- Package: `Osysharp.Privacy.Agents.HumanRequests` — `use Osysharp.Privacy.Agents.HumanRequests@1;` (an app that uses
  both kits gets it without saying so)
- Source: [`kits/privacy-agents-human-requests`](.) — `model/Holder.osy`
- Uses [`Osysharp.Privacy`](../privacy) and [`Osysharp.Agents.HumanRequests`](../agents-human-requests).
- The people on the rest of the agents' records — standing rules, changes to an agent's place, questions and
  proposals — are [`Osysharp.Privacy.Agents`](../privacy-agents)' `agents.oversight`.
