# Osysharp.Privacy.MailReceiving

**What an app's watched mailboxes hold about a person, as `Osysharp.Privacy`'s holder** — the mail they sent, as it
arrived (`ReceivedMail` and its original), in their copy of their data, and erased when they ask.

```osy
app Desk {
  use Osysharp.Mail.Receiving@0;
  use Osysharp.Privacy@0;
  // Osysharp.Privacy.MailReceiving is not written: it arrives by itself, because the app uses what it completes (`osy lock` says so)
}
```

- One category, `mail.received` (contract, portable), found by the exact sender address (`EraseMailFrom`).
- An adapter, so neither kit forces the other on an app.

Its test app (`tests/app.osy`) and `tests/received.test.osy` show a request reaching received mail.

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.Mail.Receiving;
using Osysharp.Privacy;
using Osysharp.Privacy.MailReceiving;
using PrivacyMailReceivingKitTests;

[Test(Desk)]
[runas(Ada)]
void her_erasure_takes_the_mail_she_sent_and_never_mail_from_an_address_containing_hers() {
  var r = FileRequest(RequestKind.Erasure, null, "ann@x.test", IdentityCheck.CodeToKnownAddress, "A code to her address came back.");
  Assert.Equal(2, PlanLine.Single(l => l.Request == r && l.Category == "mail.received").Items);
  ApproveRequest(r);
  Assert.Equal(RequestState.Done, PrivacyRequest.Single(x => x.Id == r.Id).State);
  Assert.Equal(0, ReceivedMail.Count(m => m.From == "ann@x.test"));
  Assert.Equal(1, ReceivedMail.Count(m => m.From == "joann@x.test"));
  Assert.NoTraceOf("OKRA1");
}
```

From [`tests/received.test.osy`](tests/received.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.Mail.Receiving;
using Osysharp.Privacy;
using Osysharp.Privacy.MailReceiving;
using PrivacyMailReceivingKitTests;

[Test(Desk)]
[runas(Ada)]
void her_copy_lists_what_she_sent_us() {
  var h = new ReceivedMailPersonalData();
  var json = h.Export(new DataSubject { Emails = ["ann@x.test"] }, CategoryOf("mail.received")).Json;
  Assert.Contains("Where is OKRA1", json);
  Assert.DoesNotContain("Unrelated", json);
}
```

From [`tests/received.test.osy`](tests/received.test.osy).

## Learn more

- [Osysharp.Privacy.MailReceiving in the package catalogue](https://osysharp.com/packages/#osysharp-privacy-mailreceiving)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
