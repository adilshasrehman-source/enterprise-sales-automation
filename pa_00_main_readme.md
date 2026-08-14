# GTM Outbound AI Automation Suite

**Author:** Adilsha S Rehman
**Stack:** Microsoft Power Automate · AI Builder / OpenAI API · Excel Online · Office 365 Outlook

A three-stage outbound sequence that generates personalised cold emails, follows up twice on a schedule, and keeps the whole conversation in a single inbox thread.

---

## Why it exists

Outbound was eating a large part of my week. Researching each prospect, writing something that wasn't a template, then remembering to follow up twice at the right interval across a few hundred leads. The writing was the visible cost; the sequencing and record-keeping were the ones that quietly consumed more.

This automates the send side end to end. It was built outside my formal role, without engineering support, on the licence tier I had access to — which shaped several of the design decisions below.

---

## Architecture

Three flows, running independently, coordinated through a shared Excel ledger.

```
Flow 1 — First Mail
  Ledger → dedupe check → AI generation → send → log subject, recipient, state

Flow 2 — First Follow-Up (daily schedule)
  Ledger → filter eligible → AI follow-up + summary → find thread → reply → update state

Flow 3 — Second Follow-Up (daily schedule)
  Ledger → filter eligible → AI final message using stored summary → find thread → reply → close lifecycle
```

The flows never talk to each other directly. Everything one needs to know about what another did is written to the ledger — which is the central constraint the whole design works around.

| Flow | Documentation |
|---|---|
| 1 | [First Mail — AI Outbound Automation](./01-first-mail.md) |
| 2 | [First Follow-Up — Threaded AI Outbound](./02-first-follow-up.md) |
| 3 | [Second Follow-Up — Final Threaded Sequence](./03-second-follow-up.md) |

---

## Design decisions

**Send vs. draft.** Early versions wrote to Outlook drafts and posted a Teams approval request before sending. Once the AI output was consistently good enough to send unedited, the review step became the bottleneck rather than the safeguard, so I moved to direct send. These were cold first-touch emails to a curated list — the cost of a mediocre one was low, and time saved was the entire point. For higher-stakes output I would keep the human in the loop; on a later business-case drafting project the output goes to a Gmail draft and is never sent automatically, because that email is intended for a CFO.

**Excel over Dataverse.** Availability and licensing. Excel Online was what I had reliable access to. Dataverse would give proper concurrency handling and relational structure. Excel meant accepting that two flows writing to the same row simultaneously is a real risk at any volume.

**Thread matching by subject and recipient.** Flow 1 logs the AI-generated subject line and the recipient address; later flows search the Sent Items folder on those two fields to locate the message and reply into the thread. It works, but it is not robust — two prospects at the same company could collide, and it depends on the AI generating a distinct subject line each time. Storing the Message ID returned by the send action would be the correct fix.

**Summarise rather than pass full history.** Flow 2 generates a condensed summary of its own message and writes it to the ledger, so Flow 3 has context without carrying the full text of every prior email into the prompt.

---

## Known gaps

**Reply and bounce detection is manual.** Both follow-ups check a `Replied` column before sending, and Flow 1 initialises it to `False`. Nothing in these three flows writes to it — replies and bounce-backs were reviewed and flagged in the ledger by hand between runs. Automating it needs a fourth flow: trigger on new mail, match the sender against the ledger, flip the flag. Bounce-backs would match on NDR sender patterns (`postmaster@`, `mailer-daemon@`) and delivery-status content type. That listener was scoped but not built.

**No output validation.** If the AI returns something empty or malformed, it goes out. There is no check between generation and send.

**Sequential processing.** `Apply to each` runs in order by default. Fine at the volumes I ran; it would need concurrency settings and throughput consideration at scale.

**No concurrency handling on the ledger.** Two flows updating the same row at the same moment would produce a last-write-wins outcome with no warning.

---

## What this taught me

Cross-flow state is the hard part. Three flows running on different schedules, and the only thing holding them together is a shared ledger. Every design decision came back to the same question: what has to be written down so that a flow running tomorrow knows what a flow running yesterday did.
