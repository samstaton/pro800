# Pro 800 firmware 1.4.6 — bugs and wish list

Compiled by Defros (Eindhoven) against firmware 1.4.6, sent to me on 27 April 2026.
These are his own observations — bugs he found while exploring the unit, plus
some suggestions for the firmware authors. Posted here so they're discoverable;
none of this has been independently reproduced by me.

—Sam, 28 April 2026

## Bugs (firmware 1.4.6)

1. Pro-800 does not send the Arp-Mode (CC#49) but does receive. (Arp-Mode CC#49)

2. When changing the BPM-value on the Sequential Take5 the Take5 sends Midi Clock signals via Midi-OUT. When receiving these signals on the Pro800 and the Pro800 is not in [Sync Clock]-BPM menu; a value of 0-999 is shown instead of BPM 50-400.

3. When [Sync Source] = Internal
    - Midi PPQN in [Sync Clock] is not visible (and must be fixated on 24) that's ok.

   When [Sync Source] = Midi Sync or USB Sync
    - Midi PPQN in [Sync Clock] is visible (but fixed on 24); you cannot edit this. = Bug.

4. When [Sync Source] = External
    - Midi PPQN in [Sync Clock] is visible (but fixed on 24); you cannot edit this. Unless you Push 5 times [Sync Clock], so you arrive again on PPQN: now you can edit this. = Bug

5. During editing Subdivision in [Sync Clock] the Pro800 sends Divide (CC#50). CC#50 doubles with the LSB of OSC A Freq. When received the Pro800 changes the Freq of OscA. Editing of Subdivision is not possible by incoming Midi via CC#50. Better to use a free CC-number. Like CC#47 (all numbers in hex).

6. When turning ARP on, and Unison = off, the pedal (CC#64) acts als Hold on/off. When turning Unison on at this moment, the pedal (CC#64) off is not functioning anymore as Hold off. ARP stays on all the time. = Bug

7. Pedal Polarity is not in the Sysex System Dump (or changes are not written in memory) = Bug.

8. In key-tune mode ([Settings]; press key + [Tune]); only the C reacts as it should. When you change the value -999 to 999 you hear the frequency responding of C. The value can be saved in Preset, and you see the change in sysex export. In key-tune mode ([Settings]; press key + [Tune]) while pressing other keys (C#-B); you don't hear the frequency responding, and the value cannot be saved = Bug. No changes are seen in the sysex when the preset is sent via inquiry sysex.

9. Changes in Manual Tuning of Voice-Element-Octave can not be stored. In the Settings Menu:
    1. Choose **Voice Select** [4]: Voice 1 to 8
    2. Choose **Retune Element** [5]: OscA, OscB, or Filter.
    3. Choose **Retune Octave** [5]: All, 0 to 7.
    4. Choose **Retune** by **Encoder:** Press [6] to Activate

   You can edit manually a value -999 to 999. This way you can edit 8 Voices x 3 elements (OscA, OscB, Filter) x 8 Octaves. In the GliGli firmware this is saved in system data. On the Pro800 the values disappear after power off = Bug. Question: can it be exported via Sysex?

10. Name is not copied to new location. I create a patch on basis of one in A23 without name. I save it on A24 by save-procedure [REC][24][24], there still no name. Then I can give a name via an editor on my PC (e.g. [Pro~800 Editor by Christer Janson](http://www.chrutil.com/pro-800)). Now A24 is called "Test1" by me. There are 3 methods to copy:
    1. When I save it to another location via [REC][25][25]. The name is not copied to A25. (bug) (When I overwrite the same location A24 with an edited patch via [REC][24][24]; the name is kept.)
    2. When I save it to another location via [REC][25][PRESET]. The name is not copied either to A25 (bug). (When I overwrite the same location A24 with an edited patch via [REC][24][PRESET]; the name is kept also.)
    3. When I copy A24 to A25 via [24]; [PRESET][5]; go to A25 by [25]; and paste [PRESET][6] [6]. The name is copied indeed. This is the only way to copy presets with their name.

11. When pasting presets the location texts are wrong. When I copy A24 to A25 via [24]; [PRESET][5]; display says: *Copied A-24*. Go to A25 by [25]; and paste [PRESET][6]. Display says: *Again to paste A-2H*; [6] Display says: *Pasted A2-H onto A2-J*. The right preset A24 is copied with the right name, but the location-names are rubbish. (bug)

12. The Behringer Quickguide documents that the sequencer's playback speed can be programmed by adjusting SPEED during playback and pressing RECORD (which does not light) to commit. This procedure does not function in v1.4.6 = bug.

13. CC#76 is received and accepts the values 00-1F. Shown in the display like other parameters, but nothing changes in the sound. Is it supposed to do something? Seems to be an abandoned parameter of an earlier version or a test function?

14. During tuning (short press) several strings with the opcode `38` were sent out by midi:
    ```
    F0 00 20 32 00 01 24 00 38 00 11 00 F7
    F0 00 20 32 00 01 24 00 38 00 12 00 F7
    F0 00 20 32 00 01 24 00 38 00 15 00 F7
    F0 00 20 32 00 01 24 00 38 00 16 00 F7
    F0 00 20 32 00 01 24 00 38 00 17 00 F7
    ```
    Is it a bug? What does it stand for?

## Wish list

1. It is not possible to import/export SEQ1 or SEQ2 via midi sysex for saving on PC (and restore later by sending from PC to the Pro800). So only 2 sequences can be kept and all other creations have to be deleted. That's a pity.

2. Not a bug but more logical: VIB Mod Delay works only for VIB-LFO. It should be the third option under 2 (where the other 2 parameters are also for VIB-LFO). It's now under 3 the third option, and that's confusing — makes you think it is also for LFO1.

3. You can transpose the sequencer and the arpeggiator by MIDI, by holding [SETTINGS] and pushing [PERF]. But you have to do this all the time during playing. After 1 key touch the Pro-800 goes out of this mode. That's difficult and dangerous, because the manual writes push [PERF]+[SETTINGS]; in this way you often miss the [SETTINGS] and when you then retry you hit [PERF] twice, which means: you lose your edited patch without saving. I propose: it would be much more convenient if it toggled and stayed in "transpose-mode-by-key during playing" until you use the combination again. (The Pro-One acts this way: after activating sequence playing, new notes only transpose the whole sequence.)

—Defros, 27 April 2026
