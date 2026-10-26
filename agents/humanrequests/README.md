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

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.Llm;
using Osysharp.Workflow;
using Osysharp.WorkQueues;
using Osysharp.Agents;
using Osysharp.Agents.Testing;
using Osysharp.Agents.HumanRequests;
using Osysharp.Privacy;
using Osysharp.Privacy.Agents.HumanRequests;
using LostProperty;

[Test(Desk)]
void who_handed_an_item_to_an_agent_is_taken_off_it_and_nobody_else() {
  runas (Lena) {
    HandsOver("A black umbrella", true);
    HandsOver("A green scarf", false);
  }
  GoOn();
  runas (Omar) { HandsOver("A grey rucksack", false); }
  GoOn();
  var lena = new DataSubject { Account = AccountOf("lena@ferryline.example") };
  var holder = new AgentHandoverPersonalData();
  runas (Pia) {
    // Her copy lists both items she handed over.
    var copy = holder.Export(lena, CategoryOf("agents.handovers")).Json;
    Assert.Contains("A black umbrella", copy);
    Assert.Contains("A green scarf", copy);
    Assert.DoesNotContain("A grey rucksack", copy);
    Assert.Equal(2, holder.Count(lena, CategoryOf("agents.handovers")));
    // The matcher still holds the umbrella: her erasure would wait.
    Assert.Equal("an AI agent still holds an item you handed over", holder.BlockedBy(lena, CategoryOf("agents.handovers")));
  }
  runas (Lena) {
    AnswerAgentQuestion(QuestionsForMe().Single(), "Yes, the 07:40.");
    UnitOfWork.Commit();
  }
  GoOn();
  runas (Pia) {
    Assert.Equal("", holder.BlockedBy(lena, CategoryOf("agents.handovers")));
    var r = FileRequest(RequestKind.Erasure, AccountOf("lena@ferryline.example"), "", IdentityCheck.InPerson, "She asked on her last day.");
    ApproveRequest(r);
    var done = PrivacyRequest.Single(x => x.Id == r.Id);
    Assert.Equal("", done.Problem);
    Assert.Equal(RequestState.Done, done.State);
    // Taken off both of hers; that each was handed over, and how it came back, stays.
    Assert.Equal(0, holder.Count(lena, CategoryOf("agents.handovers")));
    Assert.Equal(2, AgentHandover.Count(h => h.By == null));
    Assert.Equal(3, AgentHandover.Count(h => h.State == AgentHandoverState.Back));
    // Omar's is still his.
    Assert.Equal(AccountOf("omar@ferryline.example"), AgentHandover.Single(h => h.By != null).By!.Id);
  }
}
```

From [`tests/handovers.test.osy`](tests/handovers.test.osy).

## Learn more

- [The Osysharp.Privacy.Agents.HumanRequests reference](https://osysharp.com/reference/agent/kit-privacy-agents-human-requests/)
- [Osysharp.Privacy.Agents.HumanRequests in the package catalogue](https://osysharp.com/packages/#osysharp-privacy-agents-humanrequests)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
