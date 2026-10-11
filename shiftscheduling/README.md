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
