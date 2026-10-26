# Osysharp.Privacy.KnowledgeBase

Who wrote and who reviewed your help centre's articles, in their privacy requests. Add it beside
`Osysharp.KnowledgeBase` and `Osysharp.Privacy`, and the articles and versions a person wrote or published, and the review
steps they took, are part of what they can see, have kept out of use, or have erased. Nothing is wired: the holder is
found by itself.

```osy
use Osysharp.KnowledgeBase@1;
use Osysharp.Privacy@0;
// Osysharp.Privacy.KnowledgeBase is not written: it arrives by itself, because the app uses what it completes (`osy lock` says so)
```

| | |
|---|---|
| **Found by** | the person's account: only people with an account write or review |
| **Their copy** | each version they wrote or published, and each step they took with what they said, when |
| **Kept out of use** | their review steps and notes leave every ordinary read; the articles stay readable |
| **Erased** | their name comes off every article, version and step, and their review notes are erased. The articles stay, because what readers are told belongs to the organisation |

The category is "The articles you wrote or reviewed, and what you said in reviews", kept on legitimate interest.

## Its tests

Its test app is a museum's visitor guide: a guide who has left asks for her copy, and then to be erased.

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.KnowledgeBase;
using Osysharp.Privacy;
using Osysharp.Privacy.KnowledgeBase;
using PrivacyKnowledgeKitTests;

[Test(Museum)]
void her_copy_lists_what_she_wrote_and_said() {
  Written();
  runas (Ada) {
    var gil = new DataSubject { Account = AccountOf("gil@museum.test") };
    var section = new KnowledgePersonalData().Export(gil, CategoryOf("knowledge.authorship"));
    Assert.Contains("Free entry on Sundays", section.Json);
    Assert.Contains("Checked with the front desk.", section.Json);
    Assert.DoesNotContain("Group tickets", section.Json);
    Assert.Equal(3, new KnowledgePersonalData().Count(gil, CategoryOf("knowledge.authorship")));
    // Nobody with only an address wrote anything.
    Assert.Equal(0, new KnowledgePersonalData().Count(new DataSubject { Emails = ["gil@museum.test"] }, CategoryOf("knowledge.authorship")));
  }
}
```

From [`tests/authorship.test.osy`](tests/authorship.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.KnowledgeBase;
using Osysharp.Privacy;
using Osysharp.Privacy.KnowledgeBase;
using PrivacyKnowledgeKitTests;

[Test(Museum)]
void an_erasure_takes_her_name_and_notes_and_leaves_the_articles() {
  Written();
  runas (Ada) {
    var r = FileRequest(RequestKind.Erasure, AccountOf("gil@museum.test"), "", IdentityCheck.InPerson, "Gil left and asked in person.");
    Assert.Equal(3, PlanLine.Single(l => l.Request == r && l.Category == "knowledge.authorship").Items);
    ApproveRequest(r);
    Assert.Equal(RequestState.Done, PrivacyRequest.Single(x => x.Id == r.Id).State);
  }
  runas (Ada) {
    var article = KnowledgeArticle.Single(a => a.Slug == "free-entry-on-sundays");
    Assert.Null(article.Author);
    var v = KnowledgeRevision.Single(x => x.Title == "Free entry on Sundays");
    Assert.Null(v.Author);
    Assert.Equal(KnowledgeStage.Published, v.Stage);
    var sent = KnowledgeStep.Single(x => x.Article.Id == article.Id && x.Kind == KnowledgeStepKind.Submit);
    Assert.Null(sent.By);
    Assert.Null(sent.Note);
    // Ada's own step and note on Sol's article stay hers.
    Assert.Equal("Say how far ahead, Sol.", KnowledgeStep.Single(x => x.Kind == KnowledgeStepKind.SendBack).Note);
  }
  // Readers still read the article.
  Assert.True(KnowledgeArticleView("free-entry-on-sundays").Found);
}
```

From [`tests/authorship.test.osy`](tests/authorship.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.KnowledgeBase;
using Osysharp.Privacy;
using Osysharp.Privacy.KnowledgeBase;
using PrivacyKnowledgeKitTests;

[Test(Museum)]
void a_restriction_keeps_her_review_notes_out_of_use_and_the_article_readable() {
  Written();
  runas (Ada) {
    var r = FileRequest(RequestKind.Restriction, AccountOf("ada@museum.test"), "", IdentityCheck.InPerson, "She asked us to stop using her data.");
    ApproveRequest(r);
  }
  runas (Sol) { Assert.Equal(0, KnowledgeStep.Count(x => x.Kind == KnowledgeStepKind.SendBack)); }
  Assert.True(KnowledgeArticleView("free-entry-on-sundays").Found);
}
```

From [`tests/authorship.test.osy`](tests/authorship.test.osy).

## Learn more

- [The Osysharp.Privacy.KnowledgeBase reference](https://osysharp.com/reference/ui/knowledge-kit/)
- [Osysharp.Privacy.KnowledgeBase in the package catalogue](https://osysharp.com/packages/#osysharp-privacy-knowledgebase)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
