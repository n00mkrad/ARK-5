# ARK-5 overclock-related issue transcript digest

Repository: `PSP-Arkfive/ARK-5`

Issues covered: #21, #39, #43, #59, #67, #68, #71.

> **Transcript note:** This is a faithful transcript-style **paraphrase**, not a verbatim reproduction. It preserves the authors, order of posts, technical claims, reported reproduction steps, issue status/dates, and direct GitHub permalinks while avoiding long verbatim copying of third-party text.
>
> **Image note:** Issue #59 contains seven 1200×1600 image attachments in one comment. Their image URLs have been removed as requested. Public-web and GitHub-CDN retrieval did not expose their pixels to the available image reader, so the placeholders below describe only what the author explicitly establishes them to be — a sequence showing their ARK configuration/settings — rather than inventing unverified visual details.

---

## Issue #21 — 09g problem specifically on the 3000 model bootloader cIPL TA-095V2 MOTHERBOARD MODEL

Source: https://github.com/PSP-Arkfive/ARK-5/issues/21

- **Author:** `lokaaao`
- **State:** Closed — completed
- **Created:** 2026-05-30 11:18:12 UTC
- **Updated:** 2026-06-30 06:57:57 UTC
- **Closed:** 2026-06-30 06:56:30 UTC

### Opening post — `lokaaao`

The reporter is using a late PSP-3000 09g / TA-095v2 board. With both ARK clock settings initially at their normal/default values, using any frequency above 333 MHz causes problems. They specifically report that even a small increase to roughly 338 MHz in the experimental stress tester can make the XMB visibly glitch and can cause a major reduction in frame rate.

A full shutdown and restart clears the bad state until they try the higher clock again.

After updating to a newer ARK release candidate, game overclocking at 383 MHz with the XMB kept at 333 initially seemed to reduce some of the XMB corruption, but games still ran abnormally slowly and the XMB could become sluggish after entering a game. The reporter later adds that the XMB glitching returned as well.

### Comments

#### 1. `woerldonz-pixel`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/21#issuecomment-4582713623

Reports similar behavior on PSP-2003 units: clocks over 333 MHz lead to crashes. By contrast, their PSP-1002 can reportedly exceed 400 MHz.

#### 2. `lokaaao`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/21#issuecomment-4582724526

Suggests the behavior may depend on hardware-revision/component differences between PSP models.

#### 3. `JoseAaronLopezGarcia`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/21#issuecomment-4586639215

Says not every console handles the experimental overclock identically and suggests that 09g units may need PLL/configuration values similar to those used on PSP Street hardware. Their own 09g becomes unstable as it approaches 400 MHz.

#### 4. `lokaaao`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/21#issuecomment-4586668546

Clarifies that their problem is not merely instability near 400 MHz: even about 338 MHz severely affects FPS on their unit.

#### 5. `Carsonline1`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/21#issuecomment-4586840983

Attributes the difference to later-model hardware and expresses hope that the stress tester demonstrates that a usable overclock can eventually be made to work. The comment's explanation about a different instruction set is a user claim, not an established diagnosis in the thread.

#### 6. `lokaaao`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/21#issuecomment-4587567533

Says they hope a later implementation will solve the problem.

#### 7. `lokaaao`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/21#issuecomment-4840716283

Reports that the problem was fixed by later ARK updates.

---

## Issue #39 — Possible downclock bug in sctrlHENSetSpeed

Source: https://github.com/PSP-Arkfive/ARK-5/issues/39

- **Author:** `andymcca`
- **State:** Open
- **Created:** 2026-06-14 09:52:20 UTC
- **Updated:** 2026-06-14 09:52:48 UTC
- **Comments:** 0

### Opening post — `andymcca`

The reporter asks whether an analysis produced by Cursor correctly identifies an ARK-5 downclock bug.

The described sequence is:

1. The PSP is physically running at an overclocked frequency, around 407 MHz.
2. Code calls `sctrlHENSetSpeed(333, …)` to return to stock.
3. ARK's handler assigns the newly requested 333 MHz value to `current_frequency` before attempting to cancel the overclock.
4. `cancelOverclock()` therefore no longer has the previous overclocked frequency available as its starting state.
5. The PLL ramp-down does effectively nothing useful, after which the normal clock setter is called.
6. The software/UI can consequently report 333 MHz while the physical PLL remains overclocked.

The issue asks whether that control-flow diagnosis is correct.

No comments had been posted at the time retrieved.

---

## Issue #43 — Psp 3006 crashes when launching any game with overclock above 333mhz

Source: https://github.com/PSP-Arkfive/ARK-5/issues/43

- **Author:** `PixelatedManThe2nd`
- **State:** Open — reopened
- **Created:** 2026-06-17 14:24:02 UTC
- **Updated:** 2026-07-08 12:34:48 UTC

### Opening post — `PixelatedManThe2nd`

The reporter uses a PSP-3006, hardware generation 03g. Before moving from PRO-C to ARK-5 they say they reformatted their storage and returned the firmware to a clean/original state with Chronoswitch.

With ARK installed:

- Setting the **game** clock to 383 MHz or higher causes a game launch to show the PSP startup logo briefly, then a black screen, followed by a crash/restart.
- The Custom Launcher similarly refuses to start under those conditions.
- Setting the **XMB** clock to 383 MHz or higher can also produce a black screen and crash.
- The experimental overclock stress tester, however, can take the same console as high as roughly 468 MHz.
- They report no potentially conflicting plugins apart from CXMB.

The crucial workaround is session-dependent: after running the experimental overclock tester, ARK's normal XMB and game overclock settings begin working at frequencies that otherwise crash. This only lasts for the current power session. After a full shutdown, the stress tester has to be run again before ARK's overclock works.

### Comments

#### 1. `VirgoanSub`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/43#issuecomment-4911257870

Reports a closely related problem on a 09g PSP-3001. Games and the Custom Launcher fail at the same clock range, although XMB overclocking works on that console. They had not yet tested the experimental overclock utility.

#### 2. `PixelatedManThe2nd`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/43#issuecomment-4914801534

Speculates that their installation of an early ARK-5 release might somehow be related, but reiterates that the PSP otherwise behaves normally. The repeatable inconvenience is that the experimental overclock tester must be used again after every full power cycle before ARK's own overclock settings work.

#### 3. `PixelatedManThe2nd`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/43#issuecomment-4914817532

States that the issue had been closed accidentally and reopens it.

---

## Issue #59 — Ark 5.1

Source: https://github.com/PSP-Arkfive/ARK-5/issues/59

- **Author:** `AdeqSloth`
- **State:** Open
- **Created:** 2026-07-05 18:24:47 UTC
- **Updated:** 2026-08-16 22:22:00 UTC

### Opening post — `AdeqSloth`

The reporter upgraded a PSP-3000 03g from the latest ARK-4 revision to ARK 5.1. Installation itself succeeded.

They then set both the game and XMB CPU clock to 383 MHz and selected Reset VSH. The PSP shut down and would no longer boot normally. They could only start it using the PS button plus the right trigger, at which point System Information appeared to have fallen back to ordinary 6.61 rather than the ARK environment.

They eventually reinstalled clean OFW 6.61 and returned to the latest ARK-4 revision.

### Comments

#### 1. `F0L1`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/59#issuecomment-4973677591

Confirms a similar failure on a PSP Go running ARK 5.1. Enabling XMB overclock can leave the system unable to boot ARK normally. Their recovery procedure is to boot without the problematic ARK configuration, connect over USB, and manually change the ARK XMB clock setting back to 333 MHz.

#### 2. `AdeqSloth`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/59#issuecomment-4974212101

Asks whether returning the XMB clock to 333 MHz makes ARK-5 stable afterward and whether game-only overclocking remains usable.

#### 3. `F0L1`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/59#issuecomment-4975127479

Says that, in their experience, returning the XMB setting to 333 resolves the boot problem. They would avoid XMB overclocking altogether. Game overclocking has worked for them because it is applied only when starting game/homebrew software rather than during ARK/XMB boot.

#### 4. `AdeqSloth`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/59#issuecomment-4979545936

Asks whether `F0L1` also uses Category Lite, mentioning a separate occasion where the Game folder appeared empty despite the plugin being enabled.

#### 5. `F0L1`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/59#issuecomment-4979885509

Says they do use the Lite version and have not encountered that problem.

#### 6. `gabrel-dias`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/59#issuecomment-5309950197

Reports essentially the same game-launch failure. On ARK 5 Live with firmware 6.61 and no plugins enabled, setting the game clock above 333 MHz can prevent all tested games from booting; the console crashes and returns to the XMB. The same behavior occurs with the Full installation. The failure is not entirely consistent, so the commenter posts their configuration and asks whether a setting has been missed.

The original comment then contains seven 1200×1600 image attachments.

[**Image 1 removed — configuration image 1 of 7.** The author presents this sequence specifically as screenshots/photos of the settings they are using while reproducing the >333-MHz game-launch failure. Exact visible values could not be independently decoded through GitHub's attachment CDN in this environment.]

[**Image 2 removed — configuration image 2 of 7.** Part of the same ARK/PSP configuration sequence supplied by the commenter. Exact visible values could not be independently decoded through GitHub's attachment CDN in this environment.]

[**Image 3 removed — configuration image 3 of 7.** Part of the same ARK/PSP configuration sequence supplied by the commenter. Exact visible values could not be independently decoded through GitHub's attachment CDN in this environment.]

[**Image 4 removed — configuration image 4 of 7.** Part of the same ARK/PSP configuration sequence supplied by the commenter. Exact visible values could not be independently decoded through GitHub's attachment CDN in this environment.]

[**Image 5 removed — configuration image 5 of 7.** Part of the same ARK/PSP configuration sequence supplied by the commenter. Exact visible values could not be independently decoded through GitHub's attachment CDN in this environment.]

[**Image 6 removed — configuration image 6 of 7.** Part of the same ARK/PSP configuration sequence supplied by the commenter. Exact visible values could not be independently decoded through GitHub's attachment CDN in this environment.]

[**Image 7 removed — configuration image 7 of 7.** Part of the same ARK/PSP configuration sequence supplied by the commenter. Exact visible values could not be independently decoded through GitHub's attachment CDN in this environment.]

---

## Issue #67 — Ark 5.02

Source: https://github.com/PSP-Arkfive/ARK-5/issues/67

- **Author:** `AdeqSloth`
- **State:** Open
- **Created:** 2026-07-17 21:03:11 UTC
- **Updated:** 2026-07-17 21:03:11 UTC
- **Comments:** 0

### Opening post — `AdeqSloth`

On a PSP-3000 03g, setting the in-game clock to 383 MHz does not work: starting a game causes the PSP to shut down and reboot.

No comments had been posted at the time retrieved.

---

## Issue #68 — Clock setting for XMB and GAME does not alter.

Source: https://github.com/PSP-Arkfive/ARK-5/issues/68

- **Author:** `F0L1`
- **State:** Open
- **Created:** 2026-07-17 21:04:49 UTC
- **Updated:** 2026-07-17 21:04:49 UTC
- **Comments:** 0

### Opening post — `F0L1`

The reporter noticed an emulator running unusually quickly and the battery draining faster than expected even though the configured clock was 333 MHz. They therefore added instrumentation intended to compare the selected clock setting with the effective CPU speed.

On a PSP Go running ARK 5.0.2, they report approximately:

- Configured 333 MHz → measured around 424 MHz.
- Configured 443 MHz → still measured around 424 MHz.
- Configured 222 MHz → only falls slightly, to roughly 418 MHz.

They acknowledge that their measurement code still needs refinement, but conclude that the game-side CPU clock appears to remain stuck in an overclocked state rather than following the selected setting.

No comments had been posted at the time retrieved.

---

## Issue #71 — Games won't start past 333 MHz UNLESS I do this

Source: https://github.com/PSP-Arkfive/ARK-5/issues/71

- **Author:** `Globin35`
- **State:** Open
- **Created:** 2026-07-19 17:49:27 UTC
- **Updated:** 2026-08-25 18:45:16 UTC

### Opening post — `Globin35`

On a PSP-3000, directly launching a game with the clock set to 383 MHz or higher crashes the system.

The reporter finds a workaround:

1. Launch the game at 333 MHz.
2. Exit/back out.
3. Change the clock to 423 MHz.
4. Launch the same game again.

After that sequence, the game starts successfully and can remain stable for extended play. The reporter says the performance improvement in games such as OutRun 2 and Midnight Club is large enough to confirm that the overclock really is taking effect.

### Comments

#### 1. `MrMario2011`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/71#issuecomment-5048049912

Uses a PSP-2000 that had previously been capable of 443 MHz but had recently begun having problems with every clock above 333. The opening-post workaround was not reliable for them.

They later tried the procedure described farther down the thread by `Yabe-uke`, involving the stress tool, and report that it restores usable 443-MHz operation.

They also provide a recovery tip: if an XMB overclock is left configured and the next cold boot only produces a black screen, holding Select during boot can bypass the problematic CFW settings.

#### 2. `Yabe-uke`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/71#issuecomment-5059235500

On a PSP-3004 with ARK 5.0.2, finds the behavior highly inconsistent.

Some games can boot at 443 MHz, while many others only sometimes succeed at 423. Repeating an identical launch sequence can fail many times, yet launching a different game or rebooting can make the original game suddenly work. Conversely, a game that just worked can immediately begin crashing again.

The commenter therefore doubts that the original launch-at-333-then-relaunch method explains the underlying problem.

#### 3. `Globin35`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/71#issuecomment-5062801944

After more testing:

- A normal attempt to boot a game at 383 MHz crashes at startup consistently on their unit.
- Running the experimental overclock tester until it reaches 443 MHz, leaving the tester, and then configuring ARK for 443 produces a much better success rate.
- They successfully start a broad selection of games in sequence, while Metal Gear Solid: Peace Walker still crashes at startup.
- Testing substantially different configured clocks in OutRun produces obvious performance differences, which they use to rule out the possibility that the displayed overclock is merely cosmetic.

They wonder whether progressively raising/initializing the clock is important, while acknowledging that this is only a hypothesis.

#### 4. `Yabe-uke`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/71#issuecomment-5062950177

Confirms that staged overclocking can sometimes help, although not with perfect consistency. They briefly consider whether a warm-up effect could be involved but lean toward a problem in how the firmware applies the overclock, because games can demonstrably run once the state is established.

They propose trying several PSP revisions to compare behavior and suggest games/patches that make real performance differences easier to observe.

They additionally note an important contrast: on their PSP, the experimental overclock application itself can remain running at roughly 468 MHz indefinitely without crashing or hanging.

#### 5. `gryanchristopher-coder`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/71#issuecomment-5110537421

Confirms the basic workaround while using an FPS counter. Directly starting at 423 MHz can require several crash/retry cycles.

They suggest that the dangerous part may specifically be starting a game while already at the high clock, and speculate about launching at a normal clock before applying the selected overclock after startup.

#### 6. `Yabe-uke`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/71#issuecomment-5216382172

After further testing, gives a more repeatable session-initialization procedure:

1. Boot the PSP.
2. Set the XMB clock to 443 MHz but leave the game clock at 333.
3. Start the experimental overclock stress tool.
4. Start its test.
5. Exit as soon as the tool reports that it is running.
6. The XMB is then stable at 443.
7. Set the game clock to 443 as well.

They report stable XMB/game operation after this procedure.

They warn that the PSP should be returned to 333 MHz before shutdown or the subsequent boot may fail; recovery mode can be used if necessary.

There are still workload-specific limitations. Some CSO games reportedly fail or hitch badly, with LittleBigPlanet mentioned as unable to boot and GTA titles exhibiting pauses associated with decompression. PSP minis also appear to dislike the overclock, leading the commenter to suspect that executable/boot-path differences matter.

#### 7. `gryanchristopher-coder`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/71#issuecomment-5225740855

Asks whether the stress-tool initialization has to be repeated after every full shutdown and notes that the setup can stop working after long periods in sleep mode.

#### 8. `Yabe-uke`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/71#issuecomment-5225771353

Says the procedure does need to be repeated after a full power-off, as far as they know. They recommend returning the clock to 333 before shutting down.

They also report that sleep mode is unreliable/broken while using the overclock and recommend avoiding sleep until the implementation is corrected.

#### 9. `MrMario2011`

Permalink: https://github.com/PSP-Arkfive/ARK-5/issues/71#issuecomment-5415092820

After updating the same PSP-2000 to ARK 5.1.0 (August 16, 2026 build), reports that they still need to use `Yabe-uke`'s stress-tool initialization sequence. The behavior therefore persisted for them after moving from ARK 5.0.2 to 5.1.0.

---

## Cross-reference

The recurring observations across these threads are:

- #21 — hardware-revision-sensitive instability/glitching above 333 MHz, later reported fixed for the affected 09g unit.
- #39 — software-state/downclock bug where the previous overclocked state can be lost before cancellation.
- #43 — direct >333-MHz launch crashes, but running the experimental tester once can make ARK overclocking work until power-off.
- #59 — XMB overclock can cause boot failure; later commenter also reports intermittent game launch failures above 333.
- #67 — minimal 03g report of 383-MHz game-launch crash/reboot.
- #68 — measured clock appears stuck around the overclocked range regardless of the configured value.
- #71 — extensive reports that clock/launch behavior depends on prior state; the experimental stress tool can "prime" a session, while sleep/shutdown and some boot paths remain problematic.
