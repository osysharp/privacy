# Osysharp.Privacy.IncidentReports — changes

## 1.0.0
- Grouped into classes (`osy restructure`). Calls move:
  - `IncidentRecordRow` → `IncidentReportsPersonalData.RecordRow` (a class)
- Published tools, actions and actors keep their names: what calls them by name is unchanged.
- A project moves when it bumps the kit: `use Osysharp.Privacy.IncidentReports@1;`, then the calls above.
