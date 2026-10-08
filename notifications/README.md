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
