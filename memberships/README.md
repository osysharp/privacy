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
