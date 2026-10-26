# Osysharp.Privacy.Sms

**The texts an app sent a person, as `Osysharp.Privacy`'s holder** — in their copy of their data, and erased when they
ask.

```osy
app Desk {
  use Osysharp.Sms@0;
  use Osysharp.Privacy@0;
  // Osysharp.Privacy.Sms is not written: it arrives by itself, because the app uses what it completes (`osy lock` says so)
}
```

- One category, `sms.sent` (contract), found by the number a request names: `FileRequest(…, phone: "+46…")`, which staff
  check is the person's by calling it. A request a person makes for themselves proves an ADDRESS, so it reaches texts
  only through a desk's conversation with them (`Osysharp.Privacy.Conversations`, by the channel).
- An adapter, so neither kit forces the other on an app.

Its test app (`tests/app.osy`) and `tests/texts.test.osy` show a request reaching sent texts.

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.Sms;
using Osysharp.Privacy;
using Osysharp.Privacy.Sms;
using PrivacySmsKitTests;

[Test(Desk)]
[runas(Ada)]
void an_erasure_naming_her_checked_number_takes_the_texts_to_it_and_no_others() {
  var r = FileRequest(RequestKind.Erasure, null, "", IdentityCheck.InPerson, "Called the number; she answered and confirmed.",
                      phone: "+46 70 111 22 33");
  Assert.Equal(2, PlanLine.Single(l => l.Request == r && l.Category == "sms.sent").Items);
  ApproveRequest(r);
  var done = PrivacyRequest.Single(x => x.Id == r.Id);
  Assert.Equal(RequestState.Done, done.State);
  Assert.Null(done.SubjectPhone);                         // the request keeps no copy of the number it erased
  Assert.Equal(0, OutgoingSms.Count(m => m.To == "+46701112233"));
  Assert.Equal(1, OutgoingSms.Count());
  Assert.NoTraceOf("KIWI2");
}
```

From [`tests/texts.test.osy`](tests/texts.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.Sms;
using Osysharp.Privacy;
using Osysharp.Privacy.Sms;
using PrivacySmsKitTests;

[Test(Desk)]
[runas(Ada)]
void her_copy_lists_the_texts_we_sent_her() {
  var json = new SmsPersonalData().Export(new DataSubject { Phones = ["+46701112233"] }, CategoryOf("sms.sent")).Json;
  Assert.Contains("on its way", json);
  Assert.DoesNotContain("Somebody else", json);
}
```

From [`tests/texts.test.osy`](tests/texts.test.osy).

## Learn more

- [Osysharp.Privacy.Sms in the package catalogue](https://osysharp.com/packages/#osysharp-privacy-sms)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
