# Osysharp.Privacy.Assignments

**The people in an assignment, under their privacy requests — with the assignment left standing.** Add it beside
`Osysharp.Assignments` and `Osysharp.Privacy`, and the people *on* an assignment and the person an assignment is
*about* can each see what is held, have it kept out of use, or have it erased. An erasure leaves words where the
person was, never a hole: the work your organisation did stays on its books.

```osy
use Osysharp.Assignments@2;
use Osysharp.Privacy@0;
// Osysharp.Privacy.Assignments is not written: it arrives by itself, because the app uses what it completes (`osy lock` says so)
```

Nothing is wired: the holder is found by itself.

## Use case

A housing co-operative's repair service. A tenant, Ines Falk, reports a leaking tap; a coordinator opens the repair
and a caretaker, Omar, does it — two visits, a new washer, a note that she was not home at first.

**The tenant asks to be erased.** The repair stays, completed, with both criteria met and every line of its
activity — as work done for somebody who is no longer named:

| before | after |
|---|---|
| Ines Falk's kitchen tap leaks | a former tenant's kitchen tap leaks |
| A new washer; Ines saw it run dry. | A new washer; a former tenant saw it run dry. |
| Fixed for Ines Falk on the second visit. | Fixed for a former tenant on the second visit. |

**The caretaker leaves, and asks to be erased.** He is taken off the repairs he did: his seat goes, the repair he
owned falls to whoever asked for it, and what he did reads as done by *Somebody no longer here* — not by the app,
and not by nobody. The repair is still about Ines: she did not ask.

## Two different people

| | Found by | What an erasure does |
|---|---|---|
| **People ON an assignment** — who asked, who owns it, assignees, contributors, observers, whoever wrote a line or settled a criterion | their **account** | `assignments.people`: they are taken off it. Their seat goes; who asked, who owns, who closed, who a piece of work waits on become nobody; each line they wrote and each criterion they settled is marked as a former person's. Their name and address are replaced in the assignment's sentences. The assignment, its criteria and where each stands, its work, every line and how it ended stay. |
| **The person an assignment is ABOUT** — the tenant, the customer, the patient | what your app **marked** it with: an account, an address or a number | `assignments.about`: the words that say who it was about are replaced by your stand-in ("a former tenant") in every sentence — title, goal, criteria and what met them, attached work and its result, activity, outcome. When nothing says what those words are, every sentence is erased whole. Either way, what happened stays: when it was asked for and by whom, each criterion and whether it was met, each line's kind and time, the cost, how it closed. |

## Saying whose data an assignment is

```osy
public Assignment OpenRepair(Fault fault, Person caretaker) {
  var repair = OpenAssignment(new NewAssignment { Title = fault.TenantName + "'s " + fault.What, … });
  MarkAssignmentAbout(repair, new DataSubject { Emails = [fault.TenantEmail] });   // once, where it is opened
  return repair;
}
```

Everything under a marked assignment is about the same person without a mark of its own: the assignments opened
under it, and the row each piece of attached work stands for — an agent's receipt, a request, a record of yours. So
with the other holders in the app, one request reaches the case, the agent's work on it and the items in its
repository.

## Saying which words name them

A request proves an address or an account — not a name. Your app's own records know the name, so it says which words
in an assignment's text identify the person it is about:

```osy
public class HousingPrivacy : AssignmentPrivacyRules {
  public override List<string> PersonalWords(Assignment a) {
    var fault = Fault.FirstOrDefault(f => f.Repair == (a.Root ?? a));
    return fault == null ? [] : [fault!.TenantName, fault!.TenantName.Split(" ")[0]];
  }
}

app.AssignmentsPrivacy = new AssignmentsPrivacySetup {
  StandIn = "a former tenant", FormerColleague = "a former caretaker", Rules = new HousingPrivacy(),
};
```

Each of those words, the name and address of a marked account, and every address and number the request proved are
replaced, in any capitals, longest first. **When the rule names nothing and no account is known, every sentence is
erased whole** — the safe default: an erasure never leaves a name it could not find. `AlwaysBlank = true` does that
for every assignment.

## What it holds, and for how long

| Category | Kept |
|---|---|
| `assignments.about` | **24 months** after the assignment closed, then its words are erased as above — for everybody, not only for people who asked |
| `assignments.people` | until they ask (set `KeepPeopleMonths`, or the tenant's own period, to take people off closed assignments after a time) |

Both are held on legitimate interest. The periods are your app's defaults and the tenant's on the retention page.

## What else a request does

| | |
|---|---|
| **A copy** | *About them*: every sentence of each assignment — what was asked for, each criterion and what met it, each piece of work and its result, every line of activity, how it ended. *On it*: each assignment they were on, what they were on it, and what they did. |
| **Still open** | An assignment about them is erased once it is closed: what is still being done for them needs to know who they are. The request waits ("an assignment about you is still open") and runs when it closes. |
| **Restriction** | An assignment about them leaves every ordinary read whole. Somebody who worked on assignments keeps them in use for everybody else: what leaves is each line they wrote. |
| **A legal hold** | A held person's assignments are neither erased nor purged while the hold stands. |

## Who erases

Nobody, by hand. `Osysharp.Assignments`' erasure functions ask the app's `ErasesAssignments` policy, which is
nobody's until your app says otherwise — and it need not: a privacy request's own run erases as the app itself. In
the test app nobody is named, and the coordinator who oversees every repair is refused when she tries.

## Its tests

`kits/privacy-assignments/tests` is that co-operative: a coordinator, two caretakers, tenants with no account, and
Ada at the privacy desk — who is on no repair and reads none of them, before or after.

| | |
|---|---|
| Her copy holds the assignment about her, and nobody else's | `her_copy_holds_the_assignment_about_her_and_nobody_elses` |
| An erasure leaves the repair as work done for a former tenant | `an_erasure_leaves_the_repair_as_work_done_for_a_former_tenant` |
| Work opened under a repair about her is about her too | `work_opened_under_a_repair_about_her_is_about_her_too` |
| With nothing to say who they were, every sentence goes and what happened stays | `with_nothing_to_say_who_they_were_every_sentence_goes_and_what_happened_stays` |
| An erasure waits while a repair about him is open, and runs when it closes | `an_erasure_waits_while_a_repair_about_him_is_open_and_runs_when_it_closes` |
| A caretaker who leaves is taken off, and what he did reads as done by somebody no longer here | `a_caretaker_who_leaves_is_taken_off_the_repairs_he_did_and_they_read_as_done_by_somebody_no_longer_here` |
| The words go two years after it closed, unless a hold covers the person | `a_repair_loses_the_words_that_say_whose_it_was_two_years_after_it_closed_unless_a_hold_covers_them` |
| A restriction takes it out of use for everybody | `a_restriction_takes_the_repair_about_her_out_of_use_for_everybody` |
| Nobody erases an assignment by hand; a request does | `nobody_erases_an_assignment_by_hand_and_a_privacy_request_does` |
| What others read is the same before and after | `what_others_read_is_the_same_before_and_after_somebody_is_erased` |
| An address alone never reaches somebody's seat | `a_tenant_with_only_an_address_is_never_taken_for_somebody_on_an_assignment` |

```
osy test kits/privacy-assignments/tests
```

## Source and package

- Package: `Osysharp.Privacy.Assignments` — it completes `Osysharp.Privacy` and `Osysharp.Assignments`, and arrives by itself.
- Source: [`kits/privacy-assignments`](.) — `model/Holder.osy` (the holder, `MarkAssignmentAbout`, `AssignmentsAbout`,
  `AssignmentPrivacyRules`, the copy).
- Builds on [`Osysharp.Assignments`](../assignments) (`TakePersonOffAssignment`, `AnonymiseAssignment`,
  `AssignmentsOf`) and [`Osysharp.Privacy`](../privacy) (the holder contract, `MarkAbout`, requests, the purge, holds).
- The items in an assignment's repository are held by [`Osysharp.Privacy.WorkRepositories`](../privacy-work-repositories);
  an agent's work on it by [`Osysharp.Privacy.Agents`](../privacy-agents).

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.Assignments;
using Osysharp.Privacy;
using Osysharp.Privacy.Assignments;
using Housing;

[Test(Linden)]
void a_restriction_takes_the_repair_about_her_out_of_use_for_everybody() {
  var ines = InesRepairDone();
  var bo = BoRepairOpen();
  var request = Ask(RequestKind.Restriction, null, "ines@tenant.example", "She asked us to stop using her data while a complaint is looked at.");
  runas (Ada) {
    var done = PrivacyRequest.Single(r => r.Id == request);
    Assert.Equal("", done.Problem);
    Assert.Equal(RequestState.Done, done.State);
  }
  // Nobody reads it: not the coordinator who oversees every repair, not the caretaker who did it.
  runas (Lena) {
    Assert.Equal(0, Assignment.Count(a => a.Id == ines));
    Assert.Equal(0, AssignmentActivity.Count(l => l.Assignment.Id == ines));
    Assert.Equal(0, AssignmentCriterion.Count(c => c.Assignment.Id == ines));
    // Bo's is in ordinary use.
    Assert.Equal(1, Assignment.Count(a => a.Id == bo));
  }
  runas (Omar) { Assert.Equal(0, Assignment.Count()); }
}
```

From [`tests/guards.test.osy`](tests/guards.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.Assignments;
using Osysharp.Privacy;
using Osysharp.Privacy.Assignments;
using Housing;

[Test(Linden)]
void her_copy_holds_the_assignment_about_her_and_nobody_elses() {
  InesRepairDone();
  BoRepairOpen();
  var request = Ask(RequestKind.Access, null, "ines@tenant.example", "She wrote from the address on her lease and gave the flat.");
  runas (Ada) {
    var done = PrivacyRequest.Single(r => r.Id == request);
    Assert.Equal("", done.Problem);
    Assert.Equal(RequestState.Done, done.State);
    Assert.Equal(1, PlanLine.Single(l => l.Request == done && l.Category == "assignments.about").Items);
    var export = ExportOf(done);
    // The repair about her, every sentence: what was asked for, what done meant and what met it, the work, each line.
    Assert.Contains("\"category\":\"assignments.about\"", export);
    Assert.Contains("kitchen tap leaks", export);
    Assert.Contains("A new washer; Ines saw it run dry.", export);
    Assert.Contains("\"Met: Ines Falk has been told", export);
    Assert.Contains("Visit Ines on Tuesday", export);
    Assert.Contains("Ines Falk was not home at first", export);
    Assert.Contains("Fixed for Ines Falk on the second visit.", export);
    Assert.Contains("\"State\":\"Completed\"", export);
    Assert.Contains("Kept 24 months from when the assignment was closed", export);
    // She was on no assignment herself, and nobody else's repair is in her copy.
    Assert.DoesNotContain("\"category\":\"assignments.people\"", export);
    Assert.DoesNotContain("hallway light", export);
    Assert.DoesNotContain("Bo Ek", export);
  }
  // Asking for a copy changed nothing.
  Assert.Contains("Ines Falk", Says(InesRepairId()));
}
```

From [`tests/requests.test.osy`](tests/requests.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.Assignments;
using Osysharp.Privacy;
using Osysharp.Privacy.Assignments;
using Housing;

[Test(Linden)]
void nobody_erases_an_assignment_by_hand_and_a_privacy_request_does() {
  var ines = InesRepairDone();
  // The co-operative named nobody to erase assignments. Lena asked for the repair and oversees every one: she may not…
  runas (Lena) {
    Assert.Denied(() => AnonymiseAssignment(Repair(ines), new AssignmentScrub { Blank = true, Why = "Tidying up." }));
    Assert.Denied(() => TakePersonOffAssignment(Repair(ines), Omar, "somebody", "Tidying up."));
    Assert.Denied(() => { RewriteActivity(Repair(ines).Activity.OrderBy(l => l.Seq).First()); UnitOfWork.Commit(); });
    UnitOfWork.Discard();
  }
  // …nor may Omar, who owns it.
  runas (Omar) { Assert.Denied(() => AnonymiseAssignment(Repair(ines), new AssignmentScrub { Blank = true, Why = "Tidying up." })); }
  Assert.Contains("Ines Falk", Says(ines));

  // Positive control: her request, approved, erases it.
  var request = Ask(RequestKind.Erasure, null, "ines@tenant.example", "She wrote from the address on her lease and gave the flat.");
  runas (Ada) { Assert.Equal(RequestState.Done, PrivacyRequest.Single(r => r.Id == request).State); }
  Assert.DoesNotContain("Ines", Says(ines));
}
```

From [`tests/guards.test.osy`](tests/guards.test.osy).

## Learn more

- [The Osysharp.Privacy.Assignments reference](https://osysharp.com/reference/agent/kit-privacy-assignments/)
- [Osysharp.Privacy.Assignments in the package catalogue](https://osysharp.com/packages/#osysharp-privacy-assignments)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
