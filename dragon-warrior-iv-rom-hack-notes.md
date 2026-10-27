# Dragon Warrior IV NES ROM hack notes

October 27, 2026. Notes for the screen-reader hack of the USA NES release.

The working copy used to write this hack is named Dragon Warrior IV (USA) - page clear.nes. Other people get the same changes from the BPS patch `dragon-warrior-iv-usa-smart-context.bps`. Apply it to an unmodified USA ROM and save a new file. This document does not include the game.

Every change is in that ROM. There is no Lua script and no Game Genie code. The original ROM file was not modified. 469 bytes differ from the original. The published BPS is `dragon-warrior-iv-usa-smart-context.bps`. It is 612 bytes. Its target CRC32 is `7AA6D320`. It includes page clear, Door then Talk then Search, the Select menu, and Set AI. The ROM is still 524,304 bytes, and the iNES header is unchanged.

## What you can do in the game

On the walkabout field, the A button tries three commands in this order:

1. Door, if the tile one step ahead is a real door. The Door command runs, including a locked door and its message. The game waits until you release A, so Talk and Search do not run on that press. A shop counter still goes to Talk, so the shopkeeper two tiles away is the one who answers.
2. Talk, if that tile was not a door. It looks one tile ahead first. If that tile is empty, it looks two tiles ahead, which is how shop counters are set up. The closer person wins.
3. Search, if it was not a door and nobody was there. Search may still say that nothing was found. That line in the log is expected.

Select opens the old Command menu: Talk, Spell, Status, Item, Equip, Tactics, Door, and Set AI. After you choose or cancel, release Select so the menu does not open again.

The menu Talk command also looks two tiles ahead, and the closer person still wins. It can reach a person through a one-tile wall. If both tiles are empty it still says nobody is there, and it still does the original object-in-front check. Smart A skips that object check. It tries Door, then Talk, then Search.

The message log clears when you confirm the continue mark under a full message, and when the window fills and advances. The box then shows the new chunk. After a yes or no, or any other choice a message asks for, older lines can stay until the window fills or you confirm the continue mark. That limit was kept on purpose.

The battle Tactics command is on the menu from Chapter 1. Each round you choose Fight, Run, or a new tactic. After that choice, the party fights from the tactic you picked.

## Files and saves

Leave Dragon Warrior IV (USA).nes as it shipped. Apply the BPS to a copy and save that copy under a new name. The unmodified USA ROM is 524,304 bytes and its CRC32 is `F6B24806`. The patched ROM CRC32 is `7AA6D320`. The BPS file is 612 bytes. Its CRC32 is `F9FC774E`, which is the checksum of every byte except the final four. Those four bytes store the same value. Use Floating IPS. Release v1.0.0 is an older file, and Floating IPS rejects it.

Do not delete battery saves, and do not edit them by hand. MesenCE picks the save from the ROM file name. Opening the original file name uses the original save.

Each battery holds one real quest. Save slots start at `$62EF`, then `$65DF`, then `$68CF`. The stride is `$02F0`. An empty slot begins with `$4B`. The checksum is the game's eight-round routine at `$C8AD`, not a plain sum.

## How the ROM is put together

The cartridge is iNES mapper 1 (MMC1 / SUROM): 32 banks of 16 KB program ROM, and CHR RAM. A CPU address in a bank mapped at `$8000` is file byte 16 + bank × 16384 + (address − `$8000`).

The field loop lives in the fixed bank, which the console maps at `$C000–$FFFF`. There are two copies of that bank, ROM banks `$0F` and `$1F`. The same field bytes were written to both. A change in only one copy works in some chapters and misses in others.

Bank `$16`, mapped at `$8000`, holds the text engine and the moved world-map routine. Bank `$1E` holds the command menu, Talk, Door, and Search. Bank `$11` holds the battle AI hook. Those banks do not share addresses. Bank `$16` `$BFB6` and bank `$11` `$BFB1` are different bytes.

The game calls many routines with `BRK`. The byte after `BRK` is a signature, and the next byte is an argument. Some menu opens and flag tests are four bytes. A disassembler that treats `BRK` as one byte will lose its place. For the `$03` form, the byte after the argument is also read as data and then executed. The moved world-map routine contains that form (`00 5B 33` followed by `LDA #0`). Leave that pair alone.

The controller byte in `$14` is A = `$01`, B = `$02`, Select = `$04`, Start = `$08`, then Up, Down, Left, and Right. `$0529` remembers an A press across frames. Select is not stored there. Bit 2 of `$0529` is leftover controller noise, not Select.

## Page clear

The old log scrolled. `$85AD` copied the lines upward and cleared the last row. That routine was replaced. The new `$85AD` waits one frame (`$C62D`), clears the text buffer (`$8652`), redraws (`$83FB`), sets the cursor column to `$55` and the row to 0, calls `$85E7`, and returns.

Two places call it. The continue-mark confirm at `$81C1` used to jump to `$83D2`. It now calls `$85AD`. When the row counter at `$0552` reaches 8, `$83EC` calls `$85AD` and returns. `$83F0` through `$83F9` jump to the normal text seek unless `$51` is `$17`. The return at `$83FA` stays put. The redraw path at `$83FB` calls `$8652` instead of `$85AD`, so a clear does not call itself.

A later experiment cleared the log when a menu returned. Continue and a new quest both return from a menu, so the first line of dialogue always took that path, and the clear ran before the window size was set. The game froze. The file-select screen had also crashed while that experiment used a jump into the middle of another routine. The whole experiment was removed. `$8028` calls `$83F0`, and `$83F0` jumps to `$872B` unless `$51` is `$17`. `$8A6F` and `$8AA1` are still `LDA $F8` / `RTS`. Do not call `$85AD` from `$8028`, and do not set a pending clear when a menu returns.

## Smart A and the Select menu

The field frame is `$C9ED`. It reads the controller, may move the party, handles A, then does the rest of the frame. Inside that, `$CA17` used to read the pad and spin until Select was released. The action button is only honored on some frames, because `$CA21` skips itself unless a mask of the frame counter `$3C` is zero. A still responds to a tap because `$CB98` stores the A bit in `$0529` on the frames in between. Select had no such memory, and the spin at `$CA17` usually started on a skipped frame, so a held Select was discarded before the menu check ever ran. Replacing the A bit mask with the Select bit would have sent Select down the Talk path, and the press would still have been discarded.

`$CA0E` is now the Select check, on every frame, after the existing one-frame wait. It reads the pad with `$D241`. If bit 2 is set it calls `$CC46`. `$CC46` calls the original menu entry at `$CC68`, then reads the pad until Select is up, so the menu does not reopen while the button is still down. The frame then continues with `$D1E1` and the usual increment of `$3C`. If Select is up, `$CA0E` skips straight to that same ending.

A no longer enters `$CC68`. If A is up, `$CC04` branches to the old return at `$CC63`. The five branches that used to open the menu (`$CC12`, `$CC19`, `$CC20`, `$CC26`, and `$CC2D`) now enter `$CC53`. That stub saves bank `$0507`, switches to bank `$1E` through `$FF91`, calls `$80CE`, restores the bank, and returns to the field loop. The mode-2 path is unchanged in spirit: a tile that wants the board or enter action still jumps to `$CC81` (`LDA #$0B` / `LDX #$03` / `JMP $D1ED`). An empty world-map tile calls `$BFB6` in bank `$16` and returns. The bytes there are the old `$CC42–$CC62` sequence plus an `RTS`. The signature at `$BFD8` is untouched. That call reaches the right bytes only when bank `$16` is the bank mapped at `$8000`. If boarding a ship or entering a town from the world map crashes or does nothing, move that routine back into the fixed bank or switch to bank `$16` around the call.

Smart A itself is bank `$1E` `$80CE`. The field stub at `$CC53` calls that address. It stores `$80` in `$0539` so the delayed command popup stays quiet until a window close clears it. The shared search is `$80E7`. If sprite layer `$41` bit 7 is clear, it branches to `$8142` and returns with carry clear, which skips the person search. Otherwise it looks one tile ahead (`$81AE`, then the sprite scan `$8126`) and, only on a miss, steps once more (`$81B8`) and scans again. A hit leaves the sprite index in X and jumps to the original talk code at `$80F9`, which was not moved. The shared search ends in `JMP $8126`. On a hit of the near tile it jumps there without the extra step, so the second scan is the same tile. `$8126` only compares coordinates, starting at sprite 6, and returns carry set with X equal to the sprite.

Smart A calls `$B35D` before Talk. Carry is set when the tile one step ahead has behavior `$14` through `$23`, the same range the Door command uses. Behavior `$17` is also in that range, and it is the shop-counter tile, so `$17` is sent to Talk instead of Door. Any other door jumps to `$FF83`. `$FF83` calls the real Door command at `$B2A6`, then falls into `$FF86`. `$FF86` reads the pad once per frame until A is released, then returns to the bank-switch stub. Talk and Search do not run on that press. If Door does not run, the shared search runs. A hit continues at `$80F9`. A miss jumps to Search at `$B3F3`. Behavior `$27` is outside the door range, so it already falls through to Talk.

Menu Talk stays at `$80BA`, which is where the command jump table points. It uses the same two-tile search, then `JSR $BAEA`, then either `$84B6` or the nobody message (`LDA #$F0` / `LDX #$02` / `JMP $D1ED`).

## Tactics

The goal was an optional tactic for the controlled character, off unless chosen, with the Tactics menu available from Chapter 1. The official game hides battle tactics until Chapter 5. Before that, Tactics only reorders the party.

Bank `$11` `$AE0F` was a branch on the chapter value in `$615A`. It is now two `NOP`s, so execution falls into the Chapter 5 battle-command setup. Bank `$16` `$B61B` tested a chapter flag and returned 0 or 1. It now loads 1 and returns, so the menu shows the Tactics line. The flag test `BRK` in front of that load still runs.

Bank `$11` `$88CA` runs the original AI break and does not call `$BFB1`. `$8241` calls `$BFB1` instead of the stock finder `$870C`. That finder returns the hero, character id `$00`, and does not return a chapter lead such as Ragnar. When `$6178` is `$01`, `$BFB1` skips the fight prompt and stores `$FF` in `$7358`. When `$6178` is any other value and the finder returns nobody, party slot 0 gets the prompt. In the Chapter 1 opening, that slot is Ragnar, character id `$06`. The hero's character id is `$00`, and the stock finder still returns him in Chapter 5.

The field menu line that used to say Search now says Set AI. Smart A still searches, so the menu does not need its own Search command. Set AI opens the existing yes/no menu, script `$48`. Yes stores `$01` in SRAM `$6178` and the log says Player AI On. No stores `$00` and the log says Player AI Off. Cancel changes nothing. A new game, and any value other than `$01`, stays on manual commands, the same fight prompt as stock Dragon Quest IV. Set the flag on the field before a battle. It stays for that fight. The battle Tactics menu still changes the plan and does not write `$6178`. The other party members keep using tactics either way. When `$6178` is `$01`, `$BFB1` skips the fight prompt and the existing battle AI takes the turn. When it is not `$01`, the hero keeps the stock prompt, and a chapter lead is prompted from party slot 0. `$8219` is the original compare again. Set AI was confirmed in play. On leaves the controlled character to the tactic. Off gives that character the normal command menu. The two log lines are Huffman text in bank `$17`. Bank `$16` `$8028` calls `$83F0` first, and that hook skips the normal text seek only when `$51` is `$17`. `$51` is cleared after the line. The old Search jump at bank `$1E` `$8340` is now `JMP $FF70`. Both fixed banks hold `BRK` index `$03`, argument `$78`, then `RTS`, which enters bank `$17` at `$BEE4`. Smart A still jumps to the old Search body at `$B3F3`. Each round the player still chooses Fight, Run, or Tactics. Fight plays the round from the tactic already picked. Tactics changes that plan.

## Where the bytes are

Addresses below are CPU addresses. Fixed-bank lines were written to both bank `$0F` and bank `$1F`. Switchable banks are the bank named in the row, mapped at `$8000`.

| Where | What changed |
| --- | --- |
| Bank `$16` `$81C1` | Continue-mark confirm calls `$85AD`. It used to call `$83D2`. |
| Bank `$16` `$83EC` | A full window calls `$85AD` and returns. `$83F0`–`$83F9` skip the text seek when `$51` is `$17`. The return at `$83FA` stays. |
| Bank `$16` `$83FB` | Redraw calls `$8652`, so the clear does not call itself. |
| Bank `$16` `$85AD` | Clear routine: wait, wipe the buffer, redraw, reset the cursor. Replaces the old upward scroll. |
| Bank `$16` `$8028`, `$8A6F`, `$8AA1` | `$8A6F` and `$8AA1` are still the original `LDA $F8` / `RTS`. `$8028` is `JSR $83F0`. The clear-after-a-menu experiment was removed after it froze the first line of dialogue. |
| Fixed `$CA0E` | Every frame, if Select is down, open the command menu and wait until Select is released. |
| Fixed `$CC04` and five branches | No A returns at `$CC63`. The old menu branches enter the smart stub at `$CC53`. |
| Fixed `$CC42` | Empty world-map A calls bank `$16` `$BFB6`. |
| Fixed `$CC46` | Original menu at `$CC68`, then wait until Select is up. |
| Fixed `$CC53` | Switch to bank `$1E`, call smart A at `$80CE`, switch the bank back. |
| Fixed `$CC68` and `$CC81` | Original command menu and the board/enter action. Not moved. |
| Bank `$16` `$BFB6` | Old empty-tile world-map bytes, plus `RTS`. Stops before the signature at `$BFD8`. |
| Bank `$1E` `$80BA` | Menu Talk. Shared two-tile search, then the object check and the nobody message. |
| Bank `$1E` `$80CE` | Smart A. `$0539` = `$80`, then Door. Behavior `$17` falls through to Talk. Any other door jumps to `$FF83`, which calls `$B2A6` and falls into the release wait at `$FF86`. A Talk miss jumps to Search at `$B3F3`. |
| Bank `$1E` `$80E6` | Shared search: if `$41` bit 7 is clear, skip it. Otherwise one tile, then a second tile only on a miss. |
| Bank `$1E` `$80F9` | Original talk-to-this-sprite code. Not moved. |
| Bank `$11` `$88CA`, `$8241` | If `$6178` is `$01`, `$BFB1` clears `$7358` and skips the fight prompt. Otherwise the hero keeps the stock prompt, and a chapter lead is prompted from party slot 0. `$88CA` no longer calls `$BFB1`. It still runs the AI break. |
| Bank `$11` `$AE0F` | Chapter branch replaced with `NOP`s so the later battle commands are reached from Chapter 1. |
| Bank `$16` `$B61B` | Tactics menu check returns 1, so the command is listed from the start. |

## Space that was used, and space to leave alone

There was no spare bank. The Select handler fits in the bytes that used to spin on Select, from `$CA0E` through `$CA20`, and in `$CC46` through `$CC52`. The smart-action bank switch sits in `$CC53` through `$CC62`, which had been four unused `NOP`s plus the bytes freed when the world-map tail moved. Talk, Door, and Search share `$80BA` through `$80F8`. `$80F8` is the last byte of the shared search. `$80F9` is the first byte of the original talk hit and was not overwritten. `$FF80` is `JMP $C913`, a tail call with the same result as the old `JSR $C913` / `RTS`. `$FF83` is `JSR $B2A6` and returns into the release wait at `$FF86`. Leave `$FF8E` as `JMP $C03D`. The two `NOP`s inside the `$FF74` frame wait were left in place so `$12` still counts at the old speed. `$FF70` through `$FF73` were the only shared `$FF` bytes in both fixed banks. They are now the Set AI bridge. The toggle and both log lines occupy bank `$17` `$BEE4` through `$BF5D`, which had been `$FF`. Bank `$11` `$BFB1` through `$BFD7` is the fight-prompt gate, and `$BFD8` stays the signature.

Do not use these as free space:

- Bank `$16` `$BFB6` through `$BFD7`. That is the world-map routine. `$BFD8` is the reset signature and must stay, including the DRAGON QUEST IV text.
- The `NOP`s at `$C1B6` and `$C048`, and the MMC1 timing at `$FF74` and `$FF91`.
- The four `NOP`s inside the sprite routine at `$DBF7`.
- Zero-page `$13`, `$10`, and `$11`. `$0553` bit 0 was a retired page-clear flag. Do not bring it back. `$6178` is the player-AI flag. `$01` means tactics guide the controlled character. Any other value means manual commands. It is not a scratch flag.
- Bank `$16` `$8028` jumps to `$83F0` on purpose. `$83F0` through `$83F9` skip the text seek when `$51` is `$17`. A jump into the middle of that hook is still unsafe. The old file-select crash was a jump into that spot when it was not a safe entry.

## Checked in play, and not checked

Page clear on the continue mark and on a full window was used in play and kept.

The clear after a menu choice was removed after it froze dialogue. The yes/no leftover was accepted.

The Tactics line shows up from early in the game. Each round the choice is Fight, Run, or Tactics. Fight plays the round from the tactic already picked. Set AI was confirmed in play: On lets tactics guide the controlled character, and Off gives that character the normal command menu. The other party members keep using tactics either way.

Select opens the Command menu. Confirmed in play after the every-frame check replaced the old Select wait.

Smart A tries Door, then Talk, then Search. Behavior `$17`, a shop counter, is not treated as a door on this path. If Door runs, that press waits until A is released and does not continue into Search. This has not been tried in play yet. The world-map move to bank `$16` `$BFB6` has not been proven in play either.

## If something breaks

- Select does nothing: confirm the page-clear ROM is the one loaded, not the original file name. The check is `$CA0E` in both fixed banks, and it looks at `$14` bit 2 after a fresh pad read.
- A talks to the wrong person: the near tile is scanned before the far tile. `$80E7` is that order.
- A door never opens, or a shop counter opens as a door: smart A uses `$B35D`, behaviors `$14` through `$23`, one tile ahead, and then skips behavior `$17`. Menu Door does not skip `$17`. If a door opens and Search also runs, `$FF83` did not fall into `$FF86`.
- Boarding or entering a place fails: the routine at bank `$16` `$BFB6` is the suspect, because the fixed bank calls it without switching banks.
- The first line of dialogue freezes: a page-clear call is running before the window size is set. `$8A6F` and `$8AA1` should still be `LDA $F8` / `RTS`, and `$8028` should be `JSR $83F0`. A `$51` value left at `$17` also makes the next message skip its seek.
