# Osysharp.Privacy.HumanRequests

What was asked of a person — under their privacy request.

```osy
use Osysharp.Privacy@0;
use Osysharp.HumanRequests@1;     // …and this bridge is there: nothing to wire.

// A request put to somebody is theirs by itself — by their account, or by the address its link went to.
// A request ABOUT somebody, put to somebody else, is theirs when the app says so:
var r = PutRequest(askTheSurveyor);
MarkAbout(r.Id, AskedRowKind(), new DataSubject { Account = holder.Id });
```

## Use case

A marina asks its berth holders which week suits them for lifting their boats out, and asks a surveyor outside —
who has no account and answers through a link — whether a hull is sound. A berth holder asks for a copy of what the
marina holds about them; another asks to be erased; so does the surveyor.

- **Their copy** holds every request put to them, with what they were shown and what they answered, and every
  request the app marked as about them, with its answer.
- **Erased, the request stays and the person goes.** Who it was put to, what they were shown, the facts beside it,
  what it was about, and their answer and comment are blanked; that a request was put, what kind of thing it asked,
  when, and how it ended stay — the asker's own record. `HumanRequestErased` is raised.
- **A process run that told the request is told again without them** — with
  [`Osysharp.HumanRequests.Processes`](../human-requests-processes), each of its lines says only that somebody was
  asked something here who has since asked to be erased.
- **A request still waiting for their answer holds the erasure** until it is answered, called off or runs out.
- **Kept out of use** on request, and **purged** after the app's period (`RequestsPrivacySetup.KeepMonths`, 0: until
  somebody asks), leaving out what a legal hold covers.
- **Nobody erases a request by hand** — not whoever asked, not whoever oversees requests. Only a privacy request's
  own run does (`ErasesRequestRecords`, nobody by default).

## What it adds

| | |
|---|---|
| The holder `RequestsPersonalData`, category `requests.asked` | discovered by `Osysharp.Privacy`; nothing to wire |
| `RequestsOf(subject)` | the requests that are a person's |
| `AskedRowKind()` | what a mark on a request is called |
| `RequestsPrivacySetup.KeepMonths` | how long a closed request keeps who it was put to |

It relies on `EraseFromRequest(request, account, why)` and the event `HumanRequestErased` in
[`Osysharp.HumanRequests`](../human-requests) (1.3.0).

## Its tests

`kits/privacy-human-requests/tests` is the marina.

| | |
|---|---|
| A person's copy: what they were asked and answered, and what was asked about them | `a_persons_copy_holds_what_they_were_asked_and_answered_and_what_was_asked_about_them` |
| An erasure takes the person and leaves the request, and everybody else's | `an_erasure_takes_the_person_and_leaves_the_request_and_everybody_elses` |
| Somebody outside is found by the address the link went to | `somebody_outside_is_found_by_the_address_the_link_went_to` |
| A request still waiting holds the erasure | `a_request_still_waiting_for_the_persons_answer_holds_the_erasure_until_it_is_answered` |
| Nobody erases a request by hand | `nobody_erases_a_request_by_hand` |

```
osy test kits/privacy-human-requests/tests            # 5 tests
```

The recruiting walk in [`Osysharp.HumanRequests.Processes`](../human-requests-processes) erases a candidate who
withdrew, and asserts the request and the run's lines afterwards.

## Source and package

- Package: `Osysharp.Privacy.HumanRequests` — arrives by itself in an app that uses both `Osysharp.Privacy@0` and
  `Osysharp.HumanRequests@1`
- Source: [`kits/privacy-human-requests`](.) — `model/Holder.osy`

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.Organizations;
using Osysharp.Organizations.ServicePlans;
using Osysharp.WorkQueues;
using Osysharp.HumanRequests;
using Osysharp.Privacy;
using Osysharp.Privacy.HumanRequests;
using Marina;

[Test(Harbour)]
void a_persons_copy_holds_what_they_were_asked_and_answered_and_what_was_asked_about_them() {
  var a = EverybodyAsked();
  var done = Ask(Osysharp.Privacy.RequestKind.Access, AccountOf("olav@sailor.example"), "");
  runas (Petra) {
    Assert.Equal("", done.Problem);
    Assert.Equal(2, PlanLine.Single(l => l.Request == done && l.Category == RequestsCategory()).Items);
    var export = ExportOf(done);
    // What he was asked and what he answered…
    Assert.Contains("Which week suits you for lifting Skarv out?", export);
    Assert.Contains("week 43, my brother helps", export);
    // …and what was asked of somebody else about his boat, with the answer.
    Assert.Contains("Is the hull of Skarv sound?", export);
    Assert.Contains("Osmosis blisters below the waterline.", export);
    // Nothing of Ines.
    Assert.DoesNotContain("Tärna", export);
    Assert.DoesNotContain("week 44", export);
  }
}
```

From [`tests/erasure.test.osy`](tests/erasure.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.Organizations;
using Osysharp.Organizations.ServicePlans;
using Osysharp.WorkQueues;
using Osysharp.HumanRequests;
using Osysharp.Privacy;
using Osysharp.Privacy.HumanRequests;
using Marina;

[Test(Harbour)]
void somebody_outside_is_found_by_the_address_the_link_went_to() {
  var a = EverybodyAsked();
  var done = Ask(Osysharp.Privacy.RequestKind.Erasure, null, "kai@surveys.example");
  runas (Petra) { Assert.Equal("", done.Problem); }
  runas (Hilde) {
    var survey = Request(a.Survey);
    Assert.Equal("", survey.OutsideName);
    Assert.Equal("", survey.OutsideEmail);
    Assert.Equal("", survey.AnsweredByName);
    Assert.Equal(RequestStatus.Answered, survey.Status);
    // The berth holders' own requests are untouched.
    Assert.Contains("week 43", Request(a.OlavsWeek).AnswerJson!);
  }
}
```

From [`tests/erasure.test.osy`](tests/erasure.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.Organizations;
using Osysharp.Organizations.ServicePlans;
using Osysharp.WorkQueues;
using Osysharp.HumanRequests;
using Osysharp.Privacy;
using Osysharp.Privacy.HumanRequests;
using Marina;

[Test(Harbour)]
void a_request_still_waiting_for_the_persons_answer_holds_the_erasure_until_it_is_answered() {
  Guid week = Guid.Empty;
  runas (Hilde) { week = AskForAWeek(Olav, "Skarv").Id; }
  var waiting = Ask(Osysharp.Privacy.RequestKind.Erasure, AccountOf("olav@sailor.example"), "");
  runas (Petra) { Assert.NotEqual(RequestState.Done, waiting.State); }
  runas (Hilde) { Assert.Equal("Olav Berg", Request(week).To!.DisplayName); }
}
```

From [`tests/erasure.test.osy`](tests/erasure.test.osy).

## Learn more

- [The Osysharp.Privacy.HumanRequests reference](https://osysharp.com/reference/agent/kit-privacy-human-requests/)
- [Osysharp.Privacy.HumanRequests in the package catalogue](https://osysharp.com/packages/#osysharp-privacy-humanrequests)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
