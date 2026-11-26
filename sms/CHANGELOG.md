# Osysharp.Privacy.Sms — changes

## 1.0.0
- Grouped into classes (`osy restructure`). Calls move:
  - `SentTextRow` → `SmsPersonalData.SentTextRow` (a class)
- Published tools, actions and actors keep their names: what calls them by name is unchanged.
- A project moves when it bumps the kit: `use Osysharp.Privacy.Sms@1;`, then the calls above.
