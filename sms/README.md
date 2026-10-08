# Osysharp.Privacy.Sms

**The texts an app sent a person, as `Osysharp.Privacy`'s holder** — in their copy of their data, and erased when they
ask.

```osy
app Desk {
  use Osysharp.Sms@0;
  use Osysharp.Privacy@0;
  // Osysharp.Privacy.Sms is not written: it arrives by itself, because the app uses what it completes (`osy lock` says so)
}
```

- One category, `sms.sent` (contract), found by the number a request names: `FileRequest(…, phone: "+46…")`, which staff
  check is the person's by calling it. A request a person makes for themselves proves an ADDRESS, so it reaches texts
  only through a desk's conversation with them (`Osysharp.Privacy.Conversations`, by the channel).
- An adapter, so neither kit forces the other on an app.

Its test app (`tests/app.osy`) and `tests/texts.test.osy` show a request reaching sent texts.
