# NORA migration — where these sources come from (2026-08-09)

This repository used to be the place the SPI/I2S/TDM transport HAL was edited. It
is not any more. Since the NORA migration it is a **published snapshot**: `src/`
is filled from the tree that is actually built and run on hardware, and this file
records which tree, which commit, and how the equality was checked.

## The chain

```
dsp-sonora audio board project        the tree that runs on hardware
  main
        |  vendored, byte-for-byte
        v
sulaolab/dspic33ak-hal-starter        MPLAB X project, 11 HAL modules
  refactor/nora-hal = a2ce22a
        |  published, byte-for-byte
        v
sulaolab/nora-hal-dspic33ak-spi-i2s-tdm     this repository
```

Direction matters: it used to run the other way (starter vendored *from* the
standalone repos). It was reversed because only the board project exercises the
code on silicon — 16-channel co-clocked dual-codec audio at high CPU load is not
something a sources-only repository can test — so it is the only place a fix can
be validated before it is published. A fix made here and not upstream would be a
fork.

## What the two commits did

| commit | what |
|---|---|
| `9e2c54d` | **rename only** — 10 files, all detected as R100 (100 % similarity). No byte of content changed, so the rename is reviewable on its own. |
| `a8dfac2` | **content refresh** — those files replaced with the starter's bytes, plus one new file. This is where the functional delta below entered. |

Splitting them this way is the whole point: a reviewer can confirm the rename is
mechanical without reading content, and then read the content diff without
rename noise.

### The rename mapping

| before | after | why |
|---|---|---|
| `src/dspic33ak_spi_i2s_tdm.h` | `src/nora_spi_i2s_tdm.h` | public header: no chip in the name |
| `src/dspic33ak_spi_i2s_tdm.c` | `src/nora_spi_i2s_tdm_dspic33ak.c` | backend: tagged |
| `src/dspic33ak_spi_i2s_tdm_conf.h_example` | `src/nora_spi_i2s_tdm_conf.h_example` | consumer-supplied config template; never compiled |
| `src/dspic33ak_spi_i2s_tdm_diag.h` | `src/nora_spi_i2s_tdm_diag.h` | public header |
| `src/dspic33ak_spi_i2s_tdm_diag.c` | `src/nora_spi_i2s_tdm_dspic33ak_diag.c` | backend |
| `src/dspic33ak_spi_i2s_tdm_hw.h` | `src/nora_spi_i2s_tdm_dspic33ak_hw.h` | backend-private silicon facts: tagged |
| `src/dspic33ak_spi_i2s_tdm_hw.c` | `src/nora_spi_i2s_tdm_dspic33ak_hw.c` | backend |
| `src/dspic33ak_spi_i2s_tdm_reg.h` | `src/nora_spi_i2s_tdm_dspic33ak_reg.h` | backend-private register layer: tagged |
| `src/dspic33ak_spi_i2s_tdm_fs_clc.h` | `src/nora_spi_i2s_tdm_dspic33ak_fs_clc.h` | backend-private CLC10 helper: tagged |
| `src/dspic33ak_spi_i2s_tdm_fs_clc.c` | `src/nora_spi_i2s_tdm_dspic33ak_fs_clc.c` | backend |
| — | `src/nora_spi_i2s_tdm_dspic33ak_diag_fast.h` | **new** in the refresh: the hot-path `static inline` profiler hooks |

`src/README.md` is not renamed; its content is touched only by the refresh.

The tag is `_dspic33ak`, not `_dspic33a`. `dsPIC33A` is the *core* family name;
these files drive dsPIC33AK SFRs. A dsPIC33CK backend would be `_dspic33ck` — a
different silicon family (dsPIC33**C**), never shortened to `_dspic33c`.

## Proof of identity with the upstream tree

Git blob hashes, this repository at `a8dfac2` vs `dspic33ak-hal-starter` at
`a2ce22a` (`src/hal_spi_i2s_tdm/`). Identical hash = identical bytes; git
normalises EOLs into the blob on both sides, so the CRLF working trees do not
disturb the comparison.

| file | blob | bytes |
|---|---|---|
| `README.md` | `7a7c766f9d9d` | 12470 |
| `nora_spi_i2s_tdm.h` | `6e26ad145a28` | 41720 |
| `nora_spi_i2s_tdm_conf.h_example` | `eea43c083f79` | 11745 |
| `nora_spi_i2s_tdm_diag.h` | `d7f65cf7c113` | 5078 |
| `nora_spi_i2s_tdm_dspic33ak.c` | `36f18018c47b` | 113456 |
| `nora_spi_i2s_tdm_dspic33ak_diag.c` | `e8b2f834973d` | 13546 |
| `nora_spi_i2s_tdm_dspic33ak_diag_fast.h` | `563261964b59` | 6195 |
| `nora_spi_i2s_tdm_dspic33ak_fs_clc.c` | `7687eb29c1b7` | 8849 |
| `nora_spi_i2s_tdm_dspic33ak_fs_clc.h` | `0217a33eb1aa` | 3296 |
| `nora_spi_i2s_tdm_dspic33ak_hw.c` | `7807c8792445` | 23818 |
| `nora_spi_i2s_tdm_dspic33ak_hw.h` | `7f3cb7fcdd3c` | 5379 |
| `nora_spi_i2s_tdm_dspic33ak_reg.h` | `fad6746c6fff` | 5671 |

**12 of 12 identical.** The starter's folder holds one file more — an
`UPSTREAM.md` describing the vendoring direction, which is starter bookkeeping and
deliberately not published here; this file is its counterpart.

## What actually changed in the content refresh

Method: take each new file, reverse the naming (`nora_` → `dspic33ak_`,
`NORA_` → `DSPIC33AK_`, and collapse the `_dspic33ak` backend tag out of file
names, include guards and backend-private macros), and diff against the
pre-rename blob. Whatever is left is *not* naming.

Residue, in lines (a `+`/`-` pair on the same construct counts as both):

| file | + | − | of which non-comment |
|---|---|---|---|
| `nora_spi_i2s_tdm.h` | 62 | 27 | +16 / −9 |
| `nora_spi_i2s_tdm_conf.h_example` | 23 | 0 | +6 / −0 |
| `nora_spi_i2s_tdm_diag.h` | 40 | 69 | +19 / −17 |
| `nora_spi_i2s_tdm_dspic33ak.c` | 150 | 21 | +94 / −15 |
| `nora_spi_i2s_tdm_dspic33ak_diag.c` | 93 | 14 | +74 / −14 |
| `nora_spi_i2s_tdm_dspic33ak_fs_clc.c` | 0 | 0 | — |
| `nora_spi_i2s_tdm_dspic33ak_fs_clc.h` | 0 | 0 | — |
| `nora_spi_i2s_tdm_dspic33ak_hw.c` | 103 | 46 | +80 / −40 |
| `nora_spi_i2s_tdm_dspic33ak_hw.h` | 20 | 4 | +12 / −1 |
| `nora_spi_i2s_tdm_dspic33ak_reg.h` | 1 | 1 | 0 / 0 |
| `src/README.md` | 3 | 3 | 3 / 3 |

Three files are pure: both halves of the CLC10 frame-sync helper, and — apart from
one comment line naming the sibling HALs' register headers — the register layer.
`nora_spi_i2s_tdm_dspic33ak_diag_fast.h` has no pre-rename counterpart to diff
against; it is new.

This is a much larger residue than the other repositories in this wave, from five
distinct causes. None of them is a change to the transport's wire behaviour.

* **(a) The DMA HAL's types replaced raw values.** The leg table and diagnostics
  now use `nora_dma_channel_t` / `nora_dma_trigger_t` / the DMA status type where
  they used a bare `uint8_t` channel number, a raw CHSEL byte and a `uint32_t`
  status word. Consequence worth knowing before you compile:
  `nora_spi_i2s_tdm_diag.h` now `#include`s `nora_dma.h`, so the cross-repo
  dependency on `nora-hal-dspic33ak-dma` is **explicit in a public header** — an
  include-path requirement, not just a link-time one. That accounts for most of
  `_diag.h`'s two-sided residue.
* **(b) CPU interrupt bits moved to DFP bit aliases.** `_DMAxIE` / `_DMAxIF`
  instead of a per-device `IEC`/`IFS` pointer-and-mask table, the same change made
  in the i2c and uart HALs. It also **removes the AK128 bank straddle**: that
  part's DMA channels span `IEC1` and `IEC2`, which the table had to encode by
  hand. Most of `_hw.c`'s residue is those rows going away.
* **(c) Three things moved out of the public headers into the backend-private
  ones** — the device adapter (`DEV_AK512` / `DEV_AK128` / `DEVICE`), the
  `STAT_SPIROV` mask, and `hw_sample_ack_errflags()`, now in
  `_dspic33ak_hw.h` / `_dspic33ak_reg.h`. A symbol-level comparison reports these
  as "removed from the API"; they are not removed, they are no longer public. A
  consumer that reached for them was reaching into the backend.
* **(d) `NORA_TDM_SUMPROF` became mandatory.** `nora_spi_i2s_tdm.h` `#error`s if
  the conf header does not define it. **This is the one breaking change for an
  existing consumer's `conf.h`** — see the note in the README's "Required project
  config". It is deliberate rather than defaulted: the macro gates code *out*, so
  an absent macro would evaluate to 0 in the `#if` and silently drop the profiler
  from a project whose `conf.h` predates it. Defining it as `1` restores the
  previous behaviour. The `+23 / −0` in `conf.h_example` is that block plus its
  `0`-or-`1` validation `#error`.
* **(e) The TDMsum combined-occupancy profiler.** Three public entry points
  (`tdmsum_configure` / `tdmsum_reset` / `tdmsum_get`), the snapshot type, the ISR
  hooks, and the new `_dspic33ak_diag_fast.h` holding those hooks as
  `static inline` so they cost no call in the RX-block ISR. It answers a question
  the per-leg load monitor cannot: two legs' peaks can fall in different windows,
  so adding them overstates the load — this measures the peak *union* of TDM-active
  time over one common window. That is the bulk of `_diag.c`'s and the main
  backend's residue.

`src/README.md`'s residue is exactly three pre-rename sibling-repo URLs
(`dspic33ak-hal-dma`, `dspic33ak-hal-timer`, and its own). The fourth line the raw
diff shows is the device-adapter macro's name, which reverse-normalises onto the
old text — the mechanical proof that the starter's fix there (`a2ce22a`) was
naming and nothing else.

## Comment corrections made here ahead of upstream — since converged

Everything above describes the state as published on 2026-08-08, when every file under
`src/` was byte-identical to upstream. On **2026-08-09** a documentation review found a
class of error that the identity proof above cannot see, and it was fixed here first
rather than waiting for the next upstream refresh.

**Converged later the same day, so this section is history and not an open delta.**
The corrections below are now in the audio-board tree as well: `dsp-sonora`
`fix/nora-naming-convergence` `1615e04` copied them back byte-for-byte, after
checking that each file's pre-copy blob equalled its pre-fix blob, so the identity
holds by construction rather than by inspection. That branch lands on `main`
together with the rest of the migration, which is why the chain above still names
the pre-convergence `main` commit.

* `src/README.md` was a generation behind the root README. It still listed **CPU IRQ
  `IEC`/`IFS` masks** among the things to add for a new device, which the content
  refresh removed in favour of the DFP's `_DMAxIE` / `_DMAxIF` bit aliases — the root
  README says so explicitly at "Adding a new dsPIC33AK part". That is not a branding
  slip; a reader following the folder README would have gone looking for a mask table
  that no longer exists. The section now mirrors the root README, including `DMA trigger
  values (as nora_dma_trigger_t)`.
* `src/README.md` also named `nora_spi_i2s_tdm_fs_clc.{c,h}` and
  `nora_spi_i2s_tdm_hw.{c,h}`, which do not exist — the files carry the backend tag
  (`nora_spi_i2s_tdm_dspic33ak_fs_clc.*`, `..._dspic33ak_hw.*`). The same untagged names
  appeared in four comments in `nora_spi_i2s_tdm.h`, `nora_spi_i2s_tdm_dspic33ak.c` and
  `nora_spi_i2s_tdm_dspic33ak_hw.h`. Within the folder README the references were mixed
  — `..._dspic33ak_reg.h` was already tagged correctly while `..._hw` a few lines below
  it was not — which is the signature of a rename that updated file names and only some
  of the prose.
* Eight comments across seven files said `dsPIC33A` where they mean the dsPIC33AK
  backend, including the `#error` text for an unsupported device.

No executable code changed. The edits are comments and Markdown; the compiled
result is unchanged.

What is claimed, precisely, is `Sonora == Starter` for every shared HAL file. It is
deliberately **not** stated as byte identity across the whole fleet:
`dsp-sonora-dual-partition` carries intentionally divergent variants of some
backends and is reconciled separately.

### Why the proof in "Proof of identity" does not catch this

Step 3 reverse-normalises the NORA names back to `dspic33ak_*` and diffs against the
pre-rename blob, so whatever is left is not naming. Two error classes cancel out exactly
in that diff and are therefore invisible to it. Two more slipped past the sweep that was
supposed to back the diff up, for a different reason: the sweep's own shape -- which
extensions it opened, which pattern it matched -- decided what it was able to see.

* **A document reference to a file that was renamed.** A prose mention of
  `nora_<mod>_hw.{c,h}` reverse-normalises to `dspic33ak_<mod>_hw.{c,h}`, which is the
  *correct* pre-rename name — the diff is empty, yet the file is now called
  `nora_<mod>_dspic33ak_hw.{c,h}` and the reference is dead. The same cancellation hides
  `Nora` vs `NORA` and `dsPIC33A` vs `dsPIC33AK`: both sides of the diff are naming, so
  naming errors are exactly what it is blind to.
* **A document that omits a file the refresh added.** An absent line produces no diff
  line at all.
* **A sweep whose file-extension filter is narrower than the tree.** A `.c`/`.h` sweep
  cannot see a defect in a `.h_example` file. The file is present, the pattern matches
  nothing, and the empty result reads as "clean".
* **A grep shape that cannot express the name it is looking for.** `NORA_NVM_[A-Z][a-z]`
  was used to find mixed-case function names and silently cannot match
  `NORA_NVM_CRCPreflight`, because two capitals follow the prefix. The sweep returned a
  strict subset of the real hits and looked complete.

These blind spots were observed across the NORA-HAL migration fleet; the subset that
affected *this* repository is the list above. None of them is detectable by
reverse-normalisation. What does detect them is resolving every `nora_*.{c,h}`
mentioned in prose against the actual contents of `src/`, reading every
`dsPIC33A` / `Nora` hit rather than counting them, and — for the last two — letting
the tree decide which extensions the sweep covers and searching for exact names
rather than for a pattern that merely resembles them. That is how these were found.

## Hardware evidence

There is no build or test in this repository — it is sources only. The evidence is
the upstream project's: `dspic33ak-hal-starter`
`docs/nora_hal_migration_analysis.md` §11e records a PASS run of all 11 NORA-ised
modules on PKOB4 `020085204RYN000057` (dsPIC33AK512MPS512, Device ID `0xa77c`) on
2026-08-09, TDM8 included.

Scope, stated plainly, and it is the same envelope the README's "Tested envelope"
section describes: the system-topology model is HW-verified in the upstream audio
project (co-clocked dual-codec A/B, 80-stage / 94 % CPU load, phase-locked startup
with `miss=0`, CMSIS-SAI single-instance loopback), and the snapshot itself was
bench-verified through the starter (TDM8 master smoke, `FS_PULSE` and `FS_50PCT`,
stop→restart, the negative-config self-test matrix). An exhaustive format/role
matrix is **not** complete; slave is the main tested path. The CLC10 50 %-duty FS
generator is AK512-only — on AK128 `engage()` reports `NO_FS_PIN`. AK128 itself is
compile-supported; the §11e run was on AK512.

One item specific to this refresh: §11f of the starter's record is the finding that
the TDMsum profiler was **not** dead code — it was already running in the ISR — and
its fix. That is upstream-validated code, and it is what arrived here.

## Consumer impact

* The public namespace changed from `dspic33ak_*` / `DSPIC33AK_*` to `nora_*` /
  `NORA_*` and **no compatibility aliases were added**. Call sites must be renamed;
  the substitution is purely textual, and it includes the `NORA_TDM_*` conf-header
  macros.
* The `#include` names changed — see the rename mapping above.
* **`conf.h` must define `NORA_TDM_SUMPROF`** or the build stops at an `#error`.
  This is the only source-level break beyond renaming.
* The include path must reach `nora-hal-dspic33ak-dma`'s headers, because
  `nora_spi_i2s_tdm_diag.h` includes `nora_dma.h` now.
* Code that referenced `..._DEV_AK512` / `..._DEV_AK128` / `..._DEVICE`,
  `STAT_SPIROV` or `hw_sample_ack_errflags()` from outside the HAL must include the
  backend-private `..._dspic33ak_hw.h` / `..._dspic33ak_reg.h` — or, better, stop
  depending on the backend.
* A CMSIS-SAI wrapper sits above this HAL and is unaffected other than by the
  rename; `ARM_SAI_*` still does not appear here.
