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
