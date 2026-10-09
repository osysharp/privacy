# Osysharp.Privacy.MailReceiving

**What an app's watched mailboxes hold about a person, as `Osysharp.Privacy`'s holder** — the mail they sent, as it
arrived (`ReceivedMail` and its original), in their copy of their data, and erased when they ask.

```osy
app Desk {
  use Osysharp.Mail.Receiving@0;
  use Osysharp.Privacy@0;
  // Osysharp.Privacy.MailReceiving is not written: it arrives by itself, because the app uses what it completes (`osy lock` says so)
}
```

- One category, `mail.received` (contract, portable), found by the exact sender address (`EraseMailFrom`).
- An adapter, so neither kit forces the other on an app.

Its test app (`tests/app.osy`) and `tests/received.test.osy` show a request reaching received mail.
