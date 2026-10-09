# Osysharp.Privacy.Accounts

**The step-up a person proves themselves with before asking about their own data**, for an app that signs people in
with `Osysharp.Accounts` and answers privacy requests with `Osysharp.Privacy`.

```osy
app Club {
  use Osysharp.Accounts@1;
  use Osysharp.Privacy@0;
  // Osysharp.Privacy.Accounts is not written: it arrives by itself, because the app uses what it completes (`osy lock` says so)
}
```

- `AccountsRequesterProof` implements Privacy's `IRequesterProof` with Accounts' step-up (`CodePurpose.PrivacyRequest`):
  a six-digit code mailed to the address the account holds, typed back within fifteen minutes, spent once by the
  request's own run.
- An adapter, so neither kit forces the other on an app.

Its test app (`tests/app.osy`) and `tests/proof.test.osy` show the step-up end to end.
