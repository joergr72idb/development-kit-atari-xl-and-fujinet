# Development Kit for Atari XL and FujiNet

A self-contained toolchain on macOS for writing Atari 8-bit XL/XE programs that use the FujiNet,
and testing them in an emulator with an emulated FujiNet. Everything is built into `opt/` inside
this folder; nothing is installed system-wide except a few Homebrew packages.

| Tool | Version | What it is |
|---|---|---|
| [cc65](https://github.com/cc65/cc65) | V2.19 (git) | C compiler (`cl65`) and the **ca65** assembler |
| [fujinet-lib](https://github.com/FujiNetWIFI/fujinet-lib) | 5.1.0 | C library for the FujiNet (network, SD card, clock, …) |
| [FastBasic](https://github.com/dmsc/fastbasic) | 4.8.1 | Modern structured BASIC, cross-compiled on the Mac to `.xex` |
| [atari800](https://github.com/atari800/atari800) | 7.2.1 | Emulator with NetSIO (needed to talk to FujiNet-PC) |
| [FujiNet-PC](https://github.com/FujiNetWIFI/fujinet-firmware) | 1.6.1 | The FujiNet firmware as a Mac program |

```
cl65 / fastbasic ──► build/*.xex ──► atari800 -netsio ── UDP 9997 ──► FujiNet-PC
                                                          SD card: opt/fujinet-pc/SD
                                                          web UI:  http://localhost:8000
```

## Setup

```sh
tools/setup.sh                 # Homebrew packages + builds everything missing into opt/
tools/setup.sh atari800        # rebuild single parts: cc65 fujinet-lib atari800 fastbasic fujinet-pc
```

Versions are pinned at the top of `tools/setup.sh`. Build logs go to `build/logs/`.

## Use

```sh
make                           # builds the samples: build/hello-c.xex, hello-asm.xex, hello-fb.xex
make run                       # FujiNet-PC + atari800 with the C sample
make run XEX=build/hello-fb.xex
make stop                      # stops this kit's FujiNet-PC
```

Samples in `samples/`:
- `hello.c` – C with fujinet-lib: asks the FujiNet for its version and IP address.
- `hello.s` – ca65 assembler: prints a line through CIO.
- `hello.bas` – FastBasic.

`opt/fastbasic/fastbasic.atr` is the FastBasic IDE for the Atari itself (boot it as a disk).

## Notes

- Compile with `-t atari`, not `-t atarixl`: fujinet-lib calls the OS SIO vector, which the
  atarixl runtime switches out (the call hangs).
- Start FujiNet-PC before the emulator (`make run` does that). Each emulated power-on reboots
  FujiNet-PC; `tools/run-fujinet` restarts it automatically.
- Only one FujiNet-PC can run at a time (UDP 9997, web port 8000) – stop other projects' instance first.
- The FujiNet-PC web UI has no login: never forward port 8000 to the internet.
- **OS ROM:** without ROM files atari800 uses its built-in AltirraOS, which works fine. For the
  original Atari OS put your own ROM dump into `rom/` (not in git, copyrighted) and set
  `ROM_OS_BB01R2=` in `atari800.cfg`.

## Using the kit from other projects

```make
DEV      ?= ../development-kit-atari-xl-and-fujinet
CL65     := $(DEV)/opt/cc65/bin/cl65
FNLIB    := $(DEV)/opt/fujinet-lib
ATARI800 := $(DEV)/opt/atari800/bin/atari800
export CC65_HOME := $(DEV)/opt/cc65/share/cc65
```

The folder layout under `opt/` matches the `atari-dev` folder the Berlin Blues BBS expects
(`make DEV=../development-kit-atari-xl-and-fujinet`).
