# Osysharp.Privacy.Llm

**The AI model calls made for a person, under that person's privacy request.** Add it beside `Osysharp.Llm` and
`Osysharp.Privacy` and the model meter's rows are part of what a person can see, have kept out of use, or be taken
off — and a call stops being anybody's when its time is up.

```osy
use Osysharp.Llm@2;
use Osysharp.Privacy@0;
// Osysharp.Privacy.Llm is not written: it arrives by itself, because the app uses what it completes (`osy lock` says so)
```

Nothing is wired: the holder is found by itself.

## Use case

A phrasebook app writes a phrase in another language for whoever is signed in. Each call is a row on the model
meter: the model, the tokens, what it cost, when — and who it was made for. The meter keeps **no text**: not what was
sent, not what came back.

Mia asks for a copy: two calls, each with its model and its cost. Then she asks to be erased. The rows stay — the app
still knows it made three calls that cost six thousandths of a dollar — but none of them is hers: her id is off each
row and out of what each was charged to, and what was counted against a budget of her own is gone.

```osy
ModelCall.Count(m => m.Requester == mia);     // 0
ModelCall.Count();                             // 3 — still on the meter
SpentOn("app:");                               // 0.006 — still counted
```

## What it holds

| Category | What | Kept |
|---|---|---|
| `llm.calls` | The calls made for a person: when, on which model, what the app said the call was for, tokens and cost. Also what was counted against a budget of their own, and a budget that names them. | **13 months** on their name |

Found by the **account** a request proved — a call is a person's because they were signed in when it was made. A
request that proved only an address reaches none. Held on legitimate interest (accounting for what the AI features
cost). The period is your app's default — `app.ModelCallsPrivacy = new ModelCallsPrivacySetup { KeepMonths = 6 };` —
and the tenant's on the retention page.

## What a request does

| Request | What happens |
|---|---|
| **A copy** (access) | Each call: when, the model, its purpose in the app's words, tokens, cost. |
| **Erasure** | They are taken off the meter (`ForgetModelCallRequester`): every row loses who it was for and the `requester:` line of what it was charged to; their own allowances and a budget naming them are deleted. The rows, and what they cost, stay. |
| **Restriction** | Their rows leave every ordinary read — the usage pages, a scope's total — until it is lifted. Limits keep holding: what was counted against a budget stays counted. |
| **A call still on its way** | The request waits for it to report back. |
| **A legal hold** | A held person's calls stay theirs through the daily purge. |

Every day, for everybody: a call older than the period loses who it was for. The row stays.

## Its tests

`kits/privacy-llm/tests` is that phrasebook: two travellers and Ada, who manages privacy and the model's cost. One
scripted model; none is ever called.

| | |
|---|---|
| Her copy lists the calls made for her, and nobody else's | `her_copy_lists_the_calls_made_for_her_and_nobody_elses` |
| An erasure takes her off the meter and keeps what her calls cost | `an_erasure_takes_her_off_the_meter_and_keeps_what_her_calls_cost` |
| What was counted against her own budget goes with her | `what_was_counted_against_her_own_budget_goes_with_her` |
| A call loses who it was for when its time is up, unless a hold covers the person | `a_call_loses_who_it_was_for_when_its_time_is_up_unless_a_hold_covers_the_person` |
| A restriction takes her calls out of what others read | `a_restriction_takes_her_calls_out_of_what_others_read` |

```
osy test kits/privacy-llm/tests
```

## Source and package

- Package: `Osysharp.Privacy.Llm` — it completes `Osysharp.Privacy` and `Osysharp.Llm`, and arrives by itself.
- Source: [`kits/privacy-llm`](.) — `model/Holder.osy`.
- Builds on [`Osysharp.Llm`](../llm) (`ForgetModelCallRequester`, `ModelCallsInFlightFor`) and
  [`Osysharp.Privacy`](../privacy) (the holder contract, requests, the purge, holds).
- What an agent was shown and answered is not on the meter: it is `Osysharp.Agents`' record, which
  [`Osysharp.Privacy.Agents`](../privacy-agents) holds.

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.Llm;
using Osysharp.Privacy;
using Osysharp.Privacy.Llm;
using Phrasebook;

[Test(Book)]
void her_copy_lists_the_calls_made_for_her_and_nobody_elses() {
  Asked();
  runas (Ada) {
    var mia = new DataSubject { Account = AccountOf("mia@traveller.example") };
    var holder = new ModelCallsPersonalData();
    Assert.Equal(2, holder.Count(mia, CategoryOf("llm.calls")));
    var section = holder.Export(mia, CategoryOf("llm.calls"));
    Assert.Equal(2, section.Items.Count);
    Assert.Contains("\"Model\":\"Small\"", section.Json);
    Assert.Contains("\"Cost\":0.002", section.Json);
    // The meter keeps no text of a call, so her copy holds none.
    Assert.DoesNotContain("station", section.Json);
    // A call is a person's because they were signed in: an address alone reaches none.
    Assert.Equal(0, holder.Count(new DataSubject { Emails = ["mia@traveller.example"] }, CategoryOf("llm.calls")));
  }
}
```

From [`tests/calls.test.osy`](tests/calls.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.Llm;
using Osysharp.Privacy;
using Osysharp.Privacy.Llm;
using Phrasebook;

[Test(Book)]
void an_erasure_takes_her_off_the_meter_and_keeps_what_her_calls_cost() {
  Asked();
  var mia = AccountOf("mia@traveller.example");
  var noor = AccountOf("noor@traveller.example");
  runas (Ada) {
    Assert.Equal(2, ModelCall.Count(m => m.Requester == mia));
    var r = FileRequest(RequestKind.Erasure, mia, "", IdentityCheck.InPerson, "She showed her passport at the counter.");
    Assert.Equal(2, PlanLine.Single(l => l.Request == r && l.Category == "llm.calls").Items);
    ApproveRequest(r);
    var done = PrivacyRequest.Single(x => x.Id == r.Id);
    Assert.Equal("", done.Problem);
    Assert.Equal(RequestState.Done, done.State);
    Assert.Contains("What we erased: The AI model calls made for you", done.Response);
  }
  runas (Ada) {
    // No row is hers, and none names her in what it was charged to.
    Assert.Equal(0, ModelCall.Count(m => m.Requester == mia));
    Assert.Equal(0, ModelCall.Count(m => m.Scopes.Contains(mia.ToString())));
    // The three calls are still on the meter, and still cost what they cost.
    Assert.Equal(3, ModelCall.Count());
    Assert.Equal(0.006m, SpentOn("app:"));
    // Noor's is still hers.
    Assert.Equal(1, ModelCall.Count(m => m.Requester == noor));
    Assert.Equal(1, new ModelCallsPersonalData().Count(new DataSubject { Account = noor }, CategoryOf("llm.calls")));
    Assert.Equal(0, new ModelCallsPersonalData().Count(new DataSubject { Account = mia }, CategoryOf("llm.calls")));
  }
}
```

From [`tests/calls.test.osy`](tests/calls.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.Llm;
using Osysharp.Privacy;
using Osysharp.Privacy.Llm;
using Phrasebook;

[Test(Book)]
void what_was_counted_against_her_own_budget_goes_with_her() {
  runas (Ada) { SetBudget("requester", "", LlmBudgetWindow.Day, 1m); UnitOfWork.Commit(); }
  Asked();
  var mia = AccountOf("mia@traveller.example");
  runas (Ada) {
    Assert.Equal(0.004m, SpentIn("requester", mia.ToString(), LlmBudgetWindow.Day));
    // Two calls and one allowance are about her.
    Assert.Equal(3, new ModelCallsPersonalData().Count(new DataSubject { Account = mia }, CategoryOf("llm.calls")));
    var r = FileRequest(RequestKind.Erasure, mia, "", IdentityCheck.InPerson, "She showed her passport at the counter.");
    ApproveRequest(r);
    Assert.Equal(RequestState.Done, PrivacyRequest.Single(x => x.Id == r.Id).State);
  }
  runas (Ada) {
    Assert.Equal(0, LlmSpend.Count(s => s.Subject == mia.ToString()));
    // The limit for every person still stands, and Noor's allowance with it.
    Assert.Equal(1, LlmBudget.Count());
    Assert.Equal(0.002m, SpentIn("requester", AccountOf("noor@traveller.example").ToString(), LlmBudgetWindow.Day));
  }
}
```

From [`tests/calls.test.osy`](tests/calls.test.osy).

## Learn more

- [The Osysharp.Privacy.Llm reference](https://osysharp.com/reference/agent/kit-privacy-llm/)
- [Osysharp.Privacy.Llm in the package catalogue](https://osysharp.com/packages/#osysharp-privacy-llm)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
