# Osysharp.Privacy.OfferSales

Osysharp.Privacy's holder for [Osysharp.OfferSales](https://github.com/osysharp): the purchases a person made or had made
for them — what, when, how it was paid, and what was given back. Counted in a privacy request's plan, in their copy of
their data, and erased once the books no longer need it.

## Use it

Nothing to write. In an app that uses both Osysharp.Privacy and Osysharp.OfferSales it arrives by itself (`osy lock`
includes it and says so); to keep it out, `without Osysharp.Privacy.OfferSales;` in `app.osy`.

```osy
use Osysharp.Privacy@0;
use Osysharp.OfferSales@1;
```

Its category is `purchases` — *What you bought from us* — kept on a legal obligation: restricted for 84 months after a
request unless you confirm your own period, as bookkeeping law sets it where you sell. Portable. Whoever approves privacy
requests needs to refund purchases (`RefundsPurchases`) to erase them.

## Source

[github.com/osysharp/privacy/offersales](https://github.com/osysharp/privacy/tree/main/offersales); its tests are a bike repair workshop's.
