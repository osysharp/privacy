# Osysharp.Privacy.PrepaidCredits

Osysharp.Privacy's holder for [Osysharp.PrepaidCredits](https://github.com/osysharp): a person's packs, what they used
them for and every change to them — counted in a privacy request's plan, in their copy of their data, and erased when
they ask.

## Use it

Nothing to write. In an app that uses both Osysharp.Privacy and Osysharp.PrepaidCredits it arrives by itself
(`osy lock` includes it and says so); to keep it out, `without Osysharp.Privacy.PrepaidCredits;` in `app.osy`.

```osy
use Osysharp.Privacy@0;
use Osysharp.PrepaidCredits@0;
```

Its category is `credits.ledger` — *Your prepaid credits and what you used them for* — held under the contract, and
portable. Erasing takes the person's credits with it: what they paid for and did not use is settled with them in
money first; the sale itself stays in the books.

## Source

[github.com/osysharp/privacy/prepaidcredits](https://github.com/osysharp/privacy/tree/main/prepaidcredits); its tests are a pottery studio's.
