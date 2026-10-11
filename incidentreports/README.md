# Osysharp.Privacy.IncidentReports

Osysharp.Privacy's holder for [Osysharp.IncidentReports](https://github.com/osysharp): the reports a person filed, every
report naming them — with their injury, the care they were given and their statement — and what they were told about
an incident. Counted in a privacy request's plan, in their copy of their data, and erased from the record when the
period the law keeps it for has ended.

## Use it

Nothing to write. In an app that uses both Osysharp.Privacy and Osysharp.IncidentReports it arrives by itself (`osy
lock` includes it and says so); to keep it out, `without Osysharp.Privacy.IncidentReports;` in `app.osy`.

```osy
use Osysharp.Privacy@0;
use Osysharp.IncidentReports@0;
```

Its category is `incidents` — *Incidents you reported or were part of* — **health data** (special category), kept on a
legal obligation: restricted for 36 months after a request unless you confirm your own period (accident books and
injury logs are kept three years or more in most places). Erasing blanks the person's name, injury, care notes and
statement wherever they are named and removes what they were told; the report itself stays as the organisation's record
of what happened. Whoever approves privacy requests needs to review incidents (`ReviewsIncidents`) to reach the rows.

## Source

[github.com/osysharp/privacy/incidentreports](https://github.com/osysharp/privacy/tree/main/incidentreports); its tests are a public library's.
