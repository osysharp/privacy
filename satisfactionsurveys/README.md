# Osysharp.Privacy.SatisfactionSurveys

What a person answered when you asked how it went, in their privacy requests. Add it beside `Osysharp.SatisfactionSurveys` and
`Osysharp.Privacy` and a person's questions and answers are part of what they can see, take with them, have kept out
of use, or have erased. Nothing is wired: the holder is found by itself.

```osy
use Osysharp.SatisfactionSurveys@0;
use Osysharp.Privacy@0;
// Osysharp.Privacy.SatisfactionSurveys is not written: it arrives by itself, because the app uses what it completes (`osy lock` says so)

// whoever answers privacy requests reads the questions to count, copy and erase them
partial entity FeedbackAsk { security { allow read when ManagesPrivacy; } }
```

| | |
|---|---|
| **Found by** | the account a question was asked of, or an address the person's request proved (the question was mailed there) |
| **Their copy** | each question, what they answered and what they wrote, when |
| **Kept out of use** | a restricted person's answers leave the results and every ordinary read |
| **Erased** | their questions and answers are deleted; everybody else's stay |

The category is "What you told us when we asked how it went", kept on consent, and portable.

## Its tests

Its test app runs a ferry line's passengers through a copy, a restriction and an erasure. The feedback kit itself is
`osy docs ui-feedback-kit`.

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.SatisfactionSurveys;
using Osysharp.Privacy;
using Osysharp.Privacy.SatisfactionSurveys;
using PrivacyFeedbackKitTests;

[Test(Line)]
void her_copy_lists_what_she_answered() {
  AskedAndAnswered();
  runas (Ada) {
    var section = new FeedbackPersonalData().Export(new DataSubject { Account = MiasAccount() }, CategoryOf("feedback.answers"));
    Assert.Equal(1, section.Items.Count);
    Assert.Contains("Calm sea, kind crew.", section.Json);
    Assert.DoesNotContain("café", section.Json);
  }
}
```

From [`tests/answers.test.osy`](tests/answers.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.SatisfactionSurveys;
using Osysharp.Privacy;
using Osysharp.Privacy.SatisfactionSurveys;
using PrivacyFeedbackKitTests;

[Test(Line)]
void a_request_that_proved_an_address_reaches_the_answers_mailed_there() {
  AskedAndAnswered();
  runas (Ada) {
    Assert.Equal(1, new FeedbackPersonalData().Count(new DataSubject { Emails = ["leo@sea.test"] }, CategoryOf("feedback.answers")));
    Assert.Equal(0, new FeedbackPersonalData().Count(new DataSubject { Emails = ["nobody@sea.test"] }, CategoryOf("feedback.answers")));
  }
}
```

From [`tests/answers.test.osy`](tests/answers.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.SatisfactionSurveys;
using Osysharp.Privacy;
using Osysharp.Privacy.SatisfactionSurveys;
using PrivacyFeedbackKitTests;

[Test(Line)]
void an_erasure_of_her_account_takes_her_answers_and_no_others() {
  AskedAndAnswered();
  runas (Ada) {
    var r = FileRequest(RequestKind.Erasure, MiasAccount(), "", IdentityCheck.InPerson, "She asked at the ticket office with her ID card.");
    Assert.Equal(1, PlanLine.Single(l => l.Request == r && l.Category == "feedback.answers").Items);
    ApproveRequest(r);
    Assert.Equal(RequestState.Done, PrivacyRequest.Single(x => x.Id == r.Id).State);
  }
  runas (Ola) {
    Assert.Equal(1, FeedbackAsk.Count());
    Assert.Equal("leo@sea.test", FeedbackAsk.Single().Address);
    Assert.Equal(1, FeedbackReply.Count());
  }
}
```

From [`tests/answers.test.osy`](tests/answers.test.osy).

## Learn more

- [The Osysharp.Privacy.SatisfactionSurveys reference](https://osysharp.com/reference/ui/feedback-kit/)
- [Osysharp.Privacy.SatisfactionSurveys in the package catalogue](https://osysharp.com/packages/#osysharp-privacy-satisfactionsurveys)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
