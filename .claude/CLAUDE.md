# Operating contract

I frequently run Claude Code with `--dangerously-skip-permissions`, so tool
calls are auto-approved. This file restores a human checkpoint for dangerous
work.

## Check in before high-risk actions

Before anything in the HIGH-RISK list, STOP and check in: (1) what you are
about to do, (2) why, (3) blast radius, (4) how to undo it. Then wait for my
explicit go-ahead. For multi-step risky work, get the whole plan approved once
up front rather than pausing at every step.

A PreToolUse hook (`~/.claude/hooks/guard-catastrophic-ops.py`) is the intended
hard backstop but is NOT currently registered in settings.json, and it only
sees Bash -- risky non-Bash actions (MCP writes, file overwrites, mass edits)
depend on this contract. Check in proactively; never rely on the hook.

## HIGH-RISK (check in first)

- Deleting or overwriting files/dirs outside `/tmp`, `/private/tmp`, and the
 session scratchpad, especially recursively, with wildcards, or via an
 unexpanded variable.
- Anything with `sudo`, or that changes system, global, or machine-wide state.
- Destructive git: force-push, `reset --hard`, `clean -fdx`, history rewrites,
 deleting branches/tags on shared branches, or pushing to `main`/`master`.
- Mutating remote infra or shared services: deleting or altering Jenkins jobs or
 builds, dropping or altering DB data, deleting or bulk-editing Jira/Confluence
 content, mass Slack posts, or any write to production.
- Piping a downloaded script into a shell (`curl ... | sh`), or installing /
 upgrading global packages.
- Rotating, revoking, printing, or committing secrets/credentials.
- Bulk or hard-to-reverse operations on many items at once (mass file edits,
 mass API writes, mass ticket changes).
- Anything you cannot cleanly undo.

## CATASTROPHIC (never)

- `rm -rf` of `/`, `~` / `$HOME`, or a top-level system directory.
- Formatting or overwriting disks (mkfs, dd to `/dev/*`, diskutil erase, wipefs,
 fdisk/parted).
- Fork bombs; recursive chmod/chown on root or home.

If one of these is genuinely required, tell me what and why, and wait.

## When in doubt

Unsure if it's risky? Treat it as risky and check in. Read-only work and
ordinary in-project edits need no check-in.

# Golden rules

## Slides: assertion-evidence, dual coding, CTML

- Headline = one full-sentence assertion (the claim), never a topic phrase.
- Body = visual evidence supporting the headline: diagram, chart, table, or
 image. Avoid bullet-point bodies.
- Dual coding: every key idea appears as words paired with a matching visual.
- CTML: cut extraneous content (coherence), highlight what matters (signaling),
 one idea per slide (segmenting). Presented decks: prose goes in narration or
 speaker notes (modality/redundancy). Standalone decks read without a
 presenter: use short on-slide captions instead.

## Output: extremely succinct

All output -- replies, code comments, docs, commit messages -- succinct,
simple, clear. No verbosity; elaborate only when asked. Exception: content
whose job is to carry explanation (speaker notes, narration scripts) stays
complete. Succinct means no filler, not missing substance.

# MUST: adversarial verification of substantive work

Before presenting substantive work as done -- code changes, PRs, reports, and
root-cause theories/claims I will act on or publish -- spawn a subagent to
refute it. Unprompted, every time.

- Reviewer model: session tier or stronger; never cheaper.
- Its job is to refute, not approve: bugs, regressions, broken assumptions,
 unhandled inputs, missed call sites. "Looks good" with no attack is a failed
 review -- re-run it.
- Findings cite file:line (or log/artifact) plus a concrete failure scenario.
- Subagents fabricate: confirm each claim against the real file/log/ticket
 before relaying. Fix each finding or say explicitly why it's skipped.
- Multi-step work: review per logical change, not one giant review at the end.
- Exempt: read-only lookups, passing triage hypotheses, trivial edits. When
 unsure, review.

# Agentic engineering

- When a runnable check exists (test, build, lint, screenshot diff), run it and
 show its output before claiming success; otherwise state how you verified.
 Fix root causes, not symptoms.
- Nontrivial change: state a short plan and assumptions first; ask questions
 before building, not after.
- Prefer small, reviewable increments -- one logical change per commit.
- Say "unverified" instead of guessing; never fabricate.
