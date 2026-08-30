Issue #39 is important. It confirms the downclock/state-management problem, but the analysis in the issue actually understates how broken that version of the code was.

The issue describes this sequence:

```c
current_frequency = cpu;   // becomes 333
...
cancelOverclock();
```

so `cancelOverclock()` has forgotten the previous 407 MHz state. That is correct for the ARK code being discussed.

But looking at that actual older ARK implementation, `cancelOverclock()` had an additional bug:

```c
u32 _num = (u32)(((float)(current_frequency * PLL_DEN)) / ((float)PLL_BASE_FREQ));
const u32 num = (u32)(((float)(current_frequency * PLL_DEN)) / ((float)PLL_BASE_FREQ));
```

Both the starting numerator and destination numerator use `current_frequency`.

So even if ARK had not overwritten `current_frequency` before calling `cancelOverclock()`, this:

```c
while (_num >= num) {
    updatePLLMultiplier(_num, PLL_MUL_MSB);
    _num--;
}
```

would execute exactly once.

For example, assuming it somehow retained 407:

```text
_num = PLL numerator for 407
num  = PLL numerator for 407

write 407
_num--

_num < num
exit
```

It never ramps toward 333.

The destination clearly should have been something equivalent to:

```c
const u32 num =
    (u32)(((float)(DEFAULT_FREQUENCY * PLL_DEN)) /
          ((float)PLL_BASE_FREQ));
```

So #39 identified a genuine bug, but there were actually two stacked errors:

1. `overclockHandler()` overwrote the old frequency before trying to cancel it.
2. `cancelOverclock()` used the current/old frequency for both ends of the ramp anyway.

And there is an interesting history here.

On May 23, commit `6c512231...` changed that second bug correctly:

```diff
- const u32 num = ... currFreq ...
+ const u32 num = ... DEFAULT_FREQUENCY ...
```

but at essentially the same time it stopped calling `cancelOverclock()` from the normal downclock path altogether. Instead ARK changed the handler to:

```c
else {
    if (currFreq > DEFAULT_FREQUENCY && cpu < currFreq)
        return;

    origSetClockFrequency(cpu, bus);
    currFreq = cpu;
}
```

So the evolution is approximately:

```text
Old ARK:
407 → request 333
    ↓
tries cancelOverclock()
    ↓
cancelOverclock is broken
    ↓
PLL remains high

Current ARK:
443 → request 333
    ↓
if (currFreq > 333 && 333 < currFreq)
    ↓
return
    ↓
PLL deliberately remains high
```

In other words, the exact bug reported in #39 was changed, but **the externally observable problem was not really fixed**. ARK went from “tries and fails to downclock” to “refuses to downclock.”

That makes issue #68 much less mysterious. The PSP Go user observes roughly 424 MHz while ARK says 333, and changing the requested clock doesn't restore normal behavior.  That is exactly the sort of software/hardware state divergence these bugs produce.

It also strengthens my suspicion that game-launch crashes involve clock-state transitions.

A PSP game does not necessarily behave as:

```text
XMB 333
→ launch game
→ set 443
→ stay 443 forever
```

During the transition, Sony modules or ARK itself can make clock requests. With ARK globally intercepting `sctrlHENSetSpeed()`, a perfectly legitimate request such as:

```c
sctrlHENSetSpeed(333, 166);
```

can simply disappear while the physical PLL remains overclocked.

That means code farther down the boot sequence may believe:

```text
CPU = 333
BUS = 166
normal PLL/domain configuration
```

while the hardware is actually still in ARK's custom >333-MHz configuration.

That is a very plausible way to get seemingly nonsensical results such as:

* game boot crashes but gameplay can be stable once successfully launched;
* some games behave differently;
* exiting one game changes whether another boots;
* displayed clock != actual clock;
* XMB/game selections don't correspond to the physical frequency;
* reboot/power-cycle changes behavior.

Those are precisely the kinds of reports in #71. One user reports the exact same procedure succeeding or failing unpredictably and that launching a different game can change whether the desired game subsequently works.

There is also another consequence I hadn't emphasized enough previously: **ARK maintains software clock state instead of deriving it from the actual PLL registers.**

Current code does:

```c
int currFreq = DEFAULT_FREQUENCY;
```

and later:

```c
if (currFreq > DEFAULT_FREQUENCY && cpu < currFreq)
    return;
```

So `currFreq` isn't authoritative hardware state. If anything external modifies the PLL—such as the experimental overclock tester—ARK doesn't necessarily know.

And because of the `currFreq = theoreticalFreq` off-by-one bug, even after ARK itself changes the clock its internal model is wrong by one 10-MHz step:

```text
physical requested target    currFreq
383                           393
403                           413
423                           433
443                           453
```

Thus its decisions about whether future clock requests constitute an upclock/downclock are being made using a value that can already be wrong.

### Relation to the stress-tool workaround

This also slightly broadens the explanation for why running the stress tool first helps.

The tester doesn't merely overclock the CPU. Its startup sequence explicitly does:

```c
unlockMemory();

scePowerSetClockFrequency(333, 333, 166);
cancelOverclock();

sceKernelDelayThread(300000);
```

Its `cancelOverclock()` is implemented correctly in this respect:

```c
u32 _num =
    ... THEORETICAL_FREQUENCY ...;

const unsigned int num =
    ... DEFAULT_FREQUENCY ...;
```

so it can actually walk the PLL back from an OC numerator to the 333-MHz numerator.

Therefore running the tester once potentially does at least three things ARK doesn't reliably do:

```text
1. unlocks the undocumented hardware registers
2. explicitly normalizes Sony's clock to 333/166
3. explicitly normalizes the custom PLL state back toward 333
4. waits 300 ms before proceeding
```

Then it performs its own controlled overclock.

So I would refine my previous diagnosis like this:

**The removed `unlockMemory()` remains the single best explanation for why merely running the tester changes the remainder of the power session.**

But **the broken clock state machine is probably equally important to actual game-launch reliability**. Issue #39 is fairly strong evidence that this area has been broken throughout multiple revisions of ARK-5 rather than being a speculative concern.

In fact, I'd now structure an experimental fix in three separate stages:

1. Restore only `unlockMemory()` and test cold-boot → 383/403/423/443.
2. Fix `currFreq` bookkeeping and implement a real >333 → ≤333 cancellation path.
3. Then test whether longer settle delays / `clearTags()` are still necessary.

I would not initially touch PLL constants or domain ratios, because that would make it harder to tell which defect actually fixed the crash.

The downclock fix in particular should probably not merely call `origSetClockFrequency(333,166)`. It should first progressively restore the custom PLL to its normal state, then hand control back to Sony's clock routine. That is what the original experimental implementation was designed to do.
