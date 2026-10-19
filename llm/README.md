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
