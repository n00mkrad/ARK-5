There are at least two concrete bugs in ARK-5's overclock implementation, and one regression is an unusually good match for the "run the stress tester once, then ARK overclock works until reboot" reports.

The strongest suspect is a setup step that ARK removed on May 23.

Earlier ARK code contained this:

```c
static inline void unlockMemory() {
    const u32 start = 0xbc000000;
    const u32 end   = 0xbc00002c;

    for (u32 reg = start; reg <= end; reg += 4) {
        hw(reg) = -1;
    }

    sync();
}
```

and `initOverclock()` called it before using the PLL overclock code.

Commit `6c512231...` removed both the function and its call, replacing:

```c
// set up
unlockMemory();
```

with:

```c
adjustValues();
```

That matters because the standalone experimental overclock program—the very tool users report "fixes" ARK for the rest of the boot—still performs essentially exactly that operation:

```c
static inline void _unlockMemory() {
    const u32 start = 0xbc000000;
    const u32 end   = 0xbc00002c;

    for (u32 reg = start; reg <= end; reg += 4) {
        hw(reg) = -1;
    }

    sync();
}
```

and its initialization path explicitly does:

```c
sceKernelIcacheInvalidateAll();
unlockMemory();

scePowerSetClockFrequency(333, 333, 166);
cancelOverclock();

sceKernelDelayThread(DELAY_AFTER_CLOCK_CHANGE);
```

with `DELAY_AFTER_CLOCK_CHANGE = 300000` µs.

This fits issue #43 almost too well: that user's PSP-3006 crashes at 383+ through ARK, can reach 468 MHz with the experimental tester, and after running the tester once ARK's XMB/game overclock works normally for the remainder of that power session. After fully powering off, the tester has to be run again.

Issue #71 independently reports basically the same phenomenon: normal 383 MHz game launch crashes, but running the tester up to 443 first makes numerous games boot successfully.  Another reported reliable recipe is explicitly to start the OC tool, press Triangle, exit as soon as it says "running," and only then set ARK to 443 MHz.

I would therefore put the removed `unlockMemory()` initialization at the top of the suspect list. The name is the upstream author's name for it; these are undocumented registers, so I wouldn't claim we know precisely what every write does. But something that the tester changes during initialization clearly persists until power-off, and this is the most conspicuous persistent initialization operation that ARK used to have and then removed.

There is also a definite, independent bug in current ARK.

Current `overclockHandler()` contains:

```c
void overclockHandler(int cpu, int bus) {
    if (cpu > DEFAULT_FREQUENCY &&
        cpu <= MAX_ALLOWED_FREQUENCY &&
        cpu > currFreq) {

        targetFreq = cpu;
        doOverclock();
    }
    else {
        if (currFreq > DEFAULT_FREQUENCY && cpu < currFreq)
            return;

        origSetClockFrequency(cpu, bus);
        currFreq = cpu;
    }
}
```

So after ARK thinks it has overclocked the PSP, it deliberately ignores every subsequent request for a lower clock.

For example:

```text
ARK reaches 443
currFreq > 333

Sony/game/loader asks for 333
→ ignored

asks for 222
→ ignored

user changes 443 → 383
→ ignored
```

That is almost certainly wrong for a global hook around the normal clock API. PSP firmware and games are allowed to change clock rates as part of loading, suspend/resume, etc.

And this wasn't always the behavior. The same May 23 change replaced:

```c
else {
    cancelOverclock();
    origSetClockFrequency(cpu, bus);
}
```

with the "if lower, just return" logic.

There is already a user report that looks exactly like this bug: on a PSP Go, setting 333 reportedly left the actual CPU around 424 MHz, selecting 443 still gave ~424, and even selecting 222 only changed it slightly.

That state-machine bug could also explain why game launch is a particularly common crash point. ARK globally hooks the clock setter. If the game boot process expects to temporarily return to a supported Sony clock/domain configuration and ARK silently refuses it, the loader is now running with hardware state it did not request.

There's another straightforward bookkeeping error. `doOverclock()` does:

```c
int theoreticalFreq = 333 + 10;

while (theoreticalFreq <= targetFreq) {
    ...
    defaultFreq += 10;
    theoreticalFreq = defaultFreq + 10;
}

currFreq = theoreticalFreq;
```

Consequently:

```text
requested target    currFreq recorded
383 MHz             393 MHz
403 MHz             413 MHz
423 MHz             433 MHz
443 MHz             453 MHz
```

The actual PLL hasn't necessarily been set to that extra step—the software variable is simply advanced once too far before being stored.

That makes the lower-clock suppression even worse. After requesting 383, ARK records 393, so even another request for 383 qualifies as `cpu < currFreq` and is discarded.

There are several additional differences from the tester that I would treat as secondary suspects.

ARK's PLL-control check is:

```c
if (!(hw(0xbc100068) & pll_ratio_index)) {
```

where `pll_ratio_index == 5`.

That isn't a test for "current index equals 5"; it tests whether the current value happens to share any bits with binary `0101`. The original test tool uses an actual comparison:

```c
if ((hw(0xbc100068) & 0xff) != PLL_RATIO_INDEX) {
```

So an index of `1` or `4`, for example, can make ARK mistakenly decide the PLL is already in the desired state. `adjustPLLRatio()` normally tries to normalize it beforehand, so I don't think this explains the whole problem, but it's still incorrect code.

ARK also waits substantially less than the experimental implementation. Its `settle()` loop starts at roughly `0x02FFFF`, whereas the tester uses roughly `0x055555`, about 1.8× as many iterations. ARK additionally uses `sceKernelDelayThread(100)` between PLL steps—only 100 µs—while the tester waits 300 ms around major clock transitions.

And the tester executes:

```c
clearTags();
```

during each PLL step, clearing CP0 tag registers, while ARK currently has:

```c
// clearTags();
```

commented out.

Finally, ARK has its own `adjustDomainRatios()` which progressively drives both:

```text
0xBC200000  CPU domain
0xBC200004  bus domain
```

toward `511/511`. That's fairly invasive hardware manipulation and is another area I'd isolate experimentally, especially given reports where CSO decompression, Minis, or particular games behave differently.  But the evidence isn't nearly as strong for this one.

The model-specific PLL constants appear to be a separate, partially solved issue. On June 24 ARK changed 04g/07g/09g/11g to use the alternate "Street-like" PLL values.  That lines up with an earlier 09g report subsequently saying the problem was fixed.  It doesn't explain why 03g PSP-3000s and PSP-2000s that demonstrably survive 443–468 MHz in the tester fail at 383 through ARK.

So I would rank the likely problems:

1. **Very strong candidate: ARK removed the `unlockMemory()` hardware setup that the working stress tester still performs.**
2. **Definite bug: ARK suppresses every downclock once overclocked instead of cancelling/restoring the custom PLL state.**
3. **Definite bug: `currFreq` is stored one 10-MHz step above the requested target.**
4. **Definite correctness issue: PLL-ratio test uses bitwise overlap instead of equality.**
5. **Plausible stability issue: ARK uses much shorter settling delays and omits `clearTags()`.**
6. **Possible game-specific problem: ARK's forced CPU/bus domain-ratio changes.**

The first thing I would test on real hardware is a very small ARK build that changes nothing except restoring `unlockMemory()` in `initOverclock()`. That gives a clean A/B test of the most compelling hypothesis. If 383+ suddenly works from a cold boot without running the tester, the issue is essentially identified.

After that, I would fix the clock state machine regardless, because the current code is objectively inconsistent. In particular, downclocking should restore/cancel the custom PLL state, and `currFreq` should never be set to `target + 10`.
