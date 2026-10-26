# Osysharp.Privacy.UserAccounts

**The step-up a person proves themselves with before asking about their own data**, for an app that signs people in
with `Osysharp.UserAccounts` and answers privacy requests with `Osysharp.Privacy`.

```osy
app Club {
  use Osysharp.UserAccounts@1;
  use Osysharp.Privacy@0;
  // Osysharp.Privacy.UserAccounts is not written: it arrives by itself, because the app uses what it completes (`osy lock` says so)
}
```

- `AccountsRequesterProof` implements Privacy's `IRequesterProof` with Accounts' step-up (`CodePurpose.PrivacyRequest`):
  a six-digit code mailed to the address the account holds, typed back within fifteen minutes, spent once by the
  request's own run.
- An adapter, so neither kit forces the other on an app.

Its test app (`tests/app.osy`) and `tests/proof.test.osy` show the step-up end to end.

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.Privacy;
using Osysharp.Privacy.UserAccounts;

[Test(Family)]
void the_guardians_copy_lists_the_account_she_looks_after() {
  runas (Ann) { MakeManagedAccount("Sam"); }
  // The holder reads as the request's run does — as the system; the sign-in flow stands in for it here.
  runas (AuthBootstrap) {
    var ann = User.Single(u => u.Email == "ann@club.test");
    var holder = new ManagedAccountsPersonalData();
    var me = new DataSubject { Account = ann.Id };
    Assert.Equal(1, holder.Count(me, CategoryOf("accounts.managed")));
    var copy = holder.Export(me, CategoryOf("accounts.managed"));
    Assert.Contains("Sam", copy.Json);
    Assert.Equal(1, copy.Items.Count);
    Assert.Equal("An account you look after", copy.Items[0].Title);
    // Somebody who looks after nobody has nothing here.
    Assert.Equal(0, holder.Count(new DataSubject { Account = User.Single(u => u.Email == "ada@club.test").Id }, CategoryOf("accounts.managed")));
  }
}
```

From [`tests/managed.test.osy`](tests/managed.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.Mail;
using Osysharp.Privacy;
using Osysharp.Privacy.UserAccounts;

[Test(Club)]
void the_accounts_step_up_is_found_without_being_wired() {
  runas (Ann) { Assert.True(CanAskForMyself()); }
}
```

From [`tests/proof.test.osy`](tests/proof.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.Privacy;
using Osysharp.Privacy.UserAccounts;

[Test(Family)]
void erasing_the_person_it_was_made_for_erases_their_account_and_its_name() {
  runas (Ann) { MakeManagedAccount("Sam"); }
  var samId = Guid.Empty;
  runas (AuthBootstrap) { samId = ManagedAccountRequest.Single(r => r.Name == "Sam").Account!.Id; }
  runas (Ada) {
    var r = FileRequest(RequestKind.Erasure, samId, "", IdentityCheck.InPerson, "His mother came to the desk with him.");
    Assert.Equal(1, PlanLine.Single(l => l.Request == r && l.Category == "accounts.managed").Items);
    ApproveRequest(r);
    Assert.Equal(RequestState.Done, PrivacyRequest.Single(x => x.Id == r.Id).State);
  }
  runas (AuthBootstrap) {
    Assert.False(ManagedAccountRequest.Any(r => r.Name == "Sam"));
    var erased = ManagedAccountRequest.Single().Account!;
    Assert.False(erased.IsActive);
    Assert.Equal("", erased.DisplayName);
  }
}
```

From [`tests/managed.test.osy`](tests/managed.test.osy).

## Learn more

- [Osysharp.Privacy.UserAccounts in the package catalogue](https://osysharp.com/packages/#osysharp-privacy-useraccounts)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
