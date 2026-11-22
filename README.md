# HAI loop attacks: the examples table

This repository holds one table, `examples.csv.gz`. It names 24,132 examples of 300 seconds from
the HAI security dataset, each seen through one control loop of a power plant testbed and
labelled `attack` or `normal`. A TimeNet connector, `icsdataset/hai-loop-attacks`, is proposed in
a pull request to [OpenTSLM/TimeNet](https://github.com/OpenTSLM/TimeNet). It downloads this
table, pinned by commit and SHA-256, and builds the series from the HAI data files.

The table holds no series values. It says which HAI file, which window, which loop and which of
its series, the label, the split, the attack interval and a short rule-written text.

## Attribution

The table is derived from the **HAI security datasets**, <https://github.com/icsdataset/hai>,
commit `2a814cebc9a66b06c9e5cd545e2d72e65d383737`, by the HAI authors named in that repository.
HAI is licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). Please
cite HAI as its README asks.

What we changed: we read the attack tables of the HAI manual
(`hai_dataset_technical_details.pdf`), fixed entries where the data files disagree with it, mapped
the column names of each version to the 22.04 names, cut 300-second windows from the data files,
labelled each window once per control loop, drew splits and a training sample with fixed seeds,
and wrote one text per example by rule.

## Licence

The table is licensed under the Creative Commons Attribution-ShareAlike 4.0 International
License, the same licence as HAI. The full text is in [LICENSE](LICENSE). If you share the table
or something built from it, keep the attribution above and use the same licence.

## Files

| File | What it is |
| --- | --- |
| `examples.csv.gz` | the table: gzip-compressed CSV, UTF-8, one header row, 24,132 data rows |
| `SHA256SUMS` | the SHA-256 of `examples.csv.gz` |
| `LICENSE` | CC BY-SA 4.0 |

Check the file:

```bash
shasum -a 256 -c SHA256SUMS
```

## Columns

| Column | Meaning |
| --- | --- |
| `example_id` | `<window_id>/<loop>` |
| `window_id` | `<version>/<file>/<start, YYYYmmddTHHMMSS>` |
| `version`, `file` | the HAI version (20.07, 21.03, 22.04, 23.05) and data file, without extension |
| `split` | `train`, `validation` or `test`, by whole file |
| `window_kind`, `anchor` | `attack` or `normal` window; the attack the window was cut around |
| `scored` | one of the six windows of each attack, picked at random, to score per attack |
| `start` | first second of the window, local time as HAI prints it |
| `loop` | the control loop, such as `P1-PC` (boiler pressure) |
| `label` | `attack` when an attack that the HAI manual assigns to this loop is flagged in the window |
| `attack_ids` | the HAI attack ids of this loop in the window, space-separated |
| `loop_uncertain` | the manual names one loop in the scenario and another in the points |
| `attack_start_s`, `attack_end_s` | first and last flagged second inside the window, or -1 |
| `no_visible_change` | an attack example whose series show neither a level shift nor a swing change |
| `reacting` | a normal example whose series start or stop swinging when a loop of the same process is attacked |
| `train_use` | the class-balanced training sample |
| `tags`, `roles`, `source_tags` | the loop's series: 22.04 names, roles, and column names in that version |
| `answer` | the rule-written text, ending in `Answer: attack` or `Answer: normal` |

## Counts

| Split | attack | normal |
| --- | --- | --- |
| train | 822 | 17,814 |
| validation | 144 | 948 |
| test | 492 | 3,912 |

98 examples are `no_visible_change`, 323 are `reacting` and 3,880 are `train_use`.

## Limits

- The label says which loop the HAI manual says was attacked. It does not check that the attack
  is visible in that loop's series.
- The texts come from simple rules and were read against charts for a few dozen examples only.
- `reacting` is a mark, not a label. Whether a marked loop was pushed by the attack or moved by
  chance is not settled.
- The pipeline that writes this table is Night Shift,
  [vid-t/night-shift](https://github.com/vid-t/night-shift): `uv run hai-build`, then
  `uv run hai-export`. The table is byte-for-byte reproducible from it: every draw uses a fixed
  seed and the gzip header carries no time stamp.
