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
