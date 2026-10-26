# Osysharp.Privacy.Notifications

`Osysharp.Privacy`'s holder for `Osysharp.Notifications`: what the app told a person in their bell, and how they asked
to be mailed, under their privacy request — counted in every plan, in their copy of their data, kept restricted when
they ask (Art. 18), erased when they ask.

**Use case.** A teammate leaves and asks to be erased. Their account is found by the request; their notifications and
preferences go with it, and nobody else's. Notifications never COPY another person's words (a ticket's subject, a
customer's name is read from the record when shown), so a customer's erasure needs nothing from here.

```osy
use Osysharp.Privacy@0;
use Osysharp.Notifications@0;
// Osysharp.Privacy.Notifications is not written: it arrives by itself, because the app uses what it completes (`osy lock` says so)

// The privacy desk reads them to answer a request:
partial entity Notification { security { allow read when ManagesPrivacy; } }
partial entity NotificationPreferences { security { allow read when ManagesPrivacy; } }
```

Found only by the ACCOUNT a request proved — an address alone reaches none. Category `notifications.told` (Contract,
portable).

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.Notifications;
using Osysharp.Privacy;
using Osysharp.Privacy.Notifications;
using PrivacyNotificationsKitTests;

[Test(Team)]
void her_copy_lists_what_she_was_told_and_how_she_asked_to_be_mailed() {
  TellTheTeam();
  runas (Ada) {
    var section = new NotificationsPersonalData().Export(new DataSubject { Account = MiasAccount() }, CategoryOf("notifications.told"));
    Assert.Equal(3, section.Items.Count);
    Assert.Contains("A task is yours now.", section.Json);
    Assert.Contains("A daily digest of what I have not seen", section.Json);
  }
}
```

From [`tests/told.test.osy`](tests/told.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.Notifications;
using Osysharp.Privacy;
using Osysharp.Privacy.Notifications;
using PrivacyNotificationsKitTests;

[Test(Team)]
void an_erasure_of_her_account_takes_her_notifications_and_preferences_and_no_others() {
  TellTheTeam();
  runas (Ada) {
    var r = FileRequest(RequestKind.Erasure, MiasAccount(), "", IdentityCheck.InPerson, "She asked at the front desk with her ID card.");
    Assert.Equal(3, PlanLine.Single(l => l.Request == r && l.Category == "notifications.told").Items);
    ApproveRequest(r);
    Assert.Equal(RequestState.Done, PrivacyRequest.Single(x => x.Id == r.Id).State);
  }
  runas (Mia) {
    Assert.Equal(0, Notification.Count());
    Assert.Equal(0, NotificationPreferences.Count());
  }
  runas (Leo) { Assert.Equal(1, Notification.Count()); }
}
```

From [`tests/told.test.osy`](tests/told.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.Notifications;
using Osysharp.Privacy;
using Osysharp.Privacy.Notifications;
using PrivacyNotificationsKitTests;

[Test(Team)]
void a_request_that_proved_only_an_address_reaches_no_notifications() {
  TellTheTeam();
  runas (Ada) {
    Assert.Equal(0, new NotificationsPersonalData().Count(new DataSubject { Emails = ["mia@team.test"] }, CategoryOf("notifications.told")));
  }
}
```

From [`tests/told.test.osy`](tests/told.test.osy).

## Learn more

- [Osysharp.Privacy.Notifications in the package catalogue](https://osysharp.com/packages/#osysharp-privacy-notifications)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
