# Dragon Warrior IV — Smart Context

A BPS patch for the USA NES game Dragon Warrior IV. It changes seven parts of the game. Each one stands on its own: the message window, the A button, the Select menu, Hunt, Set AI, battle tactics, and the dialogue font.

This repository does not include the game. You supply your own unmodified USA ROM. The patch file is 854 bytes.

This is a fan project. It is not made by or affiliated with Enix, Square, or Nintendo.

Made with Grok by xAI.

The project is [MIT licensed](LICENSE).

**[Download the patch](https://github.com/dunjeon/DragonWarriorIV-SmartContext/releases/latest)**

Notes: [dragon-warrior-iv-rom-hack-notes.md](dragon-warrior-iv-rom-hack-notes.md)

## Page clear

The original message window scrolls. When more text is waiting, a continue mark appears under the block. Confirming that mark slides the old lines up and leaves some of them on screen. A screen reader then speaks those old lines again before it reaches the new text.

This patch clears the window. The next chunk is the only text in the box. That happens when you confirm the continue mark, and it happens again when the window is already full and the game needs another page.

A message can ask you to answer yes or no, or to pick something from a small menu. After that answer, older lines can stay in the window until the box fills or you confirm the continue mark. The clear still runs on the continue mark and when the window fills.

The continue-mark clear and the full-window clear were kept after they were used in play.

## Smart A

On the field, A tries three commands in order and stops at the first one that applies.

1. Door, if the tile one step ahead is a real door. The original Door command runs, so a locked door still asks for its key and still prints its own message. Talk does not run on that press. Search does not run on that press. The game waits until you release A, so the next check cannot Search while your finger is still down.

   A shop counter is stored in the same group of tiles as a door. Smart A leaves that counter alone and goes on to Talk, so the shopkeeper two tiles away is the person who answers.

2. Talk, if that tile was not a door. Talk looks one tile ahead. If that tile is empty, it looks two tiles ahead. The closer person wins. The second look is there because a shopkeeper stands two tiles away, with the counter between you and them. Talk still runs in a town or a dungeon, so a person walking in a dungeon can be talked to. On the world map, Talk stays quiet.

3. Search, if nobody was there. Search may still say that nothing was found. That line is part of the command. On the world map, A searches, and the window shows that search's own line.

Door comes first so a person standing on the far side of a door does not take the button press that should open the door. You can still choose Talk or another command yourself from the Select menu. Door is not on that menu. Smart A is the only Door command.

Door, Talk, and Search were checked in play, including A on the world map. A door does not also Talk or Search on that press.

## The Select menu

Select opens the command menu: Talk, Spell, Status, Item, Equip, Tactics, Hunt, and Set AI.

The old Search line is now Set AI. The old Door line is now Hunt. Smart A still searches the ground and still opens doors, so the menu does not need a second Search command or a second Door command.

Menu Talk looks two tiles ahead, and the closer person wins. It can reach a person through a one-tile wall. If both tiles are empty, it still says that nobody is there, and it still checks for an object in front of you. Smart A skips that object check and uses Door, then Talk, then Search instead.

Release Select after you choose a command or cancel. The menu was checked in play: Select opens it, and it stays closed until you press Select again.

## Hunt

Hunt is the field command in the slot Door used to occupy. It asks the game's own yes or no question, then prints the result in the message window so a screen reader can speak it.

- Yes. The window says Hunt On. Random encounters stay as they are in the original game.
- No. The window says Hunt Off. Random monster fights do not start.
- Cancel. The setting stays where it was.

A new game starts on. Any save that has never been switched off stays on too.

A boss or other scripted fight does not use the random-encounter roll, so that fight still happens when Hunt is off. Hunt does not change Set AI, and the battle Tactics menu does not change Hunt.

The current build, including Hunt, was checked in play.

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

## Dialogue font

Nine dialogue letters were redrawn. The rest of the alphabet is the original art.

Zero is a box with a short upright stroke, so it does not match the round letter O. One has a flag and a wide foot. Lowercase f has a shorter crossbar. Lowercase m has gaps through the top. Lowercase w pinches at the bottom. The period is a two by two block. The comma is unchanged.

Every letter is still 8 pixels wide. Lowercase i and lowercase l used to leave most of that box empty, so the next letter sat far to the right. Those two letters now use the capital pictures. A word like will is drawn as wILL. The stored text is still the same letters.

The ink color is unchanged. The game still recolors it for day and night. The heavier menu alphabet, the window frame, and the continue mark are the original shapes.

The letters load when the game starts. Reset the patched ROM to see them.

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
| Patched ROM | `CE12F61E` |
| This BPS | `8A42DF20` |

The patch file is 854 bytes. `8A42DF20` is the CRC32 of every byte except the last four. Those last four bytes store the same value, and Floating IPS checks it. A CRC32 of the whole file is the same for every valid BPS patch, so use `8A42DF20` and the 854-byte size when you check this download.

Release v1.2.0 is the previous patch. It does not include the dialogue font, and a ROM made from it has checksum `6DB88122`. Release v1.1.0 does not include Hunt, and a ROM made from it has checksum `7AA6D320`. Release v1.0.0 is older, and Floating IPS rejects that file. A ROM made from it has checksum `D22F542B`. Use the current download. Its patched ROM checksum is `CE12F61E`.
