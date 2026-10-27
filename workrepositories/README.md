# Osysharp.Privacy.WorkRepositories

**What a piece of work knows about a person, under that person's privacy request — each kind for its own time.** Add
it beside `Osysharp.WorkRepositories` and `Osysharp.Privacy`, and the items of an assignment's repository that are
about a person are part of what they can see, have kept out of use, or have erased; and each kind of item is erased
for everybody when its own period ends.

```osy
use Osysharp.WorkRepositories@1;
use Osysharp.Privacy@0;
// Osysharp.Privacy.WorkRepositories is not written: it arrives by itself, because the app uses what it completes (`osy lock` says so)
```

Nothing is wired: the holder is found by itself.

## Use case

A design studio is hiring an illustrator. The hire is one assignment; its repository holds the brief, each
candidate's application, what each interviewer thought of them, and the letters. The studio's rules:

```osy
app.WorkRepositories = new WorkRepositoriesSetup {
  Kinds = [
    new KindRule { Kind = ItemKind.Finding, Restricted = true, RetainFor = TimeSpan.FromDays(365) },   // interview feedback
    new KindRule { Kind = ItemKind.Document, RetainFor = TimeSpan.FromDays(180) },                      // applications
    new KindRule { Kind = ItemKind.Correspondence, RetainFor = TimeSpan.FromDays(30), From = RetainFrom.Added },
  ],
};
```

**A candidate asks for her copy.** It holds her application, the letter to her, and what *both* interviewers wrote
about her — feedback nobody at the studio but its writer and the hiring manager may read is hers to see. Nothing of
the other candidate is in it. The privacy desk answers the request without being able to read one item of the hire.

**She withdraws, and asks to be erased.** Her application, the letter and the feedback on her lose everything they
said — every version's content, what changed, where it came from, why it was accepted — and their titles come out of
the hire's activity ("Document added: an item that was erased"). Each item keeps its place in the record: that there
was one, what kind, who added it and when, that it was erased and why. The other candidate's items are word for word
as they were, the hire is still open, and it goes on.

**Nobody asks, and time passes.** Letters go thirty days after they were written; both applications six months
after the hire closes; the feedback a year after. The brief — a kind with no time of its own — stays.

## Whose data an item is

```osy
public RepositoryItem ReceiveApplication(Assignment hire, string name, string email, string text) {
  var item = AddItem(hire, new NewItem { Kind = ItemKind.Document, Title = "Application: " + name, Content = text });
  MarkItemAbout(item, new DataSubject { Emails = [email] });       // where it is added
  return item;
}
```

When everything in a repository is about one person — a reference check, a customer's case — say it once for the
assignment instead (`MarkAbout(a.Id, "Assignment", subject)`, or `MarkAssignmentAbout` with
[`Osysharp.Privacy.Assignments`](../privacy-assignments)): every item of a marked assignment is found without a mark
of its own. The piece of an agent's work a version was written in is about the same person too.

## What it holds, and for how long

| Category | What | Kept |
|---|---|---|
| `repositories.<kind>` — one for each kind your app gave a period: `repositories.document`, `repositories.finding`, `repositories.correspondence`… | the items of that kind that are about a person | your `KindRule.RetainFor`, in months, from when the item was added or from when its assignment closed |
| `repositories.items` | every other kind | as long as the assignment (or `KeepOtherMonths` after it closed) |
| `repositories.contributions` | what a person **added** — found by their account | until they ask |

So the records of processing, a person's privacy page and every answer name each kind with its own period — and the
tenant changes a period on the retention page without touching code. The tenant's period wins over the app's rule.
Each item's own date (`RetainUntil`) is set to the day the purge will take it, so the repository says what will
happen.

## What a request does

| | |
|---|---|
| **A copy** | Each item about them: its kind, its title, where it stands and why, and every version — what it said, what changed, where it came from, when. A contributor's copy holds what they wrote. |
| **Erasure, of the person it is about** | `EraseItem` on each: every version's content, data and file, its note and source, why the item stands where it does, what a disagreement it is in was about — and its title and notes come out of the assignment's activity. The item, its kind, who added it and when, its versions' times, its standing, and that it was erased and why, stay. |
| **Erasure, of a contributor** | They are taken off what they added (`TakeContributorOffRepositories`): the items and versions stay, word for word, as added by *Somebody no longer here*; what was opened to them is closed. |
| **Restriction** | The items about them, with every version, leave every ordinary read. |
| **A legal hold** | Items about a held person — marked themselves, or in an assignment marked as about them — are neither erased nor purged. |

Every day, for everybody: each item whose kind's period has ended is erased, marked or not.

## Who reads what stays the same

An erased item is still an item of its assignment, and still restricted if it was: exactly the same people read
exactly the same items before and after. The test app checks it for each interviewer, the recruiter who oversees
every hire and reads no feedback, the hiring manager who was given it, and the privacy desk, which reads none.

## Its tests

`kits/privacy-work-repositories/tests` is that studio: a hiring manager, a recruiter, two interviewers, a records
officer, two candidates with no account, and Ada at the privacy desk.

| | |
|---|---|
| A candidate's copy holds her application, the letter and what was written about her, and no other candidate's | `a_candidates_copy_holds_her_application_the_letter_and_what_was_written_about_her_and_no_other_candidates` |
| A candidate who withdraws is erased while the hire goes on | `a_candidate_who_withdraws_is_erased_while_the_hire_goes_on` |
| An interviewer who leaves is taken off what she wrote, and the feedback stays | `an_interviewer_who_leaves_is_taken_off_what_she_wrote_and_the_feedback_stays` |
| An item of an assignment marked as about somebody is found without a mark of its own | `an_item_of_an_assignment_marked_as_about_somebody_is_found_without_a_mark_of_its_own` |
| Each kind is erased when its own time is up, for everybody, and nothing else is | `each_kind_is_erased_when_its_own_time_is_up_for_everybody_and_nothing_else_is` |
| The tenant's own period wins over the app's rule | `the_tenants_own_period_wins_over_the_apps_rule` |
| A legal hold on a candidate keeps what is about them through the purge | `a_legal_hold_on_a_candidate_keeps_what_is_about_them_through_the_purge` |
| A restriction takes what is about her out of use for everybody | `a_restriction_takes_what_is_about_her_out_of_use_for_everybody` |
| What others read is the same before and after a candidate is erased | `what_others_read_is_the_same_before_and_after_a_candidate_is_erased` |
| Nobody who could not erase an item before can now; a request does | `nobody_who_could_not_erase_an_item_before_can_now_and_a_privacy_request_does` |

```
osy test kits/privacy-work-repositories/tests
```

## Source and package

- Package: `Osysharp.Privacy.WorkRepositories` — it completes `Osysharp.Privacy` and `Osysharp.WorkRepositories`, and arrives by itself.
- Source: [`kits/privacy-work-repositories`](.) — `model/Holder.osy` (the holder, `MarkItemAbout`, `ItemsAbout`, the copy).
- Builds on [`Osysharp.WorkRepositories`](../work-repositories) (`EraseItem`, `KeepItemUntil`,
  `TakeContributorOffRepositories`), [`Osysharp.Assignments`](../assignments) and [`Osysharp.Privacy`](../privacy)
  (the holder contract, `MarkAbout`, requests, the purge, holds).
- With the daily purge running you need not schedule the repositories kit's own `EraseExpiredItems`: the purge sets
  each item's date and takes it on that day.

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.Assignments;
using Osysharp.WorkRepositories;
using Osysharp.Privacy;
using Osysharp.Privacy.WorkRepositories;
using Recruiting;

[Test(Fjord)]
void the_tenants_own_period_wins_over_the_apps_rule() {
  TestClock.Set(new DateTime(2026, 3, 2, 10, 0, 0));
  var h = HireUnderWay();
  CloseTheHire(h.Id);
  // The studio's rule says six months for applications; the tenant keeps them a year.
  runas (Ada) { SetRetention("repositories.document", 12, RetentionStart.Collected); }

  TestClock.Set(new DateTime(2026, 10, 5, 10, 0, 0));
  runas (Ada) { PurgeNow(); }
  Assert.Equal("Ten years of picture books; portfolio enclosed.", Says(h.MajasApplication));
  runas (Otto) { Assert.Equal(new DateTime(2027, 3, 3), Item(h.MajasApplication).RetainUntil); }

  TestClock.Set(new DateTime(2027, 3, 5, 10, 0, 0));
  runas (Ada) { PurgeNow(); }
  Assert.Equal("", Says(h.MajasApplication));
}
```

From [`tests/guards.test.osy`](tests/guards.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.Assignments;
using Osysharp.WorkRepositories;
using Osysharp.Privacy;
using Osysharp.Privacy.WorkRepositories;
using Recruiting;

[Test(Fjord)]
void an_item_of_an_assignment_marked_as_about_somebody_is_found_without_a_mark_of_its_own() {
  // A reference check is its own assignment, about one candidate: the studio says so once, for the assignment.
  Guid check = Guid.Empty;
  Guid note = Guid.Empty;
  runas (Hanna) {
    var a = OpenAssignment(new NewAssignment { Title = "Check a reference", Goal = "We know what the last studio thought.", Criteria = ["The reference is on file"], Owner = Rut });
    MarkAbout(a.Id, "Assignment", new DataSubject { Emails = ["sven@candidate.example"] });
    check = a.Id;
  }
  runas (Rut) {
    note = AddItem(Hire(check), new NewItem { Kind = ItemKind.Evidence, Title = "Call with the last studio", Content = "They would hire Sven again." }).Id;
    MeetCriterion(Hire(check).Criteria.Single(), "On file.");
    CompleteAssignment(Hire(check), "A good reference.");
  }

  var request = Ask(RequestKind.Erasure, null, "sven@candidate.example", "He wrote from the address on his application.");

  runas (Ada) {
    var done = PrivacyRequest.Single(r => r.Id == request);
    Assert.Equal("", done.Problem);
    Assert.Equal(RequestState.Done, done.State);
    // Evidence has no period of its own here, so it is in the general category.
    Assert.Equal(1, PlanLine.Single(l => l.Request == done && l.Category == "repositories.items").Items);
  }
  Assert.Equal("", Says(note));
  runas (Otto) { Assert.Equal("Evidence (erased)", Item(note).Title); }
}
```

From [`tests/hire.test.osy`](tests/hire.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.Assignments;
using Osysharp.WorkRepositories;
using Osysharp.Privacy;
using Osysharp.Privacy.WorkRepositories;
using Recruiting;

[Test(Fjord)]
void a_legal_hold_on_a_candidate_keeps_what_is_about_them_through_the_purge() {
  TestClock.Set(new DateTime(2026, 3, 2, 10, 0, 0));
  var h = HireUnderWay();
  CloseTheHire(h.Id);
  runas (Ada) { PlaceHold("sven@candidate.example", null, "A discrimination complaint", "His application is part of it.", new DateTime(2028, 1, 1)); }
  Background.Settle();

  TestClock.Set(new DateTime(2026, 10, 5, 10, 0, 0));
  runas (Ada) { PurgeNow(); }
  // Maja's application reached its time and went. Sven's is untouched: a hold covers him.
  Assert.Equal("", Says(h.MajasApplication));
  Assert.Equal("Editorial work for two dailies.", Says(h.SvensApplication));
  runas (Ada) { Assert.False(PurgeRecord.Any(p => p.Failed || p.Deferred)); }
}
```

From [`tests/guards.test.osy`](tests/guards.test.osy).

## Learn more

- [The Osysharp.Privacy.WorkRepositories reference](https://osysharp.com/reference/agent/kit-privacy-work-repositories/)
- [Osysharp.Privacy.WorkRepositories in the package catalogue](https://osysharp.com/packages/#osysharp-privacy-workrepositories)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
