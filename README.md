# Osysharp.Privacy

**Erasure that keeps only what the company must or may.** A person asks to be erased; their account and notes go now,
their name on an invoice stays — restricted, for the books only — until the day the law no longer requires it, and the
response tells them exactly that.

```osy
app Desk {
  use Osysharp.Privacy@0;
  use Osysharp.Workflow;
  use Osysharp.Observability;   // purpose reads are logged to the access trail
  use Osysharp.Storage;         // exports are files written about their request
}

policy ManagesPrivacy => IsAdmin;
policy PlacesLegalHolds => IsAdmin;
policy WorksInbox => IsStaff;   // …and the Inbox's questions: a request is inbox work

public class DeskPersonalData : IPersonalDataHolder { … }   // what the app holds, by category — discovered, never wired
```

- **One contract** (`IPersonalDataHolder`, versioned) every kit and the app implement; holders are discovered with
  `App.Implementations<IPersonalDataHolder>()`.
- **Categories** with a retention basis (consent, contract, legal obligation, legal claims, legitimate interest) and a
  kept period in months from a start event — fiscal-year aware (`KeepUntil`).
- **The tenant's settings** within the kit's limits, or outside them with a stated jurisdiction and reason; seven
  bookkeeping presets (SE, DE, FR, NL, DK, FI, UK — verify with your adviser); categories that need confirming.
- **Requests** (access, portability, erasure, restriction) planned per category — not held · erase · keep restricted
  until · blocked · keep on compelling grounds · export — and run by a workflow that fails naming the holder.
- **The response** — what was erased, what is kept on which basis until which day, what waits, where to complain.
- **Exports** — Art. 15 with each category's purpose, basis and retention; Art. 20 portable categories only.
- **A daily purge**, recorded per category; the kit's own records kept 36 months under legal claims.
- **Legal holds** (0.2.0) on a person or a case: an erasure waits on the hold (the person reads only "kept for legal
  claims"), the purge leaves held rows alone (`IPersonalDataHolder.Hold`, with `Restriction.SetUntil`), only
  `PlacesLegalHolds` places or lifts one, a lift needs a reason, a review date keeps a hold from becoming for ever, and
  `HoldLiftNeedsSecondPerson` (an organisation may set it) asks a second person to confirm. Page:
  `/settings/privacy/holds`.

- **Whose data a kit's row is** (0.9.0): the code that starts a piece of work says who it is about —
  `MarkAbout(assignment.Id, "Assignment", new DataSubject { Emails = [order.CustomerEmail] })` — and every holder
  finds the row from what a request proved (`RowsAbout`), with what hangs under it (`ISubjectLinks`). A mark keeps a
  fingerprint of an address, never the address; only the privacy desk reads it back; a request erases the marks last.

- **Requests on the Inbox** (0.3.0): `PrivacyRequest : WorkItem`, due one calendar month after receipt (an absolute
  `DueAt`), extendable with a reason. Staff record how they checked who it is; a signed-in person asks on `/privacy`
  after a step-up (Osysharp.Privacy.UserAccounts, which arrives by itself in an app that also uses Accounts), and the answer goes only to the address they
  proved; exports are fetched once, within seven days.

**Works with the other kits.** Each kit that holds personal data answers for it through its own small holder kit:
[Osysharp.Privacy.Conversations](https://osyrin.com/templates/kits/privacy-conversations/),
[Osysharp.Privacy.MailReceiving](https://osyrin.com/templates/kits/privacy-mail-receiving/),
[Osysharp.Privacy.Notifications](https://osyrin.com/templates/kits/privacy-notifications/),
[Osysharp.Privacy.Sms](https://osyrin.com/templates/kits/privacy-sms/), and
[Osysharp.Privacy.UserAccounts](https://osyrin.com/templates/kits/privacy-user-accounts/) for the signed-in step-up.
The agent family has four: [Osysharp.Privacy.Agents](../privacy-agents) (what an AI agent did for and about a
person — erased while its record's hash chain still verifies), [Osysharp.Privacy.Llm](../privacy-llm) (the model
meter), [Osysharp.Privacy.Assignments](../privacy-assignments) (who asked for a piece of work, who was on it and
what was written on it) and [Osysharp.Privacy.WorkRepositories](../privacy-work-repositories) (what an assignment
knows, each kind kept for its own time). A person without an account asks
the desk: staff file the request on their behalf (`FileRequest`), record how they checked who it is, and deliver the
answer themselves.

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using PrivacyKitTests;
using Osysharp.Privacy;

[Test(Desk)]
[runas(Ada)]
void an_erased_customer_leaves_no_trace_anywhere_but_the_voucher_kept_for_the_books() {
  // A word nobody else would write, in everything Dee gave the desk.
  var dee = new Customer { Number = "C-9", Name = "Dee quillwort-9", Email = "dee@customer.test" };
  new Note { Customer = dee, Text = "Dee quillwort-9 asked twice." };
  new Voucher { Number = "V-9", Amount = 40m, IssuedAt = new DateTime(2025, 9, 1), BuyerName = "Dee quillwort-9",
    BuyerEmail = "dee@customer.test" };
  UnitOfWork.Commit();
  Dee_is_found_before_the_erasure();

  var r = FileRequest(RequestKind.Erasure, null, "dee@customer.test", IdentityCheck.KnownDetails, "Matched the order number and the address on it.");
  ApproveRequest(r);
  Assert.Equal(RequestState.Done, PrivacyRequest.Single(x => x.Id == r.Id).State);
  Assert.NoTraceOf("quillwort-9");
  Assert.NoTraceOf("dee@customer.test");
}
```

From [`tests/copies.test.osy`](tests/copies.test.osy).

```osy
using PrivacyKitTests;
using Osysharp.Identity;
using Osysharp.WorkQueues;
using Osysharp.Privacy;

[Test(Visitors)]
[runas(Ada)]
void whether_a_hold_blocks_a_persons_rows_reads_the_same_however_many_rows_they_have() {
  // A hold in force on someone else's case, so the rows ARE tested against something.
  PlaceCaseHold("Customer", Customer.Single(c => c.Number == "C-1").Id, "Customer C-1", "Chargeback dispute",
                "The card issuer's chargeback is open.", DateTime.UtcNow.AddDays(30));
  // The first question sets up what every later one reuses — it is not what this compares.
  Assert.Equal("", VisitsBlockedFor("cy@customer.test"));
  var mark = Cost.Mark();
  Assert.Equal("", VisitsBlockedFor("ann@customer.test"));
  var one = Cost.StatementsSince(mark);
  mark = Cost.Mark();
  Assert.Equal("", VisitsBlockedFor("bo@customer.test"));
  var twelve = Cost.StatementsSince(mark);
  Log.Information("Blocked by a case hold: " + one + " statements for one visit, " + twelve + " for twelve");
  Assert.Equal(one, twelve);
}
```

From [`tests/cost.test.osy`](tests/cost.test.osy).

```osy
using PrivacyKitTests;
using Osysharp.Identity;
using Osysharp.WorkQueues;
using Osysharp.Privacy;

[Test(Desk)]
void every_holder_is_found_without_being_wired() {
  var keys = Holders().Select(h => h.Key()).ToList();
  Assert.Contains("PrivacyKitTests.Desk", keys);  // the app's own, written by hand — never listed anywhere
  Assert.Contains("PrivacyKitTests", keys);       // …and the one the platform writes from its `[PersonalData]` visits
  Assert.Contains("Osysharp.Privacy", keys);      // the kit's own, for the platform's copies
  Assert.Contains("Osysharp.Mail", keys);         // the outbox the kit's own answers go through
  Assert.Contains("PrivacyKitTests.Backups", keys);   // the desk's backups, which nobody can count by person
  Assert.Equal(5, keys.Count);
}
```

From [`tests/declarations.test.osy`](tests/declarations.test.osy).

## Learn more

- [The Osysharp.Privacy reference](https://osysharp.com/reference/agent/kit-privacy/)
- [Osysharp.Privacy in the package catalogue](https://osysharp.com/packages/#osysharp-privacy)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
