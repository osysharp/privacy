# Osysharp.Privacy.MailReceiving — changes

## 1.0.0
- Grouped into classes (`osy restructure`). Calls move:
  - `ReceivedMailRow` → `ReceivedMailPersonalData.ReceivedMailRow` (a class)
- Published tools, actions and actors keep their names: what calls them by name is unchanged.
- A project moves when it bumps the kit: `use Osysharp.Privacy.MailReceiving@1;`, then the calls above.
