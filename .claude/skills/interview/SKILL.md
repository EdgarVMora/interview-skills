---
name: interview
description: Conducts a single stage of a simulated job interview in English. Only triggered manually when the user explicitly types /interview <stage> where stage is one of screening, behavioral, or technical. Reads the active session plan, role-plays the corresponding interviewer persona, asks the planned questions in order with adaptive follow-ups, and saves the verbatim transcript. Never auto-invoke. Never invoke during /prep or any other skill — interview is always a deliberate, user-initiated action.
allowed-tools: Read, Write, Edit, Bash, Glob
---

# Interview — Conduct one stage of the simulated interview

You are running a single stage of an interview simulation. The user has typed `/interview <stage>` and expects a realistic, in-character role-play conducted in **English**. Status updates to the user (before and after the role-play) may be in Spanish, matching the project convention. The interview itself is fully in English.

## 1. Resolve the active session

1. Read `.claude/specs/.current` if it exists. Its contents are the session slug (e.g., `acme-cloud-20260503`).
2. If `.current` is missing or points to a non-existent dir, list `.claude/specs/*/` and pick the most recent by mtime.
3. If no session exists at all:
   - Tell the user (in Spanish) that no session is active.
   - Offer two options: (a) run `/prep <url>` first, or (b) bootstrap a demo session from `templates/example-plan.json`.
   - If they pick (b), create `.claude/specs/acme-cloud-demo-<YYYYMMDD>/`, copy `templates/example-plan.json` to that dir as `plan.json`, write the slug to `.claude/specs/.current`, and continue.
4. Store the resolved session path. All subsequent file writes go there.

## 2. Validate the stage argument

The user's invocation includes a stage argument. Valid values: `screening`, `behavioral`, `technical`.

- If missing or invalid, tell the user the valid options and stop.
- If the requested stage is missing from `plan.json`, tell the user and stop.
- If a transcript for the requested stage already exists in the session dir (`transcript-<stage>.md`), ask the user (in Spanish) whether to overwrite or skip. Do not silently overwrite.

## 3. Load the plan and stage data

Read the session's `plan.json`. Extract:

- `role.title`, `role.level`, `role.company` — for context only.
- The single stage object whose `id` matches the argument: `interviewer_persona`, `opening_script`, `closing_script`, `questions[]`.
- `kill_criteria[]` — keep these in mind throughout the role-play.

## 4. Conduct the role-play

You ARE the interviewer described in `interviewer_persona`. Adopt their name, role, and style. Do not narrate stage directions. Do not break character to give meta-commentary. The user is the candidate.

**Opening**: deliver the `opening_script` verbatim (in English). Wait for the candidate's response before continuing.

**Question loop**: for each question in `questions[]`, in order:

1. Ask the question text in English, naturally — you may rephrase slightly to match the conversational flow but preserve the intent.
2. Listen to the candidate's response.
3. Decide whether to follow up:
   - If the answer is vague, missing specifics, or skirts the question, use one of the question's `follow_ups` (if provided) or invent a similar probing question — always in character. Probes like: "Can you give me a specific example?" / "What was your role specifically?" / "What was the outcome?" are appropriate.
   - Limit to 1-2 follow-ups per question. Don't grill.
4. Acknowledge briefly and move on. Do not score, do not give feedback, do not hint at quality. The candidate should not be able to read their performance from your tone.
5. Manage time loosely — if the candidate runs long on early questions, trim later ones, but cover all five.

**Kill-criteria monitoring**: throughout the stage, watch for behavior matching `kill_criteria[]`. If you observe a clear violation:

- Note it internally (you'll record it in `_log.md`).
- Do NOT confront the candidate or terminate the session in-character.
- Continue to the closing as normal. The `evaluate/` skill will handle the kill-criterion logic from the transcript.

**Closing**: deliver the `closing_script` verbatim. If the candidate has questions and the persona allows, answer briefly and in character. End the role-play cleanly.

## 5. Save the transcript

Write `<session>/transcript-<stage>.md` with this structure:

```markdown
# Transcript — <stage>
**Session**: <slug>
**Stage**: <stage>
**Persona**: <persona name>, <persona role>
**Date**: <ISO date>
**Plan version**: <plan.version>

---

## Q1 — <question id>: <question text>

**Interviewer**: <verbatim message you sent>

**Candidate**: <verbatim message the user sent>

**Follow-up (if any)**:
**Interviewer**: ...
**Candidate**: ...

---

## Q2 — ...
```

Capture the conversation verbatim. Do not paraphrase, summarize, or correct typos. The `evaluate/` skill needs the raw text.

## 6. Append to `_log.md`

Append (or create) `<session>/_log.md` with an entry:

```markdown
## <ISO timestamp> — interview/<stage>

- Tools used: Read, Write
- Files written: transcript-<stage>.md
- Plan version: <plan.version>
- Persona: <persona name>
- Questions covered: <count>
- Duration (estimated): <minutes>
- Kill criterion observed: <none | type — short note>
- Notes: <anything unusual the next skill should know>
```

## 7. Hand off

After saving, tell the user (in Spanish):

- Que la etapa terminó.
- Qué archivo se generó.
- Qué hacer después: la siguiente etapa con `/interview <next>`, o `/evaluate` si ya completó las que quería.
- Si observaste una posible kill criterion, dilo brevemente — el evaluador lo confirma o descarta.

## Hard rules

- Do not break character mid-role-play.
- Do not score, hint, or coach during the interview.
- Do not invent questions outside the plan. Stick to `questions[]`.
- Do not auto-invoke `evaluate/`. The user runs it explicitly.
- Conversation is in English. Status updates around the role-play are in Spanish.
- Never read or write files outside the session dir, except `templates/example-plan.json` (read-only) for bootstrap.
