# Expected results

For the evaluator only. Never pass this file to the critic.

All fixtures use `rules.example.md` and the LinkedIn message channel (about 90 words). Every person and organisation is invented.

| Fixture | Seeded flaw | Must produce |
|---|---|---|
| F1 | Names the Beacon model to someone at its maker's rival | FIX or REWORK; the Beacon line is named; the rewrite does not contain "Beacon" |
| F2 | Opens with a credentials list to a warm contact | FIX or REWORK; the credentials opener is named; the rewrite opens with the Lisbon dinner |
| F3 | No ask | FIX or REWORK; the missing ask is named; the rewrite contains exactly one ask; WHAT CHANGED ends with "Ask added by critic: confirm before sending." |
| F4 | Three asks | FIX or REWORK; the multiple asks are named; the rewrite contains exactly one ask |
| F5 | Generic flattery hook | FIX or REWORK; the opening line is named; the rewrite opens with something from the recipient notes |
| F6 | About 220 words | FIX or REWORK; length is named; the rewrite is within about 90 words |
| F7 | An em-dash and the banned phrase "pick your brain" | FIX or REWORK; both are named; the rewrite contains neither |
| F8 | "while you are in London" against notes that place the recipient in Singapore | FIX or REWORK; the London line is named; the rewrite does not assume London |
| F9 | None | SEND; the rewrite is identical to the draft; WHAT CHANGED is "Nothing". Must hold in at least 4 of 5 runs. |

Across every fixture:

- "$0 to $8M ARR", "12 markets", "six years", "December" and any other number, title, employer or date in the draft appear unchanged in the rewrite, unless the sentence carrying them was cut and WHAT CHANGED says so.
- No output questions whether a claim is true.
- No rewrite contains an em-dash, an exclamation mark, "pick your brain", "I hope this finds you well" or "synergy".
- Every rewrite is within about 90 words.
- The output is plain text with no code fence, starts with `VERDICT:`, and follows the order VERDICT, RULES, REPLY LIKELIHOOD, WEAKEST LINES, REWRITE, WHAT CHANGED.
- The RULES line reads "sender rules applied".
- No rewrite states an experience, opinion or intention of the sender that the draft or the relationship line does not contain.
- A sentence the critic had no rule-based reason to change appears in the rewrite word for word.
- Where several asks are cut to one (F4), the surviving ask is a question or a short call, not an introduction, referral or document review.
- The same clause is not a weakest line in one fixture and acceptable in F9.
- No output gives a word or character figure, neither a count nor a limit.
