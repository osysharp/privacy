# Osysharp.Privacy.Guardianship

**What Osysharp.Guardianship holds about a person, as `Osysharp.Privacy`'s holder** — in their copy of their data, and
erased when they ask.

```osy
app School {
  use Osysharp.Guardianship@0;
  use Osysharp.Privacy@0;
  // Osysharp.Privacy.Guardianship is not written: it arrives by itself, because the app uses what it completes (`osy lock` says so)
}
```

- `guardianship.links` — who looks after the person and whom they look after; either side's erasure removes the link.
- `guardianship.profile` — date of birth and care notes (special-category data).
- `guardianship.consents` — the consents answered for them, with the words answered; an answer a parent gave for their
  child is the child's record and stays when the parent is erased.
- `guardianship.contacts` — their emergency contacts and who may collect them.
- `guardianship.invitations` — invitations to their address, and the ones they sent.
- An adapter, so neither kit forces the other on an app.

Its test app (`tests/app.osy`, a youth choir) and `tests/families.test.osy` show a child's erasure taking everything of his
and nothing of anyone else's, and a parent's ending the link and keeping the child's data.
