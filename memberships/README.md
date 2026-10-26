# Osysharp.Privacy.Memberships

Osysharp.Privacy's holder for [Osysharp.Memberships](https://github.com/osysharp): the memberships a person holds, the
ones they are on, every change and every time they came or were turned away — counted in a privacy request's plan, in
their copy of their data, and erased when they ask.

## Use it

Nothing to write. In an app that uses both Osysharp.Privacy and Osysharp.Memberships it arrives by itself (`osy lock`
includes it and says so); to keep it out, `without Osysharp.Privacy.Memberships;` in `app.osy`.

```osy
use Osysharp.Privacy@0;
use Osysharp.Memberships@0;
```

Its category is `memberships` — *Your memberships and your visits* — held under the contract, and portable. Erasing
ends what the person holds (what was paid and not used — `UnusedValueOf` — is settled with them first); somebody else's
membership they were on goes on without them. A card's visits are erased with Osysharp.Privacy.PrepaidCredits, which
arrives beside it.

## Source

[github.com/osysharp/privacy/memberships](https://github.com/osysharp/privacy/tree/main/memberships); its tests are a leisure centre's.

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.Privacy;
using Osysharp.PrepaidCredits;
using Osysharp.Memberships;
using Osysharp.Privacy.Memberships;

[Test(Centre)]
[runas(Ada)]
void her_copy_lists_what_she_holds_what_she_is_on_and_when_she_came() {
  Sold();
  var holder = new MembershipsPersonalData();
  var ann = new DataSubject { Account = Ann.Id };
  Assert.Equal(3, holder.Count(ann, CategoryOf("memberships")));
  var copy = holder.Export(ann, CategoryOf("memberships"));
  Assert.Contains("Gym card", copy.Json);
  Assert.Contains("Couple year", copy.Json);
  Assert.Contains("Spa entrance", copy.Json);
  Assert.DoesNotContain("Pool", copy.Json);
  Assert.Equal(3, copy.Items.Count);
}
```

From [`tests/memberships.test.osy`](tests/memberships.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.Privacy;
using Osysharp.PrepaidCredits;
using Osysharp.Memberships;
using Osysharp.Privacy.Memberships;

[Test(Centre)]
[runas(Ada)]
void an_erasure_takes_her_memberships_visits_and_credits_and_leaves_his() {
  Sold();
  var r = FileRequest(RequestKind.Erasure, Ann.Id, "", IdentityCheck.InPerson, "Came to reception and asked.");
  Assert.Equal(3, PlanLine.Single(l => l.Request == r && l.Category == "memberships").Items);
  ApproveRequest(r);
  Assert.Equal(RequestState.Done, PrivacyRequest.Single(x => x.Id == r.Id).State);
  Assert.Equal(0, MembershipsOf(Ann).Count);
  Assert.Equal(0, AdmissionsOf(Ann).Count);
  Assert.Equal(0, CreditPacksOf(Ann).Count);
  Assert.Equal(1, MembershipsOf(Bo).Count);
  Assert.Equal(0, MembersOf(MembershipsOf(Bo)[0]).Count);
  Assert.Equal(1, AdmissionsOf(Bo).Count);
  Assert.NoTraceOf("Spa entrance");
}
```

From [`tests/memberships.test.osy`](tests/memberships.test.osy).

## Learn more

- [The Osysharp.Privacy.Memberships reference](https://osysharp.com/reference/ui/memberships-kit/)
- [Osysharp.Privacy.Memberships in the package catalogue](https://osysharp.com/packages/#osysharp-privacy-memberships)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
