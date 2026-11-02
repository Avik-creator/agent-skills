# Sources

## The author's skill
The author pasted `which-skill` in chat on 2026-11-02 and asked for a skill like it "for agent to have properly". This version keeps its scope rules (one skill when one fits, no chain to look thorough, one question when two fit equally), its pick table, its chains, its approval pause and its "doesn't deploy, doesn't edit" limits, rewritten in the library's voice.

Name mapping from the author's set to the library:

| Author's skill | Library skill |
|---|---|
| no-slop | coding |
| plain-paper | design-docs |
| cases-to-run | test-cases, with code tests in testing |
| open-doors | security |
| one-shape | refactor |
| red-pen | `review` |
| the-cause | debug |
| go-or-stop | ship (release check, done gate) plus database for migrations |
| the-map | design-docs (system design, architecture and scale reviews) and observability (measured performance) |
| whats-next | no library skill. It was left out of the table |

## Changed from the original
- **Library skills the original didn't name:** testing, prototype, observability, database, automation, writing-pr and writing-skills were added to the table, with an "alongside" list for go, database, security and observability.
- **Bug chain:** debug now stops for a yes before coding. The reproduction test comes first through testing, and test-cases is used only for a case no automated test can reach. The original went cause, fix, cases, review.
- **New chains:** "Slow" and "Live incident".
- **`review` limit:** checked against review/SKILL.md:52,56. It fixes proven findings in the user's own work and reports only on others' PRs, so "edits nothing until approved" was not written for it.

## Added 2026-11-03
- Rows for `prove` and `deliver`, and a line in the feature chain pointing to `deliver`.
