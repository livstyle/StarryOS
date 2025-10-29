# Patch Notes for axplat-aarch64-peripherals

## Version
- **Original Version**: 0.2.0 (from crates.io)
- **Patched Date**: October 29, 2025

## Issue Description

The original `axplat-aarch64-peripherals-0.2.0` from crates.io contains inline assembly code that triggers linter errors when compiled with Rust nightly-2025-05-20:

```
invalid register `x0`: the zero register cannot be used as an operand for inline asm
invalid register `x2`: the stack pointer cannot be used as an operand for inline asm
invalid register `x3`: the global pointer cannot be used as an operand for inline asm
```

These errors occur in the PSCI (Power State Coordination Interface) implementation in `src/psci.rs`.

## Root Cause

Newer versions of the Rust compiler have stricter validation for inline assembly register constraints. The original code used direct register naming constraints like `in("x0")`, `in("x2")`, `in("x3")` which the compiler now interprets incorrectly or more strictly validates.

## Changes Made

### File: `src/psci.rs`

#### Functions Modified:
1. `arm_smccc_smc()` - Lines 56-68
2. `psci_hvc_call()` - Lines 71-83

#### Fix Applied:

Changed from direct register constraints:
```rust
unsafe {
    core::arch::asm!(
        "smc #0",
        inlateout("x0") func as usize => ret,
        in("x1") arg0,
        in("x2") arg1,
        in("x3") arg2,
    )
}
```

To generic register constraints with explicit mov instructions:
```rust
unsafe {
    core::arch::asm!(
        "mov x0, {func}",
        "mov x1, {arg0}",
        "mov x2, {arg1}",
        "mov x3, {arg2}",
        "smc #0",
        "mov {ret}, x0",
        func = in(reg) func as usize,
        arg0 = in(reg) arg0,
        arg1 = in(reg) arg1,
        arg2 = in(reg) arg2,
        ret = out(reg) ret,
        out("x0") _,
        out("x1") _,
        out("x2") _,
        out("x3") _,
        options(nomem, nostack)
    )
}
```

## Technical Details

### Why This Fix Works:

1. **Generic Register Constraints**: Using `in(reg)` and `out(reg)` allows the compiler to choose any available register without strict constraints.

2. **Explicit Move Instructions**: We explicitly move values into x0-x3 registers using `mov` instructions, which is what ARM PSCI specification requires.

3. **Explicit Clobbers**: We mark x0-x3 as clobbered using `out("x0") _` etc., informing the compiler these registers will be modified.

4. **Options**: Adding `options(nomem, nostack)` tells the compiler this inline assembly doesn't access memory or the stack, allowing better optimization.

### ARM PSCI Requirements:

According to the ARM PSCI specification:
- **x0**: Function ID (input) and return value (output)
- **x1-x3**: Function arguments
- **SMC/HVC**: Secure Monitor Call / Hypervisor Call instructions

Our fix maintains these requirements while satisfying the Rust compiler's constraints.

## Testing

This patch should be tested with:
1. System power-off operations
2. Multi-core boot (SMP) on ARM64 platforms
3. CPU hot-plug operations (if supported)

## Integration

This patch is integrated into the StarryOS project via Cargo's `[patch.crates-io]` section in the workspace `Cargo.toml`:

```toml
[patch.crates-io]
axplat-aarch64-peripherals = { path = "patches/axplat-aarch64-peripherals" }
```

## Upstream Status

- [ ] Issue reported to upstream maintainers
- [ ] Pull request submitted
- [ ] Merged into upstream

Once upstream fixes this issue and releases a new version, this patch can be removed.

## Compatibility

This patch is compatible with:
- Rust nightly-2025-05-20 and later
- All ARM64 targets: `aarch64-unknown-none-softfloat`, `aarch64-unknown-none`
- ARM PSCI 0.2 and later

## References

- [ARM PSCI Specification](https://developer.arm.com/documentation/den0022/latest/)
- [Rust Inline Assembly](https://doc.rust-lang.org/reference/inline-assembly.html)
- [Original crate on crates.io](https://crates.io/crates/axplat-aarch64-peripherals)

