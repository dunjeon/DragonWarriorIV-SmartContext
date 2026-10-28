# Dragon Warrior IV NES ROM hack notes

October 28, 2026. Notes for the screen-reader hack of the USA NES release. This file matches Smart Context v1.3.0.

The working copy used to write this hack is named Dragon Warrior IV (USA) - page clear.nes. Other people get the same changes from the BPS patch `dragon-warrior-iv-usa-smart-context.bps`. Apply it to an unmodified USA ROM and save a new file. This document does not include the game.

Every change is in that ROM. There is no Lua script and no Game Genie code. The original ROM file was not modified. 693 bytes differ from the original. The patched ROM CRC32 is `CE12F61E`. The BPS is 854 bytes. Its target CRC32 is `CE12F61E`. It includes page clear, Door then Talk then Search, the Select menu, Hunt, Set AI, and the dialogue font. The ROM is still 524,304 bytes, and the iNES header is unchanged.

## What you can do in the game

On the walkabout field, the A button tries three commands in this order:

1. Door, if the tile one step ahead is a real door. The Door command runs, including a locked door and its message. The game waits until you release A, so Talk and Search do not run on that press. A shop counter still goes to Talk, so the shopkeeper two tiles away is the one who answers.
2. Talk, if that tile was not a door. It looks one tile ahead first. If that tile is empty, it looks two tiles ahead, which is how shop counters are set up. The closer person wins. On the zoomed world map this search is skipped, because people are not walking around out there. In a town or a dungeon it still runs, so a person in a dungeon can be talked to.
3. Search, if it was not a door and nobody was there. Search may still say that nothing was found. That line in the log is expected. On the world map, A reaches this command, and the line is the real search sentence.

Select opens the Command menu: Talk, Spell, Status, Item, Equip, Tactics, Hunt, and Set AI. After you choose or cancel, release Select so the menu does not open again.

Hunt uses the same yes/no menu as Set AI. Yes stores `$00` in SRAM `$62D6` and the log says Hunt On. No stores `$01` and the log says Hunt Off. Cancel changes nothing. On is the default, including a new game and any value other than `$01`, and random encounters stay. Off skips the random-encounter roll. The check is bank `$18` `$9BED`, which calls `$BFC8`. When Hunt is on, the old `$6288` bit 7 skip still works. That bit is not the Hunt flag, because it also stops the day and night clock. A boss or other scripted fight does not use that roll, so it still happens. Set AI and the battle Tactics menu do not change this flag. Door is no longer on the Select menu. Smart A is the only Door command, including a locked door and its key message. The menu can no longer treat a shop counter as a door.

The menu Talk command also looks two tiles ahead, and the closer person still wins. It can reach a person through a one-tile wall. If both tiles are empty it still says nobody is there, and it still does the original object-in-front check. Smart A skips that object check. It tries Door, then Talk, then Search.

The message log clears when you confirm the continue mark under a full message, and when the window fills and advances. The box then shows the new chunk. After a yes or no, or any other choice a message asks for, older lines can stay until the window fills or you confirm the continue mark. That limit was kept on purpose.

The battle Tactics command is on the menu from Chapter 1. Each round you choose Fight, Run, or a new tactic. After that choice, the party fights from the tactic you picked.

Nine dialogue letters were redrawn so they are easier to tell apart. Zero is a box with a short upright stroke, so it does not match the round letter O. Lowercase i and lowercase l use the capital pictures, so the gap on their right is one pixel. A word such as will is drawn as wILL. The stored text is still the same letters. The ink color is the color the game already uses. The menu alphabet, the window frame, and the continue mark are the original shapes. The letters load when the game starts, so reset the ROM to see them.

## Files and saves

Leave Dragon Warrior IV (USA).nes as it shipped. Apply the BPS to a copy and save that copy under a new name. The unmodified USA ROM is 524,304 bytes and its CRC32 is `F6B24806`. The patched ROM CRC32 is `CE12F61E`. The BPS file is 854 bytes. Its CRC32 is `8A42DF20`, which is the checksum of every byte except the final four. Those four bytes store the same value. Use Floating IPS.

Release v1.2.0 is the previous patch. Its target CRC32 is `6DB88122`. It does not include the dialogue font. Release v1.1.0 is older. Its target CRC32 is `7AA6D320`. It does not include Hunt, and it still has the world-map sentence bug. Release v1.0.0 is older still, and Floating IPS rejects that file. Its target CRC32 was `D22F542B`.

Do not delete battery saves, and do not edit them by hand. MesenCE picks the save from the ROM file name. Opening the original file name uses the original save.

Each battery holds one real quest. Save slots start at `$62EF`, then `$65DF`, then `$68CF`. The stride is `$02F0`. An empty slot begins with `$4B`. The checksum is the game's eight-round routine at `$C8AD`, not a plain sum.

## How the ROM is put together

The cartridge is iNES mapper 1 (MMC1 / SUROM): 32 banks of 16 KB program ROM, and CHR RAM. A CPU address in a bank mapped at `$8000` is file byte 16 + bank × 16384 + (address − `$8000`).

The field loop lives in the fixed bank, which the console maps at `$C000–$FFFF`. There are two copies of that bank, ROM banks `$0F` and `$1F`. The same field bytes were written to both. A change in only one copy works in some chapters and misses in others.

Bank `$16`, mapped at `$8000`, holds the text engine and the moved world-map routine. Bank `$1E` holds the command menu, Talk, Door, and Search. Bank `$11` holds the battle AI hook. Bank `$17` holds Set AI and Hunt. Bank `$18` holds the encounter roll and the dialogue font. Those banks do not share addresses. Bank `$16` `$BFB6` and bank `$11` `$BFB1` are different bytes.

The game calls many routines with `BRK`. The byte after `BRK` is a signature, and the next byte is an argument. Some menu opens and flag tests are four bytes. A disassembler that treats `BRK` as one byte will lose its place. For the `$03` form, the byte after the argument is also read as data and then executed. The moved world-map routine contains that form (`00 5B 33` followed by `LDA #0`). Leave that pair alone.

The controller byte in `$14` is A = `$01`, B = `$02`, Select = `$04`, Start = `$08`, then Up, Down, Left, and Right. `$0529` remembers an A press across frames. Select is not stored there. Bit 2 of `$0529` is leftover controller noise, not Select.

## Page clear

The old log scrolled. `$85AD` copied the lines upward and cleared the last row. That routine was replaced. The new `$85AD` waits one frame (`$C62D`), clears the text buffer (`$8652`), calls `$858F`, calls `$83FE` for the rest of the redraw, sets the cursor column from `$55` and the row to 0, calls `$85E7`, and returns.

Two places call it. The continue-mark confirm at `$81C1` used to jump to `$83D2`. It now calls `$85AD`. When the row counter at `$0552` reaches 8, `$83EC` calls `$85AD` and returns.

`$8028` calls `$83F0`. That hook saves the sentence number that arrived in A. If `$51` is `$17`, it pulls the saved number and returns, so a custom Huffman line does not also run the dictionary seek. Otherwise it restores A and jumps to `$872B`. The sentence is the one the text command named. `$83FA` is the high byte of that jump, not its own return. `$83FB` is the pull-and-return used when `$51` is `$17`. The redraw does not start there.

`$83FE` is `JSR $8652` and the original redraw tail. `$85AD` reaches it after `JSR $858F`. An earlier build started the redraw at `$83FB`. That entry is now the `$17` return, so the clear must not call `$83FB`.

A later experiment cleared the log when a menu returned. Continue and a new quest both return from a menu, so the first line of dialogue always took that path, and the clear ran before the window size was set. The game froze. The file-select screen had also crashed while that experiment used a jump into the middle of another routine. The whole experiment was removed. `$8A6F` and `$8AA1` are still `LDA $F8` / `RTS`. Do not call `$85AD` from `$8028`, and do not set a pending clear when a menu returns.

## Smart A, Hunt, and the Select menu

The field frame is `$C9ED`. It reads the controller, may move the party, handles A, then does the rest of the frame. Inside that, `$CA17` used to read the pad and spin until Select was released. The action button is only honored on some frames, because `$CA21` skips itself unless a mask of the frame counter `$3C` is zero. A still responds to a tap because `$CB98` stores the A bit in `$0529` on the frames in between. Select had no such memory, and the spin at `$CA17` usually started on a skipped frame, so a held Select was discarded before the menu check ever ran. Replacing the A bit mask with the Select bit would have sent Select down the Talk path, and the press would still have been discarded.

`$CA0E` is now the Select check, on every frame, after the existing one-frame wait. It reads the pad with `$D241`. If bit 2 is set it calls `$CC68`. `$CC68` saves the current bank, switches to bank `$17`, calls the old menu preamble at `$BF5E`, reads the pad until Select is up, and switches the bank back. `$CC46` is the Hunt bridge. It switches to bank `$17`, calls the Hunt routine at `$BF77`, then switches back to bank `$1E` so the menu syscall can finish. The frame then continues with `$D1E1` and the usual increment of `$3C`. If Select is up, `$CA0E` skips straight to that same ending.

A no longer enters `$CC68`. If A is up, `$CC04` branches to the old return at `$CC63`. The five branches that used to open the menu (`$CC12`, `$CC19`, `$CC20`, `$CC26`, and `$CC2D`) now enter `$CC53`. That stub saves bank `$0507`, switches to bank `$1E` through `$FF91`, calls `$80CE`, restores the bank, and returns to the field loop. The mode-2 path is unchanged in spirit: a tile that wants the board or enter action still jumps to `$CC81` (`LDA #$0B` / `LDX #$03` / `JMP $D1ED`). An empty world-map tile calls `$BFB6` in bank `$16` and returns. The bytes there are the old `$CC42–$CC62` sequence plus an `RTS`. The signature at `$BFD8` is untouched. That call reaches the right bytes only when bank `$16` is the bank mapped at `$8000`. If boarding a ship or entering a town from the world map crashes or does nothing, move that routine back into the fixed bank or switch to bank `$16` around the call.

Smart A itself is bank `$1E` `$80CE`. The field stub at `$CC53` calls that address. It stores `$80` in `$0539` so the delayed command popup stays quiet until a window close clears it. The shared search is `$80E7`. If sprite layer `$41` bit 7 is clear, it branches to `$8142` and returns with carry clear, which skips the person search. Otherwise it looks one tile ahead (`$81AE`, then the sprite scan `$8126`) and, only on a miss, steps once more (`$81B8`) and scans again. A hit leaves the sprite index in X and jumps to the original talk code at `$80F9`, which was not moved. The shared search ends in `JMP $8126`. On a hit of the near tile it jumps there without the extra step, so the second scan is the same tile. `$8126` only compares coordinates, starting at sprite 6, and returns carry set with X equal to the sprite.

Smart A calls `$B35D` before Talk. Carry is set when the tile one step ahead has behavior `$14` through `$23`, the same range the Door command uses. Behavior `$17` is also in that range, and it is the shop-counter tile, so `$17` is sent to Talk instead of Door. Any other door jumps to `$FF83`. `$FF83` calls the real Door command at `$B2A6`, then falls into `$FF86`. `$FF86` reads the pad once per frame until A is released, then returns to the bank-switch stub. Talk and Search do not run on that press. If Door does not run, the shared search runs. A hit continues at `$80F9`. A miss jumps to Search at `$B3F3`. Behavior `$27` is outside the door range, so it already falls through to Talk.

Menu Talk stays at `$80BA`, which is where the command jump table points. It uses the same two-tile search, then `JSR $BAEA`, then either `$84B6` or the nobody message (`LDA #$F0` / `LDX #$02` / `JMP $D1ED`).

On the world map, A used to print an item-shop line, or the line about coming back with fewer items, and it did not open a shop. Hunt On and Hunt Off both did it. Towns did not. Door and Search store the facing tile's X coordinate in `$51` before they print. The old `$83F0` then used `$51` as the sentence number, so the log spoke whatever sentence that coordinate selected. The hook above keeps the sentence number from the text command. World-map A now shows the real search line. The `$51` = `$17` skip is still there, so Hunt On, Hunt Off, Player AI On, and Player AI Off still print through the normal log.

## Tactics

The goal was an optional tactic for the controlled character, off unless chosen, with the Tactics menu available from Chapter 1. The official game hides battle tactics until Chapter 5. Before that, Tactics only reorders the party.

Bank `$11` `$AE0F` was a branch on the chapter value in `$615A`. It is now two `NOP`s, so execution falls into the Chapter 5 battle-command setup. Bank `$16` `$B61B` tested a chapter flag and returned 0 or 1. It now loads 1 and returns, so the menu shows the Tactics line. The flag test `BRK` in front of that load still runs.

Bank `$11` `$88CA` runs the original AI break and does not call `$BFB1`. `$8241` calls `$BFB1` instead of the stock finder `$870C`. That finder returns the hero, character id `$00`, and does not return a chapter lead such as Ragnar. When `$6178` is `$01`, `$BFB1` skips the fight prompt and stores `$FF` in `$7358`. When `$6178` is any other value and the finder returns nobody, party slot 0 gets the prompt. In the Chapter 1 opening, that slot is Ragnar, character id `$06`. The hero's character id is `$00`, and the stock finder still returns him in Chapter 5.

The field menu line that used to say Search now says Set AI. Smart A still searches, so the menu does not need its own Search command. Set AI opens the existing yes/no menu, script `$48`. Yes stores `$01` in SRAM `$6178` and the log says Player AI On. No stores `$00` and the log says Player AI Off. Cancel changes nothing. A new game, and any value other than `$01`, stays on manual commands, the same fight prompt as stock Dragon Quest IV. Set the flag on the field before a battle. It stays for that fight. The battle Tactics menu still changes the plan and does not write `$6178`. The other party members keep using tactics either way. When `$6178` is `$01`, `$BFB1` skips the fight prompt and the existing battle AI takes the turn. When it is not `$01`, the hero keeps the stock prompt, and a chapter lead is prompted from party slot 0. `$8219` is the original compare again. Set AI was confirmed in play. On leaves the controlled character to the tactic. Off gives that character the normal command menu. The two log lines are Huffman text in bank `$17`. Bank `$16` `$8028` calls `$83F0` first, and that hook skips the normal text seek only when `$51` is `$17`, after pulling the saved sentence number. `$51` is cleared after the line. The old Search jump at bank `$1E` `$8340` is now `JMP $FF70`. The Door jump at `$8343` is now `JMP $CC46`, the Hunt bridge. The menu label at bank `$16` `$B061` is HUNT, bytes `2C 39 32 38`. SET AI at `$B05B` is unchanged. Both fixed banks hold `BRK` index `$03`, argument `$78`, then `RTS`, which enters bank `$17` at `$BEE4`. Smart A still jumps to the old Search body at `$B3F3`. Each round the player still chooses Fight, Run, or Tactics. Fight plays the round from the tactic already picked. Tactics changes that plan.

## Dialogue font

The dialogue font is one bit per pixel, 8 bytes per glyph, in bank `$18` at `$B83D` through `$BC9D`. That block is 140 glyphs, indexes `$00` through `$8B`, at file offset `0x6384D`. Digits are `$00` through `$09`. Lowercase a through z are `$0A` through `$23`. Uppercase A through Z are `$24` through `$3D`. The period is `$77`. The heavier menu font starts at `$BC9D` and was left as it shipped. The loader at `$B798` was left as it shipped. It still writes eight zero bytes into the second bit plane, so each letter is still one color of ink. Day and night, and the other palette changes, recolor that ink. Do not change the font color.

The plotter in bank `$16` copies exactly 8 bytes and then advances a fixed 8 pixels. There is no width table. The rightmost column is the one-pixel gap before the next letter.

Nine glyphs differ from the original. Those nine account for 41 bytes. The other 1,079 bytes of this font are the original bytes.

`$00`, zero, used to be nearly the same oval as the letter O. A diagonal slash was drawn first. In a line of text it read as a 9, so that slash was dropped. The glyph in the ROM is a square box with a one-pixel upright stroke in the middle. A one-pixel gap sits on each side of that stroke. The letter O stays round. The number 8 is unchanged.

`$01`, one, has a small flag at the upper left and a wide foot. The stem is narrower, so the flag and the foot stick out.

`$0F`, lowercase f, still has the hook at the top. The crossbar no longer runs the full width of the letter. Lowercase t still has a full-width bar.

`$12`, lowercase i, uses the bitmap of capital I at `$2C`. Capital I was not edited. The text byte stored in a message is still lowercase i. Capital I has a bar across the top and a bar across the bottom, so the cell is full and the next letter sits one pixel away.

`$13`, lowercase j, is a little narrower. The hook at the bottom is still there, and there is still a blank row under the dot.

`$15`, lowercase l, uses the bitmap of capital L at `$2F`. Capital L was not edited. The text byte stored in a message is still lowercase l. A word such as will is drawn as wILL. The foot of L runs to the right edge of the cell, and that foot is what meets the next letter. The upright stroke itself still has open space beside it. That space is inside the letter.

`$16`, lowercase m, used to fill the top row, so the two gaps were hidden. The top row is now open. The gaps go through the top. Each gap is one pixel.

`$20`, lowercase w, used to fill the bottom row. The bottom now pinches into points, so the letter is not an m.

`$77`, the period, used to be one pixel wide. It is now a 2 by 2 block on the same baseline. The comma at `$76` was left as it shipped.

The window frame and the down-triangle continue mark at `$7F` are the original bytes. The letters load when the game starts. A ROM that is already running keeps the old shapes until it is reset on this file.

## Where the bytes are

Addresses below are CPU addresses. Fixed-bank lines were written to both bank `$0F` and bank `$1F`. Switchable banks are the bank named in the row, mapped at `$8000`.

| Where | What changed |
| --- | --- |
| Bank `$16` `$81C1` | Continue-mark confirm calls `$85AD`. It used to call `$83D2`. |
| Bank `$16` `$83EC` | A full window calls `$85AD` and returns. |
| Bank `$16` `$83F0` | Saves A. Jumps to `$872B` with that A unless `$51` is `$17`, in which case it pulls A and returns at `$83FB`. |
| Bank `$16` `$83FE` | Redraw continuation: `JSR $8652` and the original tail. `$85AD` calls this after `JSR $858F`. |
| Bank `$16` `$85AD` | Clear routine: wait, wipe the buffer, `JSR $858F`, `JSR $83FE`, reset the cursor. Replaces the old upward scroll. |
| Bank `$16` `$8028`, `$8A6F`, `$8AA1` | `$8A6F` and `$8AA1` are still the original `LDA $F8` / `RTS`. `$8028` is `JSR $83F0`. The clear-after-a-menu experiment was removed after it froze the first line of dialogue. |
| Fixed `$CA0E` | Every frame, if Select is down, open the command menu and wait until Select is released. |
| Fixed `$CC04` and five branches | No A returns at `$CC63`. The old menu branches enter the smart stub at `$CC53`. |
| Fixed `$CC42` | Empty world-map A calls bank `$16` `$BFB6`. |
| Fixed `$CC46` | Hunt bridge. Switches to bank `$17`, calls `$BF77`, then returns through `$FF91` with bank `$1E`. |
| Fixed `$CC53` | Switch to bank `$1E`, call smart A at `$80CE`, switch the bank back. |
| Fixed `$CC68` and `$CC81` | `$CC68` saves the bank, runs the old menu preamble from bank `$17` `$BF5E`, waits until Select is up, and restores the bank. `$CC81` is still the board/enter action and was not moved. |
| Bank `$16` `$BFB6` | Old empty-tile world-map bytes, plus `RTS`. Stops before the signature at `$BFD8`. |
| Bank `$1E` `$80BA` | Menu Talk. Shared two-tile search, then the object check and the nobody message. |
| Bank `$1E` `$80CE` | Smart A. `$0539` = `$80`, then Door. Behavior `$17` falls through to Talk. Any other door jumps to `$FF83`, which calls `$B2A6` and falls into the release wait at `$FF86`. A Talk miss jumps to Search at `$B3F3`. |
| Bank `$1E` `$80E7` | Shared search: if `$41` bit 7 is clear, skip it. Otherwise one tile, then a second tile only on a miss. |
| Bank `$1E` `$80F9` | Original talk-to-this-sprite code. Not moved. |
| Bank `$11` `$88CA`, `$8241` | If `$6178` is `$01`, `$BFB1` stores `$FF` in `$7358` and skips the fight prompt. Otherwise the hero keeps the stock prompt, and a chapter lead is prompted from party slot 0. `$88CA` no longer calls `$BFB1`. It still runs the AI break. |
| Bank `$11` `$AE0F` | Chapter branch replaced with `NOP`s so the later battle commands are reached from Chapter 1. |
| Bank `$16` `$B61B` | Tactics menu check returns 1, so the command is listed from the start. |
| Bank `$17` `$BF77` | Hunt yes/no. Yes stores `$00` at `$62D6`. No stores `$01`. Cancel returns before the store. |
| Bank `$18` `$9BED` | Encounter roll calls `$BFC8`. Hunt Off (`$62D6` = `$01`) returns before a random fight. `$6288` bit 7 still does that when Hunt is on, and it still stops the day and night clock. Hunt does not use that bit. |
| Bank `$18` `$B83D` | Dialogue font. Nine glyphs, 41 bytes: `$00`, `$01`, `$0F`, `$12`, `$13`, `$15`, `$16`, `$20`, and `$77`. Lowercase i and l use the capital I and L bitmaps. The loader at `$B798` and the menu font at `$BC9D` are unchanged. |

## Space that was used, and space to leave alone

There was no spare bank. The Select handler fits in the bytes that used to spin on Select, from `$CA0E` through `$CA20`. `$CC46` through `$CC52` is the Hunt bridge. The menu preamble that used to live at `$CC68` now lives at bank `$17` `$BF5E`, and `$CC68` is the bank-switch opener plus the Select release wait. The smart-action bank switch sits in `$CC53` through `$CC62`, which had been four unused `NOP`s plus the bytes freed when the world-map tail moved. Talk, Door, and Search share `$80BA` through `$80F8`. `$80F8` is the last byte of the shared search. `$80F9` is the first byte of the original talk hit and was not overwritten. `$FF80` is `JMP $C913`, a tail call with the same result as the old `JSR $C913` / `RTS`. `$FF83` is `JSR $B2A6` and returns into the release wait at `$FF86`. Leave `$FF8E` as `JMP $C03D`. The two `NOP`s inside the `$FF74` frame wait were left in place so `$12` still counts at the old speed. `$FF70` through `$FF73` were the only shared `$FF` bytes in both fixed banks. They are now the Set AI bridge. The Set AI toggle and both of its log lines occupy bank `$17` `$BEE4` through `$BF5D`. The old menu preamble now starts at `$BF5E`. The Hunt routine, its pointer table, and the Hunt On and Hunt Off lines occupy `$BF77` through `$BFC5`. `$BFC6` through `$BFD7` is still `$FF`, and `$BFD8` stays the signature. Bank `$11` `$BFB1` through `$BFD7` is the fight-prompt gate, and `$BFD8` stays the signature. Bank `$18` `$BFC8` is the Hunt encounter check. The dialogue font edit stays inside the existing cells at bank `$18` `$B83D`. No new routine was added for the font.

Do not use these as free space:

- Bank `$16` `$BFB6` through `$BFD7`. That is the world-map routine. `$BFD8` is the reset signature and must stay, including the DRAGON QUEST IV text.
- The `NOP`s at `$C1B6` and `$C048`, and the MMC1 timing at `$FF74` and `$FF91`.
- The four `NOP`s inside the sprite routine at `$DBF7`.
- Zero-page `$13`, `$10`, and `$11`. `$0553` bit 0 was a retired page-clear flag. Do not bring it back. `$6178` is the player-AI flag. `$01` means tactics guide the controlled character. Any other value means manual commands. `$62D6` is the Hunt flag. `$01` means random encounters are off. Any other value means Hunt is on. Neither byte is a scratch flag. `$617B` is the last byte of a 10-byte wagon record that bank `$10` walks from `$616A`, so Hunt does not use it. `$6178` sits inside that same wagon record. It was left there because Set AI worked in play.
- Bank `$16` `$8028` jumps to `$83F0` on purpose. The hook runs from `$83F0` through the return at `$83FC`. A jump into the middle of that hook is still unsafe. The old file-select crash was a jump into that spot when it was not a safe entry. `$83C8`, `$83E0`, and `$83EA` return at `$83EF`. Do not point them at `$83FA`.

## Checked in play

Page clear on the continue mark and on a full window was used in play and kept.

The clear after a menu choice was removed after it froze dialogue. The yes/no leftover was accepted.

The Tactics line shows up from early in the game. Each round the choice is Fight, Run, or Tactics. Fight plays the round from the tactic already picked. Set AI was confirmed in play: On lets tactics guide the controlled character, and Off gives that character the normal command menu. The other party members keep using tactics either way.

Select opens the Command menu. Confirmed in play after the every-frame check replaced the old Select wait.

Smart A tries Door, then Talk, then Search. Behavior `$17`, a shop counter, is not treated as a door on this path. If Door runs, that press waits until A is released and does not continue into Search. Door, Talk, and Search were confirmed in play in towns. A on the world map was confirmed after the sentence hook: the log shows the real search line, not a shop line. The current build, including Hunt, was checked in play.

If boarding a ship or entering a town from the world map fails, bank `$16` `$BFB6` is the first place to look. The fixed bank calls it without switching banks.

## If something breaks

- Select does nothing: confirm the page-clear ROM is the one loaded, not the original file name. The check is `$CA0E` in both fixed banks, and it looks at `$14` bit 2 after a fresh pad read.
- A talks to the wrong person: the near tile is scanned before the far tile. `$80E7` is that order.
- A door never opens, or a shop counter opens as a door: smart A uses `$B35D`, behaviors `$14` through `$23`, one tile ahead, and then skips behavior `$17`. The Select menu no longer has Door, so it no longer treats behavior `$17` as a door. Smart A still skips `$17`. If a door opens and Search also runs, `$FF83` did not fall into `$FF86`.
- World-map A shows a shop line: `$83F0` should save A and jump to `$872B` with that A. When `$51` is `$17` it must pull A before it returns. `$85AD` should call `$858F` and then `$83FE`, not `$83FB`.
- Boarding or entering a place fails: the routine at bank `$16` `$BFB6` is the suspect, because the fixed bank calls it without switching banks.
- The first line of dialogue freezes: a page-clear call is running before the window size is set. `$8A6F` and `$8AA1` should still be `LDA $F8` / `RTS`, and `$8028` should be `JSR $83F0`. A `$51` value left at `$17` also makes the next message skip its seek.
- Hunt Off still rolls random fights: `$62D6` should be `$01`, and bank `$18` `$9BED` should call `$BFC8`. Do not use bit 7 of `$6288` as the Hunt flag.
- The letters still look like the original: reset on the patched ROM. The loader at bank `$18` `$B798` runs when the game starts. `$B83D` should hold the nine new glyphs. The menu font at `$BC9D` stays the original heavier alphabet.
