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

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.Privacy;
using Osysharp.Guardianship;
using Osysharp.Privacy.Guardianship;

[Test(Choir)]
void the_childs_data_is_counted_in_every_category() {
  runas (Eva) {
    var holder = new GuardianshipPersonalData();
    var max = Subject("Max");
    Assert.Equal(1, holder.Count(max, CategoryOf("guardianship.links")));
    Assert.Equal(1, holder.Count(max, CategoryOf("guardianship.profile")));
    Assert.Equal(1, holder.Count(max, CategoryOf("guardianship.consents")));
    Assert.Equal(1, holder.Count(max, CategoryOf("guardianship.contacts")));
    Assert.Equal(1, holder.Count(max, CategoryOf("guardianship.invitations")));
    Assert.True(CategoryOf("guardianship.profile").Special);
  }
}
```

From [`tests/families.test.osy`](tests/families.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.Privacy;
using Osysharp.Guardianship;
using Osysharp.Privacy.Guardianship;

[Test(Choir)]
void the_parents_copy_names_the_child_she_looks_after_and_the_invitation_she_sent() {
  runas (Eva) {
    var holder = new GuardianshipPersonalData();
    var pia = Subject("Pia");
    var links = holder.Export(pia, CategoryOf("guardianship.links"));
    Assert.Contains("Max", links.Json);
    Assert.Equal("Someone you look after", links.Items[0].Title);
    Assert.Contains("ola@choir.test", holder.Export(pia, CategoryOf("guardianship.invitations")).Json);
    Assert.Equal(0, holder.Count(pia, CategoryOf("guardianship.profile")));
  }
}
```

From [`tests/families.test.osy`](tests/families.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.Privacy;
using Osysharp.Guardianship;
using Osysharp.Privacy.Guardianship;

[Test(Choir)]
void an_invitation_is_found_by_the_address_it_went_to() {
  runas (Eva) {
    var holder = new GuardianshipPersonalData();
    var byAddress = new DataSubject { Emails = ["Ola@Choir.test"] };
    Assert.Equal(1, holder.Count(byAddress, CategoryOf("guardianship.invitations")));
    Assert.Contains("Max", holder.Export(byAddress, CategoryOf("guardianship.invitations")).Json);
  }
}
```

From [`tests/families.test.osy`](tests/families.test.osy).

## Learn more

- [Osysharp.Privacy.Guardianship in the package catalogue](https://osysharp.com/packages/#osysharp-privacy-guardianship)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
