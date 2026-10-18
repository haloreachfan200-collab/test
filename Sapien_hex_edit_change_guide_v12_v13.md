# Halo Reach Sapien EXE — Hex-Editor Change Guide

This guide lists the byte changes used to build **v12** from the specified v10 executable, plus the separate v13 object-list-header change. It is intentionally limited to changed values, offsets, and the exact bytes to replace.

## Use the correct executable and offset type

**Base executable:** `sapien_stable_object_ai1024_obj10240_scripts1024_stim1024_mem169M_v10_listrefs1024_v9havok.exe`

The listed **VA** values are Ghidra/runtime virtual addresses. The **file offset** values are the positions to go to in a hex editor such as HxD (`Ctrl+G` → offset). Use the **file offset** column for hex-editor editing. These offsets are for this exact x64 executable; do not apply them blindly to another build.

All byte patterns below are shown in file order. The changed instruction sequences are the same length as their originals; no bytes are inserted or removed.

## A. Core changes: v10 → v11/v12

The following 18 byte replacements form the v11 change set. v12 includes all of them, plus the contact-point change in Section B.

| # | Setting / purpose | Before → after | VA | Hex-editor file offset | Exact bytes: before → after |
|---:|---|---|---|---|---|
| 1 | Object-pool constructor count: **10,240 → 20,480** (`0x2800 → 0x5000`) | `10240 → 20480` | `0x14109255C` | `0x0109195C` | `BA 00 28 00 00` → `BA 00 50 00 00` |
| 2 | Related object-slot global/initialization count: **4,096 → 20,480** (`0x1000 → 0x5000`) | `4096 → 20480` | `0x141092561` | `0x01091961` | `C7 05 29 F5 2E 05 00 10 00 00` → `C7 05 29 F5 2E 05 00 50 00 00` |
| 3 | Object backing allocation: **`0xABE000 → 0x157C000` bytes** | `11,264,000 → 22,528,000 bytes` | `0x1410925B7` | `0x010919B7` | `BA 00 E0 AB 00` → `BA 00 C0 57 01` |
| 4 | Object render-data backing: **`0xA000 → 0x14000` bytes** | `40,960 → 81,920 bytes` | `0x141092621` | `0x01091A21` | `41 B9 00 A0 00 00` → `41 B9 00 40 01 00` |
| 5 | HS script-thread/command-script pool: **1,024 → 2,024** (`0x400 → 0x7E8`) | `1024 → 2024` | `0x14045E262` | `0x0045D662` | `BA 00 04 00 00` → `BA E8 07 00 00` |
| 6 | Havok component-capacity assertion: **1,500 → 3,000** (`0x5DC → 0xBB8`) | `1500 → 3000` | `0x140504DEC` | `0x005041EC` | `41 81 7D 2C DC 05 00 00` → `41 81 7D 2C B8 0B 00 00` |
| 7 | Havok fixed allocator backing size: **`0x380000 → 0xE00000` bytes** (4×) | `3,670,016 → 14,680,064 bytes` | `0x140A581CA` | `0x00A575CA` | `41 B8 00 00 38 00` → `41 B8 00 00 E0 00` |
| 8 | Related Havok allocator setting: **`0x2F9999 → 0xBE6664`** (4×) | `0x2F9999 → 0xBE6664` | `0x140A5821B` | `0x00A5761B` | `BA 99 99 2F 00` → `BA 64 66 BE 00` |
| 9 | Havok allocation-size validation: **`0x380000 → 0xE00000`** | same as #7 | `0x140A58627` | `0x00A57A27` | `81 FD 00 00 38 00` → `81 FD 00 00 E0 00` |
| 10 | Havok allocation-budget assignment: **`0x271000 → 0x9C4000`** (4×) | `0x271000 → 0x9C4000` | `0x140A5863F` | `0x00A57A3F` | `48 C7 40 28 00 10 27 00` → `48 C7 40 28 00 40 9C 00` |
| 11 | Havok deallocation-size validation: **`0x380000 → 0xE00000`** | same as #7 | `0x140A586F0` | `0x00A57AF0` | `81 FE 00 00 38 00` → `81 FE 00 00 E0 00` |
| 12 | Havok budget reset on release: **`0x380000 → 0xE00000`** | same as #7 | `0x140A58708` | `0x00A57B08` | `48 C7 40 28 00 00 38 00` → `48 C7 40 28 00 00 E0 00` |
| 13 | Havok overflow-accounting initial limit: **`0x271000 → 0x9C4000`** | same as #10 | `0x14078351C` | `0x0078291C` | `C7 05 12 9E 46 04 00 10 27 00` → `C7 05 12 9E 46 04 00 40 9C 00` |
| 14 | Matching Havok overflow-accounting guard: **`0x271000 → 0x9C4000`** | same as #10 | `0x140783B5D` | `0x00782F5D` | `81 F9 00 10 27 00` → `81 F9 00 40 9C 00` |
| 15 | Havok allocator self-test full-size request: **`0x271000 → 0x9C4000`** | same as #10 | `0x1407837D4` | `0x00782BD4` | `41 B8 00 10 27 00` → `41 B8 00 40 9C 00` |
| 16 | Havok allocator self-test half-size request A: **`0x138800 → 0x4E2000`** | `1,280,000 → 5,120,000` | `0x140783801` | `0x00782C01` | `41 B8 00 88 13 00` → `41 B8 00 20 4E 00` |
| 17 | Havok allocator self-test half-size request B: **`0x138800 → 0x4E2000`** | same as #16 | `0x140783826` | `0x00782C26` | `41 B8 00 88 13 00` → `41 B8 00 20 4E 00` |
| 18 | Havok allocator self-test near-limit request: **`0x270FF0 → 0x9C3FF0`** | cap minus `0x10` | `0x14078385D` | `0x00782C5D` | `41 B8 F0 0F 27 00` → `41 B8 F0 3F 9C 00` |

### Resulting v11/v12 capacities from this section

| Limit / allocation | Before | After |
|---|---:|---:|
| Object-pool elements | 10,240 | **20,480** |
| Related object-slot initialization count | 4,096 | **20,480** |
| Object backing allocation | `0xABE000` bytes | **`0x157C000` bytes** |
| Object render-data backing | `0xA000` bytes | **`0x14000` bytes** |
| Script-thread/command-script pool | 1,024 | **2,024** |
| Havok component-count assertion constant | 1,500 | **3,000** |
| Havok fixed allocator backing | `0x380000` bytes | **`0xE00000` bytes** |
| Havok budget/accounting values | `0x271000` | **`0x9C4000`** |
| Related Havok allocator setting | `0x2F9999` | **`0xBE6664`** |

## B. v12 addition: Havok contact-point datums

| Setting / purpose | Before → after | VA | Hex-editor file offset | Exact bytes: before → after |
|---|---|---|---|---|
| Havok contact-point pool count: **1,024 → 4,096** (`0x400 → 0x1000`) | `1024 → 4096` | `0x14061CCCE` | `0x0061C0CE` | `BA 00 04 00 00` → `BA 00 10 00 00` |

This changes the count passed to the `havok contact points` pool. The datum element size remains `0x50` (80 bytes); nominal backing for the entries therefore changes from `0x14000` (80 KiB) to `0x50000` (320 KiB).

**v12 = all 18 replacements in Section A, plus this contact-point replacement.**

## C. Optional v13 addition: script object-list headers

This is an additional change on top of v12, separate from the v12 core list above.

| Setting / purpose | Before → after | VA | Hex-editor file offset | Exact bytes: before → after |
|---|---|---|---|---|
| `object list header` count per initialized thread: **48 → 256** (`0x30 → 0x100`) | `48 → 256` | `0x1405B01C8` | `0x005AF5C8` | `41 8D 51 30` → `31 D2 B6 01` |

The original instruction calculates `EDX = 0x30` (48) after `R9D` has been zeroed. The replacement instructions set `EDX = 0x100` (256): `xor edx,edx` followed by `mov dh,1`.

**v13 = v12 plus this one four-byte replacement.**

## Leave these values unchanged

These are intentional non-changes; do not alter them when reproducing v12 or v13.

| Purpose | VA | Hex-editor file offset | Bytes / value to preserve |
|---|---|---|---|
| `list object` pool count: keep **1,024** (`0x400`) | `0x1405B0210` | `0x005AF610` | `00 04 00 00` |
| User-designated protected value at `0x141092461`: keep the original `0x800` (2,048) immediate | `0x141092461` | `0x01091861` | `B8 00 08 00 00` begins at this VA; **do not patch it** |

Do not make extra changes to the slot-free/diagnostic calculation or to `0x141092461` as part of this guide.
