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

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.Privacy;
using Osysharp.IncidentReports;
using Osysharp.Privacy.IncidentReports;

[Test(Library)]
[runas(Mo)]
void her_copy_holds_her_injury_and_her_statement_and_his_holds_only_his_line() {
  var holder = new IncidentReportsPersonalData();
  var una = new DataSubject { Account = Una.Id };
  Assert.Equal(1, holder.Count(una, CategoryOf("incidents")));
  var copy = holder.Export(una, CategoryOf("incidents"));
  Assert.Contains("Bruised wrist", copy.Json);
  Assert.Contains("The step was wet from umbrellas.", copy.Json);
  Assert.Equal(1, copy.Items.Count);
  Assert.Equal(1, holder.Count(new DataSubject { Account = Kofi.Id }, CategoryOf("incidents")));
}
```

From [`tests/incidents.test.osy`](tests/incidents.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.Privacy;
using Osysharp.IncidentReports;
using Osysharp.Privacy.IncidentReports;

[Test(Library)]
[runas(Mo)]
void the_category_is_special_and_kept_on_a_legal_obligation() {
  var c = CategoryOf("incidents");
  Assert.True(c.Special);
  Assert.Equal(RetentionBasis.LegalObligation, c.Basis);
  var r = FileRequest(RequestKind.Erasure, Una.Id, "", IdentityCheck.InPerson, "Asked at the desk.");
  Assert.Equal(1, PlanLine.Single(l => l.Request == r && l.Category == "incidents").Items);
}
```

From [`tests/incidents.test.osy`](tests/incidents.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.Privacy;
using Osysharp.IncidentReports;
using Osysharp.Privacy.IncidentReports;

[Test(Library)]
[runas(Mo)]
void erasing_her_blanks_her_lines_and_keeps_the_report_and_his() {
  new IncidentReportsPersonalData().Erase(new DataSubject { Account = Una.Id }, CategoryOf("incidents"));
  var line = IncidentPerson.Single(p => p.Person == Una);
  Assert.Equal("Erased", line.Name);
  Assert.Equal("", line.Injury);
  Assert.Equal("", line.Statement);
  Assert.Equal(1, IncidentReport.Count());
  Assert.Equal("Kofi Mensah", IncidentPerson.Single(p => p.Person == Kofi).Name);
}
```

From [`tests/incidents.test.osy`](tests/incidents.test.osy).

## Learn more

- [The Osysharp.Privacy.IncidentReports reference](https://osysharp.com/reference/ui/incident-reports-kit/)
- [Osysharp.Privacy.IncidentReports in the package catalogue](https://osysharp.com/packages/#osysharp-privacy-incidentreports)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
