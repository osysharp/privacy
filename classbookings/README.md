# Osysharp.Privacy.ClassBookings

**The bookings Osysharp.ClassBookings holds about a person, as `Osysharp.Privacy`'s holder** — in their copy of their data,
and erased when they ask.

```osy
app Studio {
  use Osysharp.ClassBookings@0;
  use Osysharp.Privacy@0;
  // Osysharp.Privacy.ClassBookings is not written: it arrives by itself, because the app uses what it completes (`osy lock` says so)
}
```

- One category, `class-bookings.bookings` (contract, portable): every place the person booked or waited for, when they
  were checked in, cancelled (late or not) or marked absent.
- Erasing it is the kit's own `ForgetBookingsOf`, so a place the person still holds is offered to the next person waiting.
- An adapter, so neither kit forces the other on an app.

Its test app (`tests/app.osy`, a yoga studio) and `tests/bookings.test.osy` show a member's copy and her erasure passing her
place on.

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.Privacy;
using Osysharp.ClassBookings;
using Osysharp.Privacy.ClassBookings;

[Test(Studio)]
void her_copy_lists_every_booking_and_says_which_she_cancelled_late() {
  runas (Eva) {
    var holder = new ClassBookingsPersonalData();
    var mia = new DataSubject { Account = User.Single(u => u.DisplayName == "Mia").Id };
    Assert.Equal(2, holder.Count(mia, CategoryOf("class-bookings.bookings")));
    var copy = holder.Export(mia, CategoryOf("class-bookings.bookings"));
    Assert.Contains("Sunrise flow", copy.Json);
    Assert.Contains("Yin evening", copy.Json);
    Assert.Equal(2, copy.Items.Count);
    Assert.Contains("cancelled late", copy.Items[0].Body);
    Assert.DoesNotContain("Noa", copy.Json);
  }
}
```

From [`tests/bookings.test.osy`](tests/bookings.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.Privacy;
using Osysharp.ClassBookings;
using Osysharp.Privacy.ClassBookings;

[Test(Studio)]
[runas(Eva)]
void erasing_her_takes_her_bookings_and_offers_her_place_to_the_next_in_line() {
  var mia = User.Single(u => u.DisplayName == "Mia");
  var r = FileRequest(RequestKind.Erasure, mia.Id, "", IdentityCheck.InPerson, "She asked at the desk; we know her.");
  Assert.Equal(2, PlanLine.Single(l => l.Request == r && l.Category == "class-bookings.bookings").Items);
  ApproveRequest(r);
  Assert.Equal(RequestState.Done, PrivacyRequest.Single(x => x.Id == r.Id).State);
  Assert.Equal(0, Booking.Count(b => b.Person.DisplayName == "Mia"));
  Background.Settle();
  Assert.Equal(BookingState.Offered, Booking.Single(b => b.Person.DisplayName == "Noa").State);
}
```

From [`tests/bookings.test.osy`](tests/bookings.test.osy).

## Learn more

- [Osysharp.Privacy.ClassBookings in the package catalogue](https://osysharp.com/packages/#osysharp-privacy-classbookings)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
