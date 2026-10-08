# Osysharp.Privacy.Conversations

**What a support desk's conversations hold about a person, as `Osysharp.Privacy`'s holder** — which of it is erased
when they ask, which is kept RESTRICTED for legal claims (out of use, read only for a claim, every read logged), and
when each is gone for everyone.

```osy
app Desk {
  use Osysharp.Conversations@0;
  use Osysharp.Privacy@0;
  // Osysharp.Privacy.Conversations is not written: it arrives by itself, because the app uses what it completes (`osy lock` says so)
}

public class DeskPrivacy : ConversationPrivacyRules {
  // Which conversations concern a claim is the app's to say — the kit cannot know what a complaint is.
  public override bool ConcernsAClaim(Conversation c) => Complaint.Any(x => x.Conversation == c.Id);
  // What the app keeps beside a conversation — a reply draft on its ticket — goes when its words do…
  public override void Erased(Conversation c) {
    foreach (var d in ReplyDraft.Where(x => x.Conversation == c.Id).ToList()) { Restriction.Lift(d); d.Delete(); }
  }
  // …and is kept, the same way, while a complaint is kept.
  public override void Kept(Conversation c, string category, DateTime? until) {
    foreach (var d in ReplyDraft.Where(x => x.Conversation == c.Id).ToList()) { Restriction.Restrict(d, category, until); }
  }
}
app.ConversationsPrivacy = new ConversationsPrivacySetup { Rules = new DeskPrivacy(), KeepForMonths = 12 };
// The Conversations kit classifies its fields `Personal.*`: say who reads them — whoever each row's security admits.
app = new() { Classifications = [ new Classification(Personal.Contact) { Open = true },
                                  new Classification(Personal.Content) { Open = true },
                                  new Classification(Personal.Identity) { Open = true } ] };
```

| Category | Basis | When a person asks | For everyone |
|---|---|---|---|
| `conversations.claims` — a complaint or a dispute | legal claims, 36 months from its last entry (the tenant sets it) | **kept restricted** — every entry, rendering, file and delivery — and the person is told "kept … for the establishment, exercise or defence of legal claims, until …" | lifted and erased when its period ends (a legal hold keeps it past that) |
| `conversations.service` — every other conversation | legitimate interest | their words, the subjects they wrote and the files marked about them are erased | erased `KeepForMonths` after its last entry — the app's default, the tenant's on the retention page |
| `conversations.contact-points` — name, addresses, numbers | contract | erased, and every copy a channel's outbox keeps — or, while a complaint of theirs is kept, **kept restricted with it** (`KeptWith`) and erased when the last one ends | — |
| `conversations.analysis` — redacted copies search reads | legitimate interest | erased | — |

- **What the desk wrote about them beside their conversation goes with it.** A side conversation the desk started
  beside theirs (asking a carrier "where is Ana's parcel to Elm Street?") is the other side's conversation, but the
  desk's words in it are about the person: their erasure blanks every message and note the desk wrote there, the mail
  that carried it (the channel's `EraseCopyOf`), the side conversation's subject, and the line on their conversation
  that repeats it. The other side's own answers stay theirs. An export of their data does not include them.
- **Files are erased because they are about the person, never because the person sent them**: the desk
  marks one with `MarkFileAbout`. A conversation that reaches the end of its period takes all of its files.
- **Nobody erases any other way.** The Conversations kit records an erasure only from a run (`PartyErasure` is `deny
  create` for every person), so every erasure follows a request's plan or a period's end.
- **A kept complaint is read only for the claim.** `ConversationForAClaim(id)` — a `[PurposeRead("conversations.claims")]`
  for whoever `HandlesClaims` (by default whoever `PlacesLegalHolds`; an app says otherwise with `policy HandlesClaims =>
  …;`) — answers the transcript and who the customers in it are; every read is written to the access trail. Nothing else
  on the desk reads it: the conversation, its messages and the person are absent from every ordinary read.
- **The app keeps what it holds beside a conversation the same way** — `ConversationPrivacyRules.Kept(conversation,
  category, until)` (Harbour marks the ticket "Kept for legal claims until …" and restricts the answers and drafts on it).
- **An Art. 18 restriction** keeps every conversation of theirs, and who they are, out of use until they lift it.
- **Nothing reaches the audit trail.** The Conversations kit classifies its fields `Personal.*`, which the trail records
  as changed, never as themselves — so an app keeps no `app.Audit.Redact` list for them. The kit announces its levels in
  its `package.osy`, and an app says nothing about them: a level it does not map is read through each row's own security.
- **A desk's own rows can be found from a request with no holder written** — the kit ships `PartiesOfASubject`
  (`[FindsDataSubject]`), so an app's `[DataSubject] Customer Customer;` (where `Customer : Party`) is followed through the
  party's addresses (`osy docs security-declared-personal-data`).

Its test app (`tests/app.osy`) with `tests/erasure.test.osy` and `tests/files.test.osy` show an erasure across a
desk's conversations and their files.
