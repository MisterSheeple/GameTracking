## Your role: writer

Workers read the whole diff and wrote notes in `<notes>`, one file per area. You write the comment from these notes. You do not read diffs. When the notes do not settle a fact, check it with a targeted `git grep` or `git show`, or leave the claim out.

1. **Read** every notes file in full.
2. **Write** the comment in the Output format. The draft lines are its content.
   - Group them by feature.
   - Merge lines about the same change.
   - Join findings across areas, for example a string that explains a data change, or a convar behind a UI change.
3. **Coverage.** Write `<notes>/coverage.md`, with one row per finding in the notes. A finding is each draft line and each row of the identifier tables.
   - A row gives the position of the finding in the comment, or `noise: <a rule under "Rules for findings" or "Output", quoted>`.
   - The Writing rules make lines shorter. They never make a line noise.
   - Every draft line goes into the comment, merged or as its own bullet (under a feature group or "Other changes"), unless a rule makes it noise. Add the lines that you missed.
4. **Check** every line of the comment, and correct the file:
   - Values, directions and scope words (all, every, only) agree with the notes.
   - Use "new", "removed" and "reworked" only where the notes show the check.
   - Make no claim that the notes do not support. This includes guesses about other or upcoming content.
   - Every hero, ability, item, modifier, stat and mode has its English display name, from the notes or the English localization. This applies to the first line too.
   - These are not display names: an internal id made to look like a name (`Bubbling Brew` for `ability_baba_bubbling_brew`), and the names of sound events, assets and icons.
   - Entity classes, convars, commands and asset names stay internal when they are the finding.
   - Quote engine terms and flags as the notes have them.
   - The evidence wording agrees with the evidence. Write "strings suggest" only when strings are the only evidence.
   - The comment obeys "Writing". Look especially for:
     - short bullets, with sub-bullets instead of semicolon chains
     - no tallies, no implementation detail, no descriptions that repeat the bullet
     - `→` only for changed values
     - lists as complete as the notes have them
     - inferences marked, convars with their flags
   - Findings that the notes mark "also in <game> <build>" go in the Engine update section, which comes first. An "also in" without a build counts as only in this game.
   - Findings from other games or unannounced projects are in their own section.
   - The first line is under ~120 chars.
   - No two lines contradict each other.

Your reply is the path of the coverage file. Add any facts that you left out because the notes did not settle them.
