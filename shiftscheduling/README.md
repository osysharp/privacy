# Osysharp.Privacy.ShiftScheduling

Osysharp.Privacy's holder for [Osysharp.ShiftScheduling](https://github.com/osysharp): the shifts a person was on, the time off
they asked for, the hours they said they are not free and the cover they asked for — counted in a privacy request's
plan, in their copy of their data, and erased when they ask.

## Use it

Nothing to write. In an app that uses both Osysharp.Privacy and Osysharp.ShiftScheduling it arrives by itself (`osy lock`
includes it and says so); to keep it out, `without Osysharp.Privacy.ShiftScheduling;` in `app.osy`.

```osy
use Osysharp.Privacy@0;
use Osysharp.ShiftScheduling@0;
```

Its category is `rota` — *Your shifts and time off* — held under the contract, and portable. Erasing frees the person's
places on shifts still to come; the shifts and everybody else's places stay.

## Source

[github.com/osysharp/privacy/shiftscheduling](https://github.com/osysharp/privacy/tree/main/shiftscheduling); its tests are a community kitchen's.

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.Privacy;
using Osysharp.ShiftScheduling;
using Osysharp.Privacy.ShiftScheduling;

[Test(Kitchen)]
[runas(Cat)]
void her_copy_lists_her_shifts_time_off_and_cover_and_nobody_elses() {
  Planned();
  var holder = new ShiftSchedulingPersonalData();
  var ann = new DataSubject { Account = Ann.Id };
  Assert.Equal(4, holder.Count(ann, CategoryOf("rota")));
  var copy = holder.Export(ann, CategoryOf("rota"));
  Assert.Contains("Lunch service", copy.Json);
  Assert.Contains("Hospital appointment", copy.Json);
  Assert.Contains("Car in for repair", copy.Json);
  Assert.Equal(4, copy.Items.Count);
  Assert.Equal(1, holder.Count(new DataSubject { Account = Bo.Id }, CategoryOf("rota")));
}
```

From [`tests/rota.test.osy`](tests/rota.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.Privacy;
using Osysharp.ShiftScheduling;
using Osysharp.Privacy.ShiftScheduling;

[Test(Kitchen)]
[runas(Cat)]
void an_erasure_frees_her_places_and_leaves_his() {
  Planned();
  var r = FileRequest(RequestKind.Erasure, Ann.Id, "", IdentityCheck.InPerson, "Asked at the kitchen.");
  Assert.Equal(4, PlanLine.Single(l => l.Request == r && l.Category == "rota").Items);
  ApproveRequest(r);
  Assert.Equal(RequestState.Done, PrivacyRequest.Single(x => x.Id == r.Id).State);
  Assert.Equal(0, ShiftsOf(Ann).Count);
  Assert.Equal(0, TimeOffRequest.Count(x => x.Worker == Ann));
  Assert.Equal(1, ShiftsOf(Bo).Count);
  Assert.Equal(2, OpenPlaces(Shift.Single(s => s.Title == "Lunch service")));
  Assert.NoTraceOf("Hospital appointment");
}
```

From [`tests/rota.test.osy`](tests/rota.test.osy).

## Learn more

- [The Osysharp.Privacy.ShiftScheduling reference](https://osysharp.com/reference/ui/shift-scheduling-kit/)
- [Osysharp.Privacy.ShiftScheduling in the package catalogue](https://osysharp.com/packages/#osysharp-privacy-shiftscheduling)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
