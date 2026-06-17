# Changelog

All notable changes to this project should be documented in this file. Future commits that change emulator behavior, compatibility, tests, or user-visible workflows should update this changelog in the same commit.

## Unreleased

- Patched the git-sourced ARM emulator dependency on master through the vendored crate so iPodLinux ARM fixes are used by clicky-core.
- Treated zero-mode SPSR writes as user-mode returns, which lets iPodLinux init enter with its user stack instead of the supervisor stack.
- Fixed ARM exception-return block transfers (`LDM ... ^` with PC) to restore registers into the bank selected by SPSR before restoring CPSR.
- Validated SPSR writes as well as CPSR writes so invalid saved mode bits cannot poison later exception returns.
- Fixed ARM block-transfer writeback for user-bank `LDM/STM ... ^` instructions, which Linux uses around privileged/user context transitions.
- Patched the vendored ARM emulator to validate CPSR restores/writes, avoiding a post-root iPodLinux panic when userland startup hits invalid mode bits.

## 2026-06-08

### Added

- Added iPodLinux-oriented IDE command support for the iPod 4G boot path:
  - Recalibrate (0x10) now completes and asserts IDE IRQ.
  - ReadNativeMaxAddress (0xf8) reports the final available LBA and asserts IDE IRQ.
  - ReadSectors now asserts IRQ when data is ready.
- Added focused regression tests for IDE command completion, sector-read IRQ assertion, native-max-address reporting, EIDE IRQ acknowledgement, pending IRQ clearing, and PCF5060x ADC status reads.
- Added an HLE bootloader SysI sysinfo tag stub needed by iPodLinux.

### Changed

- Reworked IDE register updates away from the earlier spawner-delayed register path and toward direct command completion state updates.
- Changed IDE interrupt delivery to pulse IRQs on command/data readiness so guests see completion edges more reliably.
- Changed EIDE IDE0 interrupt acknowledgement so either bit 3 or bit 4 clears the IDE0 IRQ state.
- Changed iPod 4G interrupt handling to sample level-style interrupt status every step after clearing pending edge interrupts, preventing missed IDE interrupts during multi-sector PIO transfers.
- Changed PCF5060x ADC emulation so battery and ADCIN1 muxes return a plausible ready ADC sample of about 4.0 V instead of zero.
- Changed unsupported PCF5060x ADC mux reads to return harmless ready/zero stub values instead of terminating the emulator as unimplemented.

### Fixed

- Fixed irq::Pending::clear() so it actually clears the pending trigger.
- Fixed iPodLinux boot progress past IDE command handling and interrupt acknowledgement paths that previously led to disk wait stalls such as hda: lost interrupt.
- Fixed Rockbox boot failure on Pcf5060x > ADCS2 by returning a nonfatal ready battery ADC status sample.
- Fixed the PCF5060x ADCS2 low-bit result calculation so the low two ADC bits are returned directly with the ready bit.
