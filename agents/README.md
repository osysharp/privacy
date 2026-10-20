# Osysharp.Privacy.Agents

**What your AI agents did for a person, and about a person, under that person's privacy request.** Add it beside
`Osysharp.Agents` and `Osysharp.Privacy` and every piece of agent work is part of what a person can see, have kept
out of use, or have erased — and none of it is kept past its time. After an erasure the record of what the agent did
still proves it was not changed.

```osy
use Osysharp.Agents@1;
use Osysharp.Privacy@0;
// Osysharp.Privacy.Agents is not written: it arrives by itself, because the app uses what it completes (`osy lock` says so)
```

Nothing is wired: the holder is found by itself.

## Use case

A library's desk is an AI agent. Mia, a member, asks it to renew a book: the agent looks her loans up, renews one and
tells her. A librarian has it draft a reminder about Leo, who borrowed a book and only ever left an address.

Mia asks for a copy of what the library holds about her. It contains what she asked, what each tool was given and
answered, and what the desk told her — and nothing of Leo's or anybody else's.

Then she asks to be erased. What the desk was shown and answered for her is gone, for the auditor too. She is taken
off the work: it is nobody's, about nothing, and each line of its record reads `Erased.` What stays is that the desk
renewed a loan — the steps in order, the model, the tools, what it cost — and the check that none of it was changed:

```osy
var check = VerifyTrace(receipt);
// check.Intact == true · check.Lines == 9 · check.Erased == 9

var lines = TraceOf(receipt);
lines[1].CostMicros = 1;                 // somebody turns a cost down afterwards
CheckChain(lines).Problem;               // "Line 2 is not as it was written."
```

## Whose data a piece of work is

| | |
|---|---|
| **Done FOR a person** | They asked, or somebody asked on their behalf (`AgentTask.Requester`, `StartedBy`). Found by the **account** a request proved. Nothing to write. |
| **ABOUT a person** | Only your app knows. Say it once, where the work is given: `MarkAgentWorkAbout(outcome.Task, new DataSubject { Emails = [loan.BorrowerEmail] });` — by an account, an address or a number. The note keeps a fingerprint of the address, never the address, and only the privacy desk reads it back. |

```osy
public AgentOutcome<Reminder> RemindAbout(Loan loan) {
  var outcome = await Agents.Of<DeskAgent>().Do(new DraftReminder(), loan, about: "Reminder to " + loan.BorrowerName);
  MarkAgentWorkAbout(outcome.Task, new DataSubject { Emails = [loan.BorrowerEmail] });
  return outcome;
}
```

A piece of work is found whole: the top piece and everything it handed on share one record and one chain, and are
copied, kept out of use, erased and deleted together. Work nobody marked is still found as the work of whoever gave
it — never out of every request's reach — but not by the address of the person it was about.

## What it holds, and for how long

| Category | What | Kept |
|---|---|---|
| `agents.trace` | What the work **said**: what the agent was shown, what each tool was given and answered, what the model said, what it asked when it asked for help, and the result it handed back. | **1 month** after the work ended |
| `agents.record` | **That it happened**: which agent and skill, who it was for, what it was about, each step in a sentence with its model, tool and cost, how it ended — and the hash chain. | **7 months** after the work ended |
| `agents.corrections` | What somebody wrote to **correct** work done for or about a person, and the instructions an agent is given for that person's work. | **7 months** after it was written, and never longer than the record of the work it corrects |
| `agents.teaching` | **That a person taught**: the corrections they wrote, the instructions they proposed, activated, retired or rolled back. | while the agent is told it |
| `agents.oversight` | **That a person oversaw the agents**: the standing rules they gave or took back, pausing, resuming and sponsoring an agent, the questions an agent put to them and their answers, the proposals sent to them or decided by them. Erased, they are taken off each; it waits while a question or proposal waits on them, or while they sponsor an agent. | while what was decided is on record |

All are held on legitimate interest, so a person's erasure erases them unless somebody records compelling grounds.
A standing rule for a person's work ("from now on, alone") is part of `agents.corrections`: erased, it is taken back
and says nobody's work. With `Osysharp.Agents.HumanRequests`, who handed an inbox item to an agent is
[`Osysharp.Privacy.Agents.HumanRequests`](../privacy-agents-human-requests)' `agents.handovers`.

The periods are your app's defaults — `app.AgentsPrivacy = new AgentsPrivacySetup { KeepContentMonths = 3,
KeepRecordMonths = 24 };` — and the tenant's on the retention page. Seven months is six and a margin: where the EU AI
Act applies to you, Art. 19 and Art. 26(6) ask for the logs of a high-risk system to be kept at least six months.
Whether it applies is yours to establish; the record's period has no upper limit so that you can keep it longer.

## What a request does

| Request | What happens |
|---|---|
| **A copy** (access) | Every step of their work with its content, in order; each piece of work with its agent, skill, what it was about, how it ended and what it cost; whether it was for them, given by them, or about them. |
| **Erasure** | `agents.trace`: the content of every line, the help asked and the result are emptied. `agents.record`: they are taken off — who it was for, what it was about and how it ended in words are blanked, and every line's sentence becomes `Erased.` The receipt, each step's kind, model, tool and cost, and the chain stay. |
| **Restriction** | Their work leaves every ordinary read — theirs, a supervisor's, an auditor's — until it is lifted. |
| **What an agent was taught** | A copy carries the corrections of their work and the instructions for it. An erasure takes each correction's words and erases an instruction that is for them — retired first: the agent's next instruction set, made by nobody. A restriction withholds it. A member of staff who asks is taken off what they taught; what was decided stays, and the agent goes on being told it. `MarkAgentInstructionAbout(instruction, subject)` says an instruction names somebody the agent does not work for. |
| **Work still running** | The request waits ("an AI agent is still working on something for you or about you") and is asked again each day. |
| **A legal hold** | A held person's work is neither erased nor purged while the hold stands — found by their account, or by the address the work was marked with. |

Every day, for everybody and not only for people who asked: content older than its period is emptied, and work older
than the record's period is deleted whole — receipts, lines, requests for help, and the notes of whose it was.

## Who erases

Nobody, by hand. `Osysharp.Agents`' erasure functions ask the app's `ErasesAgentRecords` policy, which is nobody's
until your app says otherwise — and it need not: a privacy request's own run erases as the app itself. In the test
app nobody is named, and Ada, who manages privacy *and* audits the desk, is refused when she tries.

## Its tests

`kits/privacy-agents/tests` is that library — a desk agent, two members with accounts, a borrower with only an
address, a librarian in charge of the desk and Ada at the privacy desk. One scripted model; none is ever called.

| | |
|---|---|
| The chain verifies after a person is erased, and a line changed afterwards is still found | `the_chain_verifies_after_a_person_is_erased` |
| Her copy holds what the desk was shown and answered for her, and nobody else's | `her_copy_holds_what_the_desk_was_shown_and_answered_for_her_and_nobody_elses` |
| A person with no account is found by the address the app marked | `a_borrower_without_an_account_is_found_by_the_address_the_library_marked` |
| After an erasure nothing of hers is left in anything the desk recorded, on the meter, or in a second copy | `after_an_erasure_nothing_of_hers_is_left_in_anything_the_desk_recorded` |
| Erasing one person leaves everybody else's work word for word | `erasing_the_borrower_takes_the_reminder_about_him_and_leaves_the_others` |
| Content goes after a month and the record after seven, for everybody | `what_the_work_said_goes_after_a_month_and_the_record_after_seven` |
| A legal hold keeps a person's work through the purge | `a_legal_hold_keeps_a_persons_agent_work_through_the_purge` |
| A restriction takes it out of use for everybody | `a_restriction_takes_her_agent_work_out_of_use_for_everybody` |
| An erasure waits while the agent is still working | `an_erasure_waits_while_the_desk_is_still_working_for_her` |
| Nobody erases a record by hand; a request does | `nobody_erases_an_agents_record_by_hand_and_a_privacy_request_does` |
| Whose data a piece of work is, is said through the kit and read only by the privacy desk | `whose_data_a_piece_of_work_is_is_said_through_the_kit_and_read_only_by_the_privacy_desk` |
| What others may read is the same before and after somebody is erased | `what_others_read_of_the_desks_work_is_the_same_before_and_after_somebody_is_erased` |
| Her copy holds what was corrected in her work and what the desk is told for her | `her_copy_holds_what_was_corrected_in_her_work_and_what_the_desk_is_told_for_her` |
| Erasing her takes what her corrections said and the instruction for her, and nobody else's; the record still verifies | `erasing_her_takes_what_her_corrections_said_and_the_instruction_for_her_and_nobody_elses` |
| A librarian who leaves is taken off what he taught, and the desk is still told it | `a_librarian_who_leaves_is_taken_off_what_he_taught_and_the_desk_is_still_told_it` |
| Kept out of use, an instruction for her is no longer sent | `kept_out_of_use_an_instruction_for_her_is_no_longer_sent_and_a_correction_no_longer_read` |
| What a correction said goes at the end of its time | `what_a_correction_said_goes_at_the_end_of_its_time_for_everybody` |

```
osy test kits/privacy-agents/tests
```

## Source and package

- Package: `Osysharp.Privacy.Agents` — it completes `Osysharp.Privacy` and `Osysharp.Agents`, and arrives by itself.
- Source: [`kits/privacy-agents`](.) — `model/Holder.osy` (the holder, `MarkAgentWorkAbout`, `AgentWorkOf`, the copy).
- Builds on [`Osysharp.Agents`](../agents) (`EraseAgentWorkContent`, `AnonymiseAgentWork`, `ForgetAgentWork`,
  `VerifyTrace`) and [`Osysharp.Privacy`](../privacy) (the holder contract, `MarkAbout`, requests, the purge, holds).
- The model calls an agent made are on `Osysharp.Llm`'s meter, which [`Osysharp.Privacy.Llm`](../privacy-llm) holds.
