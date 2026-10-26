# Osysharp.Privacy.Qualifications

Osysharp.Privacy's holder for [Osysharp.Qualifications](https://github.com/osysharp): the qualifications a person holds, what
they signed and on which text, and the files behind them — counted in a privacy request's plan, in their copy of their
data, and erased when they ask.

## Use it

Nothing to write. In an app that uses both Osysharp.Privacy and Osysharp.Qualifications it arrives by itself (`osy lock`
includes it and says so); to keep it out, `without Osysharp.Privacy.Qualifications;` in `app.osy`.

```osy
use Osysharp.Privacy@0;
use Osysharp.Qualifications@0;
```

Its category is `qualifications` — *Your qualifications and what you signed* — held under the contract, and portable. A signed
waiver an organisation must keep for legal claims is the app's to say: set the category's retention, and a request keeps
it restricted until that ends.

## Source

[github.com/osysharp/privacy/qualifications](https://github.com/osysharp/privacy/tree/main/qualifications); its tests are a makerspace's.

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.Privacy;
using Osysharp.Storage;
using Osysharp.Qualifications;
using Osysharp.Privacy.Qualifications;

[Test(Space)]
[runas(Sam)]
void her_copy_lists_what_she_holds_and_the_text_she_signed() {
  Held();
  var holder = new QualificationsPersonalData();
  var ann = new DataSubject { Account = Ann.Id };
  Assert.Equal(3, holder.Count(ann, CategoryOf("qualifications")));
  var copy = holder.Export(ann, CategoryOf("qualifications"));
  Assert.Contains("LC-118", copy.Json);
  Assert.Contains("I never leave a running machine unattended.", copy.Json);
  Assert.Contains("induction.pdf", copy.Json);
  Assert.Equal(3, copy.Items.Count);
  Assert.Equal(1, holder.Count(new DataSubject { Account = Bo.Id }, CategoryOf("qualifications")));
}
```

From [`tests/credentials.test.osy`](tests/credentials.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.Privacy;
using Osysharp.Storage;
using Osysharp.Qualifications;
using Osysharp.Privacy.Qualifications;

[Test(Space)]
[runas(Sam)]
void an_erasure_takes_her_qualifications_and_files_and_leaves_his() {
  Held();
  var r = FileRequest(RequestKind.Erasure, Ann.Id, "", IdentityCheck.InPerson, "Asked at the front desk.");
  Assert.Equal(3, PlanLine.Single(l => l.Request == r && l.Category == "qualifications").Items);
  ApproveRequest(r);
  Assert.Equal(RequestState.Done, PrivacyRequest.Single(x => x.Id == r.Id).State);
  Assert.Equal(0, QualificationsOf(Ann).Count);
  Assert.Equal(0, QualificationEvidence.Count());
  Assert.True(HasValidQualification(Bo, "laser-induction"));
  Assert.NoTraceOf("LC-118");
}
```

From [`tests/credentials.test.osy`](tests/credentials.test.osy).

## Learn more

- [The Osysharp.Privacy.Qualifications reference](https://osysharp.com/reference/ui/qualifications-kit/)
- [Osysharp.Privacy.Qualifications in the package catalogue](https://osysharp.com/packages/#osysharp-privacy-qualifications)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
