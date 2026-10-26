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

## Examples

Taken from this package's own tests, which compile and run on every release, and from its reference page, whose examples are compiled when the site is built.

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.Privacy;
using Osysharp.OfferSales;
using Osysharp.Privacy.OfferSales;

[Test(Workshop)]
[runas(Max)]
void her_copy_lists_what_she_bought_and_an_erasure_takes_it_and_leaves_his() {
  var plan = Offer.Single(o => o.Key == "service");
  SellAtCounter(plan, Ivy, "Cash");
  SellAtCounter(plan, Sol, "Cash");
  var holder = new OfferSalesPersonalData();
  var ivy = new DataSubject { Account = Ivy.Id };
  Assert.Equal(1, holder.Count(ivy, CategoryOf("purchases")));
  var copy = holder.Export(ivy, CategoryOf("purchases"));
  Assert.Contains("Annual service plan · 95.00 GBP · Paid", copy.Items[0].Body);
  Assert.Contains("R-000001", copy.Json);
  Assert.Equal(RetentionBasis.LegalObligation, CategoryOf("purchases").Basis);
  holder.Erase(ivy, CategoryOf("purchases"));
  Assert.Equal(0, holder.Count(ivy, CategoryOf("purchases")));
  Assert.Equal(1, holder.Count(new DataSubject { Account = Sol.Id }, CategoryOf("purchases")));
}
```

From [`tests/purchases.test.osy`](tests/purchases.test.osy).

```osy
app Cafe {
  use Osysharp.Identity@0;
  use Osysharp.Payments@0;
  use Osysharp.Csv@0;
  use Osysharp.OfferSales@1;
  use Osysharp.Storage;
  use Osysharp.Ui;
  use Osysharp.Workflow;
  model "model/**/*.osy";
}

using Osysharp.Identity;
using Osysharp.Payments;
using Osysharp.OfferSales;

[Role] enum CafeRole { Authenticator, Regular, Barista, Owner }
policy ManagesOffers => RoleGrant.Any(g => g.Member == user && g.Role == CafeRole.Owner);
policy SellsAtCounter => RoleGrant.Any(g => g.Member == user && g.Role == CafeRole.Barista);

app.OfferSales = new OfferSalesSetup {
  Providers = [ new ManualPayment { Key = "tab", Label = "Pay at the till" } ],
  CounterMethods = ["Cash", "Card on the terminal"], Currency = "EUR", Seller = "Corner Café",
};

/// A card of coffees — the café's own kind of offer.
entity CoffeeCard : Offer {
  [Min(1)] int Coffees = 10;
  security { allow read when IsAnonymous || IsAuthenticated; allow create, update when ManagesOffers; }
}

/// The stamps a regular holds — handed over when a card is paid, and once only.
entity Stamps {
  [Required("Whose stamps?")] Principal Holder;
  int Left = 0;
  [MaxLength(64)] string Reference = "";
  security { allow read when IsAuthenticated where Holder == user; }
}

[On]
void HandOverStamps(PurchasePaid e) {
  var card = CoffeeCard.FirstOrDefault(c => c.Id == e.Purchase.Offer.Id);
  if (card == null) { return; }
  new Stamps { Holder = e.Purchase.For, Left = card!.Coffees * e.Purchase.Quantity, Reference = e.Purchase.Reference };
}
```

From [the reference page](https://osysharp.com/reference/ui/offer-sales-kit/).

## Learn more

- [The Osysharp.Privacy.OfferSales reference](https://osysharp.com/reference/ui/offer-sales-kit/)
- [Osysharp.Privacy.OfferSales in the package catalogue](https://osysharp.com/packages/#osysharp-privacy-offersales)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
