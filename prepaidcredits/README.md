# Osysharp.Privacy.PrepaidCredits

Osysharp.Privacy's holder for [Osysharp.PrepaidCredits](https://github.com/osysharp): a person's packs, what they used
them for and every change to them — counted in a privacy request's plan, in their copy of their data, and erased when
they ask.

## Use it

Nothing to write. In an app that uses both Osysharp.Privacy and Osysharp.PrepaidCredits it arrives by itself
(`osy lock` includes it and says so); to keep it out, `without Osysharp.Privacy.PrepaidCredits;` in `app.osy`.

```osy
use Osysharp.Privacy@0;
use Osysharp.PrepaidCredits@0;
```

Its category is `credits.ledger` — *Your prepaid credits and what you used them for* — held under the contract, and
portable. Erasing takes the person's credits with it: what they paid for and did not use is settled with them in
money first; the sale itself stays in the books.

## Source

[github.com/osysharp/privacy/prepaidcredits](https://github.com/osysharp/privacy/tree/main/prepaidcredits); its tests are a pottery studio's.

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.Privacy;
using Osysharp.PrepaidCredits;
using Osysharp.Privacy.PrepaidCredits;

[Test(Studio)]
[runas(Ada)]
void her_copy_lists_her_packs_and_what_she_used_them_for_and_nobody_elses() {
  PacksSold();
  var holder = new PrepaidCreditsPersonalData();
  var ann = new DataSubject { Account = Ann.Id };
  Assert.Equal(2, holder.Count(ann, CategoryOf("credits.ledger")));
  var copy = holder.Export(ann, CategoryOf("credits.ledger"));
  Assert.Contains("Six wheel sessions", copy.Json);
  Assert.Contains("Wheel, Tuesday", copy.Json);
  Assert.DoesNotContain("glaze", copy.Json);
  Assert.Equal(2, copy.Items.Count);
  Assert.Equal("Granted — Six wheel sessions", copy.Items[0].Title);
}
```

From [`tests/credits.test.osy`](tests/credits.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.Privacy;
using Osysharp.PrepaidCredits;
using Osysharp.Privacy.PrepaidCredits;

[Test(Studio)]
[runas(Ada)]
void an_erasure_takes_her_credits_and_leaves_his() {
  PacksSold();
  var r = FileRequest(RequestKind.Erasure, Ann.Id, "", IdentityCheck.InPerson, "Came to the studio and asked.");
  Assert.Equal(2, PlanLine.Single(l => l.Request == r && l.Category == "credits.ledger").Items);
  ApproveRequest(r);
  Assert.Equal(RequestState.Done, PrivacyRequest.Single(x => x.Id == r.Id).State);
  Assert.Equal(0, CreditPacksOf(Ann).Count);
  Assert.Equal(0, CreditStatementOf(Ann).Count);
  Assert.Equal(0, CreditSpendsOf(Ann).Count);
  Assert.Equal(3, CreditsLeft(Bo));
  Assert.NoTraceOf("Wheel, Tuesday");
}
```

From [`tests/credits.test.osy`](tests/credits.test.osy).

## Learn more

- [The Osysharp.Privacy.PrepaidCredits reference](https://osysharp.com/reference/ui/prepaid-credits-kit/)
- [Osysharp.Privacy.PrepaidCredits in the package catalogue](https://osysharp.com/packages/#osysharp-privacy-prepaidcredits)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
