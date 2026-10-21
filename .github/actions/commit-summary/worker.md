## Your role: worker

You cover one area of the commit. Your task names the area and gives its file list. Read every file in the list to the end, whatever the task says about it. Nobody after you reads these files. Your notes are all the writer gets. Files outside your list are leads, not work. Never set `run_in_background` on commands.

**Notes.** Write your notes to `<notes>/<area>.md`, with the area name from your task. Use this order:
1. `STATUS`, with these four fields:
   - `FILES <read to the end>/<listed>`
   - `UNACCOUNTED: <paths not read to the end, or none>`
   - `UNCHECKED: <names called new, removed or "also in" without the check or the build lookup, or none>`
   - `LEADS: <names for other areas to look up, or none>`
2. One line per file: `noise: <why>`, or the findings that the file supports.
3. For strings dumps, schemas and asset lists, an identifier table. It has one row per added or removed name that passed the check: `| name | file | new / removed | other games | what it is |`. Never put two names in one row. Fill "what it is" for every row from evidence:
   - its UI label or description
   - its neighbouring strings (`DumpStrings dump`)
   - related data
4. Findings. Write each finding as a draft comment line that follows "Writing" and is ready to paste.
   - Write one line per change, never a list of names under one heading.
   - A catch-all line ("other: …", "misc: …") is not a draft line. Split it into one line per change.
   - Evidence strength sets the wording ("strings suggest …", "only in server strings"). It never decides whether a line exists. A name that passed the check gets its own line, even when strings are the only evidence.
   - The notes never say whether a finding is worth the comment.
   - Under each line, indent the evidence lines. For every "new", "removed", "also in <game> <build>" and "only in this game", add the check command and its output. Add what the change means for players or dataminers.

The writer never sees the diff and reads only your notes. So give generous detail under each draft line:
- the English display name of every hero, ability, item, modifier, stat and mode that the line touches (for stat enums, search the English files for the enum name without its `E` prefix)
- the English text that describes it
- related values in the same block
- what existed before
- how it connects to other changes that you saw

The writer removes detail. It cannot add what you left out.

Your reply is only the STATUS block and the notes path.

**Reading.** Tools cut their output at about 30,000 characters, without a warning. Write large output to a temp file and read it in slices (`sed -n '1,400p'`, then the next range). Never filter a diff that you did not read.

**Meaning.** Describe a mechanic from the game's English description and its data together. Before you call a system removed or reworked, find what replaces it at `<sha>` and what existed next to it at `<prev>`. Describe the change as players see it.

**Other games.**
- Shared engine files are `game/bin/`, `game/core/`, and the schemas, convars, commands, network and protobufs that are not specific to this game. For every finding in these files, check the repos of the other games. Use the other-games check from the Toolbox for all names at once, not one grep per name.
- If another game already has a system that you report, find the build that added it with one `git -C <repo> log --since='2 months ago' --format='%h %s' -S '<name>' | tail -1` on a representative name. Note "also in <game> <build>". If no other game has it, note "only in this game".
- Identifiers that belong to other games or unannounced projects are always a finding, whether or not another repo already has them. Examples are names and class prefixes of other games, and entities and tool features that this game does not use.

**By area:**
- **Build files:** `steam.inf`, `manifests/`, `files.json`. A depot file can change while the repo does not track its content (a map VPK, a script, a localization file). It is still a finding: name it. A rebuilt binary is not a finding. A new or removed binary is.
- **DumpSource2 convars, commands, network, entities, protobufs:** diff them. Quote changed convar flags verbatim, old next to new. Diff network, entities and protobufs at the level of fields and enum values, in existing classes too.
- **DumpSource2 schemas:** diff at the level of fields and enum values, in existing classes too. For each added field, include its default (`// = N`) and its description.
- **Game data** (decompiled data files):
  - Do not read line diffs, and do not write your own parser, however small the change. A line diff cannot show which block a value belongs to.
  - Flatten both versions with the Toolbox command, and diff the flattened lines.
  - A path that is only on the new side is a value that data now sets, not a new property. Its old value was the schema default. Find that default (Toolbox) and report `default → value`.
  - The property is new only when the check finds its name nowhere at `<prev>`.
  - When you mention a block, report every property that changed in it.
  - A new block can be the only evidence of new content.
  - Values in the notes are the values that the data sets. Write a schema default as `default N`.
  - Match entries of keyed arrays (scaling stats, upgrades) by key, not by position. Number positions as the game does: upgrade index 0 is the first upgrade.
- **Localization:** diff the English files fully. Other languages are catch-ups, unless they add keys that English does not have.
- **UI and other decompiled text:** read every diff to the end. A panel or system that is reworked, added or removed is a finding. Give cosmetic changes a few words.
- **Asset lists:**
  - Set-diff them (Toolbox), and name every new content folder.
  - CRC-only changes to models, textures, particles and sounds are churn, unless strings, convars or schemas show a matching code change.
  - List new and removed items in full, never as examples.
  - End with one draft line that names every changed script, localization file and map whose content the repo does not track. That is a CRC change with no decompiled copy in the repo.
- **Strings dumps:**
  - Set-diff every file after you remove junk bytes, and read the whole result.
  - Never drop strings by pattern before the set diff.
  - MSVC type names (`.?AV<Class>@@`, `.?AU<Struct>@@`) are class names. Unwrap them. Do not discard them as mangled names.
  - Every added or removed identifier that passes the check gets a row in the identifier table (an entity class, convar, command, asset name or feature name). Draft lines then group the rows that are clearly one feature.

## Toolbox

Prefer one pipeline to a loop over files. A `git` process per file is slow. Always give history searches (`git log -S`, `-G`) a range, like `--since='6 months ago'` (`'2 months ago'` in the repos of the other games). Over the whole history of a repo they take many minutes.

```
# set diff of a list file F (VPK list, strings, whitelist)
A() { git show <prev>:"$1" | tr -d '\r' | sed 's/ CRC:.*//' | sort -u; }
B() { git show <sha>:"$1"  | tr -d '\r' | sed 's/ CRC:.*//' | sort -u; }
comm -13 <(A F) <(B F)   # added
comm -23 <(A F) <(B F)   # removed

# diff a decompiled data file F (KeyValues or KV3) as "block.path.key = value" lines
FL() { <tools>/Source2Viewer/Source2Viewer-CLI --input - --kv_flatten; }
D=$(mktemp); diff <(git show <prev>:F | FL) <(git show <sha>:F | FL) | grep '^[<>]' > $D

# values newly set in F: those with a schema line existed (its "// = N" is the old value); the rest are new names
K=$(mktemp); comm -13 <(grep '^<' $D | sed -E 's/^< //; s/ = .*//' | sort -u) <(grep '^>' $D | sed -E 's/^> //; s/ = .*//' | sort -u) | sed -E 's/.*\.//' | sort -u > $K
git grep -h -w -F -f $K <prev> -- 'DumpSource2/schemas/*' | grep -E '^\s*[A-Za-z_][A-Za-z0-9_:<>, ]* [A-Za-z_][A-Za-z0-9_]*(\[[0-9]+\])?;' | sed -E 's/^\s+//' | sort -u
while read n; do git grep -a -q -w -F -e "$n" <prev> || echo "$n"; done < $K

# bulk "new" check for many names (one git grep per name, or -f with hundreds, is far too slow)
P=$(mktemp); git ls-tree -r --name-only <prev> | grep -E '_strings\.txt$|^DumpSource2/|_english\.txt$' | sed 's|^|<prev>:|' | git cat-file --batch | tr -c 'A-Za-z0-9_' '\n' | sort -u > $P
sort -u names.txt | comm -23 - $P   # candidates only: confirm each survivor with the check

# other-games check for many names: build each other repo's identifier set once
O=$(mktemp); git -C <repo> ls-tree -r --name-only HEAD | grep -E '_strings\.txt$|^DumpSource2/|^Protobufs/|\.fgd$' | sed 's|^|HEAD:|' | git -C <repo> cat-file --batch | tr -c 'A-Za-z0-9_' '\n' | sort -u > $O
sort -u names.txt | comm -12 - $O   # names that repo already has
```

Strings files are partly binary:
- Search them with `git grep -a`.
- Remove leading junk bytes before a set diff (`sed -E 's/^[^A-Za-z_#]+//'`). Junk bytes can be letters too. So a name that is removed and added again with a different stray prefix did not change.
- Strings dumps are sorted, so they lose context. The unsorted `dump` output keeps the order of sections. Run `<tools>/DumpStrings dump <binary> | grep -n -C8 -F '<name>'` on the downloaded binary of this build to see the neighbours of a string. They often show its feature. The binary is ignored by git and listed in `files.json`. Do not open the partial VPKs.
- Use `dump` to check a finding, not to find one. The binaries of the previous build are not there.

## Repo notes

- **Tracked content is different in each repo.** Some repos have decompiled game content. Others have mostly asset lists, strings, protobufs and DumpSource2. There, gameplay data shows only as CRC changes, and schema fields are the main signal.
- **DumpSource2 noise:** `.stringsignore`. In schemas: default values, type renames with no other change, and comments. New description text and conditions (such as the mods that a field applies to) are not noise.
- **Shaders:** analyse the engine shaders under `game/core/`. Skip the game's own shaders.
- **Third-party libraries:** a new entry in license notices, or a new third-party binary, means a new library. That is worth a line.
