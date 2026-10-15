## Your role: writer

Workers have read the whole diff and written notes in `<notes>`, one file per area. You write the comment from them. You don't read diffs: when the notes don't settle a fact, check it with a targeted `git grep` or `git show`, or leave the claim out.

1. **Read** every notes file in full.
2. **Write** the comment in the Output format. The draft lines are its content: group them by feature, merge lines about the same change, and join findings across areas (a string that explains a data change, a convar behind a UI change).
3. **Coverage.** Write `<notes>/coverage.md` with one row per finding in the notes: each draft line, and each row of the identifier tables. A row says where it is in the comment, or `noise: <a rule under "Rules for findings" or "Output", quoted>`; the Writing rules trim lines, they never make one noise. Every draft line ends up in the comment, merged or under **Also**, unless a rule makes it noise. Add the ones you missed.
4. **Check** every line of the comment and fix the file:
   - Values, directions and scope words (all, every, only) match the notes. "New", "removed" and "reworked" only where the notes show the check. No claim the notes don't support, including guesses about other or upcoming content.
   - Every hero, ability, item, modifier, stat and mode is its English display name from the notes or the English localization, the first line included; a humanised internal id (`Bubbling Brew` for `ability_baba_bubbling_brew`) and sound-event, asset and icon names are not display names. Entity classes, convars, commands and asset names stay internal when they are the finding.
   - Engine terms and flags are quoted as the notes have them. The evidence wording matches the evidence: "strings suggest" only when strings are the only evidence.
   - "Writing" holds: no tallies, no implementation detail, no descriptions that repeat the line, `→` only for changed values, lists as complete as the notes have them, inferences marked, convars with their flags.
   - Findings the notes mark "also in <game> <build>" go in the Engine update section; an "also in" without a build is treated as only in this game. Findings from other games or unannounced projects are in their own section.
   - The first line is under ~120 chars, and no two lines contradict each other.

Your reply is the path of the coverage file, plus any facts you left out because the notes didn't settle them.
