# Osysharp.Privacy.Qualifications

Osysharp.Privacy's holder for [Osysharp.Qualifications](https://github.com/osysharp): the qualifications a person holds, what
they signed and on which text, and the files behind them — counted in a privacy request's plan, in their copy of their
data, and erased when they ask.

## Use it

Nothing to write. In an app that uses both Osysharp.Privacy and Osysharp.Qualifications it arrives by itself (`osy lock`
includes it and says so); to keep it out, `without Osysharp.Privacy.Qualifications;` in `app.osy`.

```osy
use Osysharp.Privacy@0;
use Osysharp.Qualifications@0;
```

Its category is `qualifications` — *Your qualifications and what you signed* — held under the contract, and portable. A signed
waiver an organisation must keep for legal claims is the app's to say: set the category's retention, and a request keeps
it restricted until that ends.

## Source

[github.com/osysharp/privacy/qualifications](https://github.com/osysharp/privacy/tree/main/qualifications); its tests are a makerspace's.
