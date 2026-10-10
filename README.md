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

- **Requests on the Inbox** (0.3.0): `PrivacyRequest : WorkItem`, due one calendar month after receipt (an absolute
  `DueAt`), extendable with a reason. Staff record how they checked who it is; a signed-in person asks on `/privacy`
  after a step-up (Osysharp.Privacy.UserAccounts, which arrives by itself in an app that also uses Accounts), and the answer goes only to the address they
  proved; exports are fetched once, within seven days.

**Works with the other kits.** Each kit that holds personal data answers for it through its own small holder kit:
[Osysharp.Privacy.Conversations](https://osyrin.com/templates/kits/privacy-conversations/),
[Osysharp.Privacy.MailReceiving](https://osyrin.com/templates/kits/privacy-mail-receiving/),
[Osysharp.Privacy.Notifications](https://osyrin.com/templates/kits/privacy-notifications/),
[Osysharp.Privacy.Sms](https://osyrin.com/templates/kits/privacy-sms/), and
[Osysharp.Privacy.UserAccounts](https://osyrin.com/templates/kits/privacy-user-accounts/) for the signed-in step-up. A person without an account asks
the desk: staff file the request on their behalf (`FileRequest`), record how they checked who it is, and deliver the
answer themselves.
