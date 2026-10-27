# Dragon Warrior IV — Smart Context

A BPS patch for the USA NES game Dragon Warrior IV. On the field, A tries Talk, then Door, then Search. Select opens the command menu.

This repository does not include the game. You supply your own unmodified USA ROM. The patch file is 542 bytes.

This is a fan project. It is not made by or affiliated with Enix, Square, or Nintendo.

Made with Grok by xAI.

The project is [MIT licensed](LICENSE).

**[Download the patch](https://github.com/dunjeon/DragonWarriorIV-SmartContext/releases/latest)**

Notes: [dragon-warrior-iv-rom-hack-notes.docx](dragon-warrior-iv-rom-hack-notes.docx)

Page: [dunjeon.github.io/DragonWarriorIV-SmartContext](https://dunjeon.github.io/DragonWarriorIV-SmartContext/)

## What A does

1. Talk. It looks one tile ahead. If that tile is empty, it looks two tiles ahead, which is how shop counters are set up. The closer person wins.
2. Door, if nobody was there and the tile one step ahead is a door. The real Door command runs, including a locked door. Search does not also run on that press.
3. Search, if it was not a door. Search may still say that nothing was found.

Select opens the old command menu: Talk, Spell, Status, Item, Equip, Tactics, Door, and Search. Release Select after you choose or cancel.

## Apply

The ROM must be the unmodified USA file.

| | |
| --- | --- |
| File | `Dragon Warrior IV (USA).nes` |
| Size | 524,304 bytes |
| CRC32 | `F6B24806` |

Use [Floating IPS](https://github.com/Alcaro/Flips/releases). Apply `dragon-warrior-iv-usa-smart-context.bps` and save a new file. Leave the original ROM as it is. If the CRC does not match, the patcher should refuse the file.

Mesen ties the battery save to the ROM file name. Save the patched ROM under a new name if you want a separate save. Opening the original file name uses the original save.

## Also in this patch

The message window clears when you confirm the continue mark, and when the window fills. Older lines can remain after a yes or no until the window fills or you confirm the continue mark. That behavior is aimed at a screen reader.

The Tactics command can show up before Chapter 5. Choosing a tactic does not enter fight commands for you.

## Checksums

| File | CRC32 |
| --- | --- |
| Unmodified USA ROM | `F6B24806` |
| Patched ROM | `D22F542B` |
| This BPS | `57AF999D` |
