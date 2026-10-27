# Dragon Warrior IV — Smart Context

A BPS patch for the USA NES game Dragon Warrior IV. It changes five parts of the game. Each one stands on its own: the message window, the A button, the Select menu, Set AI, and battle tactics.

This repository does not include the game. You supply your own unmodified USA ROM. The patch file is 612 bytes.

This is a fan project. It is not made by or affiliated with Enix, Square, or Nintendo.

Made with Grok by xAI.

The project is [MIT licensed](LICENSE).

**[Download the patch](https://github.com/dunjeon/DragonWarriorIV-SmartContext/releases/latest)**

Notes: [dragon-warrior-iv-rom-hack-notes.docx](dragon-warrior-iv-rom-hack-notes.docx)

Website: [dunjeon.github.io/DragonWarriorIV-SmartContext](https://dunjeon.github.io/DragonWarriorIV-SmartContext/)

GitHub shows this file on the repository page. The website is a separate page, so the same five features are written out there too.

## Page clear

The original message window scrolls. When more text is waiting, a continue mark appears under the block. Confirming that mark slides the old lines up and leaves some of them on screen. A screen reader then speaks those old lines again before it reaches the new text.

This patch clears the window. The next chunk is the only text in the box. That happens when you confirm the continue mark, and it happens again when the window is already full and the game needs another page.

A message can ask you to answer yes or no, or to pick something from a small menu. After that answer, older lines can stay in the window until the box fills or you confirm the continue mark. The clear still runs on the continue mark and when the window fills.

The continue-mark clear and the full-window clear were kept after they were used in play.

## Smart A

On the field, A tries three commands in order and stops at the first one that applies.

1. Door, if the tile one step ahead is a real door. The original Door command runs, so a locked door still asks for its key and still prints its own message. Talk does not run on that press. Search does not run on that press. The game waits until you release A, so the next check cannot Search while your finger is still down.

   A shop counter is stored in the same group of tiles as a door. Smart A leaves that counter alone and goes on to Talk, so the shopkeeper two tiles away is the person who answers. Door on the Select menu is still the original Door command, including on a shop counter.

2. Talk, if that tile was not a door. Talk looks one tile ahead. If that tile is empty, it looks two tiles ahead. The closer person wins. The second look is there because a shopkeeper stands two tiles away, with the counter between you and them.

3. Search, if nobody was there. Search may still say that nothing was found. That line is part of the command.

Door comes first so a person standing on the far side of a door does not take the button press that should open the door. You can still choose Talk, Door, or another command yourself from the Select menu.

## The Select menu

Select opens the command menu: Talk, Spell, Status, Item, Equip, Tactics, Door, and Set AI.

The old Search line is now Set AI. Smart A still searches the ground, so the menu does not need a second Search command.

Menu Talk looks two tiles ahead, and the closer person wins. It can reach a person through a one-tile wall. If both tiles are empty, it still says that nobody is there, and it still checks for an object in front of you. Smart A skips that object check and uses Door, then Talk, then Search instead.

Release Select after you choose a command or cancel. The menu was checked in play: Select opens it, and it stays closed until you press Select again.

## Set AI

Set AI is the field command in the slot Search used to occupy. It asks the game's own yes or no question, then prints the result in the message window so a screen reader can speak it.

- Yes. The window says Player AI On. Tactics guide the controlled character the same way they already guide the rest of the party.
- No. The window says Player AI Off. You enter that character's battle commands yourself, on the same fight menu as the original game.
- Cancel. The setting stays where it was.

A new game starts off. Any save that has never been switched on stays off too, so you get the original command menu until you choose Yes.

Choose it on the field, before the fight. Once the battle starts, that choice holds for the whole fight. Inside battle, Fight, Run, and Tactics still do their own jobs. Tactics changes the plan for the round. It leaves Player AI On or Player AI Off alone.

Everyone else in the party keeps using tactics either way. Set AI decides only whether the controlled character does too.

On and Off were both checked in play. On leaves that character to the tactic. Off opens the normal command menu for that character.

## Tactics from the start

The original game waits until late in the story to show the tactics commands. This patch shows them from the first chapter.

Each round the battle menu is Fight, Run, or Tactics.

Fight plays that round with the tactic already picked. The party then fights that plan.

Tactics is where the plan changes. The plans are Normal, Offensive, Defensive, Save MP, Use No MP, and Try Out. That screen also lets you change tactics, see spells, or see items.

The tactic is the plan the round is fought with. With Player AI On, it is also the plan for the controlled character. With Player AI Off, the rest of the party still fights from it while you enter your own commands.

## Apply

The ROM must be the unmodified USA file. Floating IPS checks the file and refuses a different one.

| | |
| --- | --- |
| File | `Dragon Warrior IV (USA).nes` |
| Size | 524,304 bytes |
| CRC32 | `F6B24806` |

Use [Floating IPS](https://github.com/Alcaro/Flips/releases). Apply `dragon-warrior-iv-usa-smart-context.bps` and save a new file. Leave the original ROM as it is.

Mesen ties the battery save to the ROM file name. Save the patched ROM under a new name if you want a separate save. Opening the original file name uses the original save.

## Checksums

| File | CRC32 |
| --- | --- |
| Unmodified USA ROM | `F6B24806` |
| Patched ROM | `7AA6D320` |
| This BPS | `F9FC774E` |

The patch file is 612 bytes. `F9FC774E` is the CRC32 of every byte except the last four. Those last four bytes store the same value, and Floating IPS checks it. A CRC32 of the whole file is the same for every valid BPS patch, so use `F9FC774E` and the 612-byte size when you check this download.

Release v1.0.0 is an older patch. Floating IPS rejects that file. A ROM made from it has checksum `D22F542B`, which is the previous version. Use the current download. Its patched ROM checksum is `7AA6D320`.
