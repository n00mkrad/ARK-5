The code that enables >333 MHz is in [`Compat/PSP/src/overclock.c`](Compat/PSP/src/overclock.c). This is the substantive implementation behind the README's "True overclocking" feature.

The important part is that ARK does not merely ask Sony's normal power API for 383/403/etc. `initOverclock()` hijacks ARK's clock-set/get functions:

```c
void initOverclock() {
  // override clock set/get functions
  HIJACK_FUNCTION(K_EXTRACT_IMPORT(sctrlHENSetSpeed), overclockHandler, origSetClockFrequency);
  HIJACK_FUNCTION(K_EXTRACT_IMPORT(sctrlHENGetSpeed), getOverclockSpeed, origGetClockFrequency);
  adjustValues();
  sctrlFlushCache();
}
```

Then `overclockHandler()` detects requests beyond 333 MHz:

```c
void overclockHandler(int cpu, int bus){    
    if (cpu > DEFAULT_FREQUENCY && cpu <= MAX_ALLOWED_FREQUENCY && cpu > currFreq) {
        targetFreq = cpu;
        doOverclock();
    }
    else {
        if (currFreq > DEFAULT_FREQUENCY && cpu < currFreq) return;
        origSetClockFrequency(cpu, bus);
        currFreq = cpu;
    }
}
```

The declared range is actually 333–483 MHz:

```c
#define DEFAULT_FREQUENCY        333
#define MAX_ALLOWED_FREQUENCY    483
```

`doOverclock()` is where the 333-MHz hardware/firmware limit is bypassed. It first establishes the normal 333/166 clock, then modifies the PSP PLL and clock-domain registers directly. The relevant MMIO addresses are:

```c
// PLL multiplier
hw(0xbc1000fc) = multiplier;

// PLL control/ratio
hw(0xbc100068) = 0x80 | pll_ratio_index;

// CPU domain ratio
hw(0xbc200000) = ...

// bus domain ratio
hw(0xBC200004) = ...
```

It raises the PLL progressively in 10-MHz target increments:

```c
#define FREQUENCY_STEP 10

int theoreticalFreq = defaultFreq + freqStep;

while (theoreticalFreq <= targetFreq) {
    ...
    const u32 num =
        (u32)(((float)(theoreticalFreq * pll_den)) /
              ((float)pll_base_freq));

    updatePLLControl();

    while (_num <= num) {
        updatePLLMultiplier(_num, pll_mul_msb);
        _num++;
    }

    ...
    defaultFreq += freqStep;
    theoreticalFreq = defaultFreq + freqStep;
}
```

So 383, 403, 423 and 443 are particularly convenient targets: they are all exactly `333 + N*10`.

The overclock implementation is installed from [`Compat/PSP/src/syspatch.c`](Compat/PSP/src/syspatch.c):

```c
// Implement overclocking code
initOverclock();
```

There is also resume handling there which reapplies the overclock after the PSP wakes from suspend.

The frequencies you saw in the UI are indeed exact ARK values, not estimates. The CFW SDK defines:

```c
enum cpubus_clocks
{
    CPU_BUS_CLOCK_DEFAULT,
    CPU_BUS_CLOCK_133,
    CPU_BUS_CLOCK_222,
    CPU_BUS_CLOCK_333,
    // these are overclock values
    CPU_BUS_CLOCK_383,
    CPU_BUS_CLOCK_403,
    CPU_BUS_CLOCK_423,
    CPU_BUS_CLOCK_443,
    CPU_BUS_CLOCK_CUSTOM,
};
```

And [`XMBControl/src/settings.c`](XMBControl/src/settings.c) explicitly reads/writes:

```text
cpuclock:383
cpuclock:403
cpuclock:423
cpuclock:443
```

The complete path is therefore approximately:

```text
CPU Clock in XMB / CPU Clock in Game
        ↓
XMBControl
CPU_BUS_CLOCK_383/403/423/443
        ↓
settings file: cpuclock:383 etc.
        ↓
SystemControl/src/settings.c
custom_cpu_clock = 383
cpubus_clock = CPU_BUS_CLOCK_CUSTOM
        ↓
SystemControl/src/syspatch.c
sctrlHENSetSpeed(custom_cpu_clock, custom_bus_clock)
        ↓
[Compat/PSP hijack]
overclockHandler()
        ↓
doOverclock()
        ↓
direct PLL/MMIO manipulation
        ↓
actual CPU >333 MHz
```

The generic `cpuclock:` parser is visible in [`SystemControl/src/settings.c`](SystemControl/src/settings.c), where values other than the specially handled 133/222/333 cases become `custom_cpu_clock`.  The resulting `CPU_BUS_CLOCK_CUSTOM` case calls `sctrlHENSetSpeed()` at boot.

One nuance regarding the numbers: **383/403/423/443 are the requested target frequencies**, but the hardware PLL uses an integer numerator/denominator calculation. Consequently, the physical clock can be slightly quantized rather than mathematically exactly 383.000 MHz, and the PLL constants also differ for some PSP models. So your suspicion about them being rounded is reasonable at the hardware level even though the menu/config values themselves are exactly those integers.

The source comment also identifies where ARK's implementation came from: [`mcidclan/psp-undocumented-sorcery/experimental-overclock`](https://github.com/mcidclan/psp-undocumented-sorcery/tree/main/experimental-overclock).
