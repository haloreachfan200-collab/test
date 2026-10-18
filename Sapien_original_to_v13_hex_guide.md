# Halo Reach Sapien EXE — Hex-Editor Guide: Original Build → v13

This guide documents the byte changes between these two exact executables:

- **Starting file:** `sapien_stable_object_ai256_v4_stim512_mem2_firingowners256.exe`
- **Target file:** `sapien_v13_obj20480_threads2024_havok4x_contactpoints4096_listheaders256.exe`

The guide is based on a direct comparison of those two files. It lists the values in the original file and the values present in v13, not targets copied from an older patch script.

## Before editing

1. Make a copy of the starting EXE.
2. In HxD, use **Search → Go to** (Ctrl+G), choose **hexadecimal**, and enter the **file offset** shown below. Do not enter the Ghidra virtual address (VA) in place of the file offset.
3. Replace bytes in place only. Do not insert/delete bytes, use wildcard replacements, or apply the table to another build without rechecking the bytes.
4. The byte strings are shown in file order. Multi-byte integer values in the EXE are little-endian.

For the listed starting file, the SHA-256 is:

`102d644cdad70475ef581fbe7ce51db32b5ff305566b8d6d0e6db736e6f9b12f`

The v13 target SHA-256 is:

`051b5268d926f0b8cf28f55f83c34d8761eed6960631f676521971b58b7d7d08`

## Summary of limits changed

| Setting | Starting file | v13 target |
|---|---:|---:|
| Main AI actor pool | 256 | **1,024** |
| Firing-position-owner hash buckets | 256 | **1,024** |
| AI command-script pool count | 256 | **1,024** |
| HS script-thread pool | 640 | **2,024** |
| Stimulus-data pool | 512 | **1,024** |
| Main object-slot pool | 4,096 | **20,480** |
| Related object-slot initialization count | 4,096 | **20,480** |
| Object backing allocation | `0x452000` bytes | **`0x157C000` bytes** |
| Object render-data backing | `0x4000` bytes | **`0x14000` bytes** |
| Actor debug-buffer allocations | `0xA4000` bytes | **`0x290000` bytes** |
| Script `list object`/reference count immediate | 128 | **1,024** |
| Script `object list header` count | 48 | **256** |
| Havok component maximum check | 1,500 | **3,000** |
| Related Havok body-count value | 1,696 | **4,096** |
| Related Havok cleanup threshold | 1,700 | **4,100** |
| Havok contact-point pool | 1,024 | **4,096** |
| Havok allocator block | `0x380000` bytes | **`0xE00000` bytes** |
| Havok budget/accounting constant | `0x271000` | **`0x9C4000`** |
| Havok related scalar | `0x2F9999` | **`0xBE6664`** |
| Restricted-memory total allocation | `0x01C70000` bytes | **`0x0A8F0000` bytes** |
| Restricted-memory size constant repeated through checks | `0x00EB0000` bytes | **`0x071F0000` bytes** |
| Restricted-memory subregion size | `0x420000` | **`0x4870000`** |
| Restricted-memory region size | `0x370000` | **`0xDC0000`** |
| Restricted-memory region size | `0x6E0000` | **`0x1B80000`** |

---

## 1. Actor, script, stimulus, and object-pool limits

For the following rows, replace the full byte sequence at the listed **file offset**.

| Purpose | Starting value → v13 value | VA / instruction address | File offset | Original bytes → v13 bytes |
|---|---|---:|---:|---|
| Main AI actor pool count | 256 → 1,024 | `0x1402809AF` | `0x0027FDAF` | `31 D2 B6 01` → `31 D2 B6 04` |
| Firing-position-owner hash bucket count | 256 → 1,024 | `0x140280AB3` | `0x0027FEB3` | `41 B8 00 01 00 00` → `41 B8 00 04 00 00` |
| AI command-script pool count | 256 → 1,024 | `0x1402AA014` | `0x002A9414` | `31 D2 B6 01` → `31 D2 B6 04` |
| Stimulus-data pool | 512 → 1,024 | `0x14032A43F` | `0x0032983F` | `BA 00 02 00 00` → `BA 00 04 00 00` |
| HS script-thread pool | 640 → 2,024 | `0x14045E262` | `0x0045D662` | `BA 80 02 00 00` → `BA E8 07 00 00` |
| Main object-slot pool count | 4,096 → 20,480 | `0x14109255C` | `0x0109195C` | `BA 00 10 00 00` → `BA 00 50 00 00` |
| Related object-slot initialization count | 4,096 → 20,480 | `0x141092561` | `0x01091961` | `C7 05 29 F5 2E 05 00 10 00 00` → `C7 05 29 F5 2E 05 00 50 00 00` |
| Object backing allocation size | `0x452000` → `0x157C000` bytes | `0x1410925B7` | `0x010919B7` | `BA 00 20 45 00` → `BA 00 C0 57 01` |
| Object render-data backing | `0x4000` → `0x14000` bytes | `0x141092621` | `0x01091A21` | `41 B9 00 40 00 00` → `41 B9 00 40 01 00` |
| Actor debug-buffer allocation | `0xA4000` → `0x290000` bytes | `0x14021DD29` | `0x0021D129` | `B9 00 40 0A 00` → `B9 00 00 29 00` |
| Matching actor debug-buffer clear size | `0xA4000` → `0x290000` bytes | `0x14021ECD1` | `0x0021E0D1` | `B9 00 40 0A 00` → `B9 00 00 29 00` |

### Script object-list settings

| Purpose | Starting value → v13 value | Instruction VA | Immediate VA | File offset to edit | Original bytes → v13 bytes |
|---|---|---:|---:|---:|---|
| Script list-object/reference count | 128 → 1,024 | `0x1405B020E` | `0x1405B0210` | `0x005AF610` | `80 00` → `00 04` |
| Script `object list header` count | 48 → 256 | `0x1405B01C8` | — | `0x005AF5C8` | `41 8D 51 30` → `31 D2 B6 01` |

For the first row, the instruction beginning at `0x1405B020E` is `41 B8 80 00 00 00` in the original and `41 B8 00 04 00 00` in v13. The **immediate itself begins at VA `0x1405B0210`**, which is why that address is often used to identify this setting.

---

## 2. Actor LOD stack-frame expansion

These edits expand the stack frame used by the actor LOD/rating path and move its saved registers and security-cookie references to matching stack locations. Apply the full instruction replacements below.

| Purpose | VA | File offset | Original bytes → v13 bytes | Value change / meaning |
|---|---:|---:|---|---|
| LOD frame base | `0x14036FF08` | `0x0036F308` | `48 8D AC 24 00 F4 FF FF` → `48 8D AC 24 00 D0 FF FF` | Stack displacement `-0xC00` → `-0x3000` |
| Stack allocation | `0x14036FF10` | `0x0036F310` | `48 81 EC 00 0D 00 00` → `48 81 EC 00 31 00 00` | `0xD00` → `0x3100` bytes |
| Security-cookie store offset | `0x14036FF21` | `0x0036F321` | `48 89 85 B0 0B 00 00` → `48 89 85 B0 2E 00 00` | `+0xBB0` → `+0x2EB0` |
| Save XMM6 slot | `0x14036FF5B` | `0x0036F35B` | `0F 29 B4 24 F0 0C 00 00` → `0F 29 B4 24 F0 30 00 00` | `+0xCF0` → `+0x30F0` |
| Save XMM7 slot | `0x14036FF69` | `0x0036F369` | `0F 29 BC 24 E0 0C 00 00` → `0F 29 BC 24 E0 30 00 00` | `+0xCE0` → `+0x30E0` |
| Save XMM8 slot | `0x14036FF7C` | `0x0036F37C` | `44 0F 29 84 24 D0 0C 00 00` → `44 0F 29 84 24 D0 30 00 00` | `+0xCD0` → `+0x30D0` |
| Save XMM9 slot | `0x14036FF8E` | `0x0036F38E` | `44 0F 29 8C 24 C0 0C 00 00` → `44 0F 29 8C 24 C0 30 00 00` | `+0xCC0` → `+0x30C0` |
| Actor-rating loop guard | `0x14036FFAA` | `0x0036F3AA` | `0F 1F 44 00 00 66 41 81 FE 00 01` → `0F 1F 44 00 00 66 41 81 FE 00 04` | Limit `0x100` (256) → `0x400` (1,024) |
| Restore XMM9 slot | `0x140370345` | `0x0036F745` | `44 0F 28 8C 24 C0 0C 00 00` → `44 0F 28 8C 24 C0 30 00 00` | `+0xCC0` → `+0x30C0` |
| Restore XMM8 slot | `0x14037034E` | `0x0036F74E` | `44 0F 28 84 24 D0 0C 00 00` → `44 0F 28 84 24 D0 30 00 00` | `+0xCD0` → `+0x30D0` |
| Restore XMM7 slot | `0x140370357` | `0x0036F757` | `0F 28 BC 24 E0 0C 00 00` → `0F 28 BC 24 E0 30 00 00` | `+0xCE0` → `+0x30E0` |
| Restore XMM6 slot | `0x14037035F` | `0x0036F75F` | `0F 28 B4 24 F0 0C 00 00` → `0F 28 B4 24 F0 30 00 00` | `+0xCF0` → `+0x30F0` |
| Security-cookie load offset | `0x1403704BA` | `0x0036F8BA` | `48 8B 8D B0 0B 00 00` → `48 8B 8D B0 2E 00 00` | `+0xBB0` → `+0x2EB0` |
| Stack restore in epilogue | `0x1403704C9` | `0x0036F8C9` | `4C 8D 9C 24 00 0D 00 00` → `4C 8D 9C 24 00 31 00 00` | `0xD00` → `0x3100` bytes |

### Matching x64 unwind metadata

These metadata entries are also changed to match the expanded stack frame. Replace the two bytes at each file offset.

| Purpose | VA | File offset | Original bytes → v13 bytes |
|---|---:|---:|---|
| Saved-register slot metadata | `0x141F2DE66` | `0x01F2D066` | `A8 01` → `28 06` |
| Saved-register slot metadata | `0x141F2DE6A` | `0x01F2D06A` | `A7 01` → `27 06` |
| Saved-register slot metadata | `0x141F2DE6E` | `0x01F2D06E` | `A6 01` → `26 06` |
| Stack-allocation metadata | `0x141F2DE72` | `0x01F2D072` | `A0 01` → `20 06` |
| XMM9 save-slot metadata | `0x141F2DE8E` | `0x01F2D08E` | `CC 00` → `0C 03` |
| XMM8 save-slot metadata | `0x141F2DE92` | `0x01F2D092` | `CD 00` → `0D 03` |
| XMM7 save-slot metadata | `0x141F2DE96` | `0x01F2D096` | `CE 00` → `0E 03` |
| XMM6 save-slot metadata | `0x141F2DE9A` | `0x01F2D09A` | `CF 00` → `0F 03` |

---

## 3. Restricted-memory allocation values

### 3.1 Total game-state allocation: `0x01C70000` → `0x0A8F0000` bytes

The same total-allocation immediate is changed at seven locations. For each row, overwrite the entire listed byte sequence.

| VA | File offset | Original bytes → v13 bytes |
|---|---:|---|
| `0x1404140BB` | `0x004134BB` | `BA 00 00 C7 01` → `BA 00 00 8F 0A` |
| `0x140414BA7` | `0x00413FA7` | `BA 00 00 C7 01` → `BA 00 00 8F 0A` |
| `0x14041663B` | `0x00415A3B` | `B9 00 00 C7 01` → `B9 00 00 8F 0A` |
| `0x140416676` | `0x00415A76` | `81 C1 00 00 C7 01` → `81 C1 00 00 8F 0A` |
| `0x140416B7A` | `0x00415F7A` | `41 B8 00 00 C7 01` → `41 B8 00 00 8F 0A` |
| `0x140416FFA` | `0x004163FA` | `C7 03 00 00 C7 01` → `C7 03 00 00 8F 0A` |
| `0x14041700D` | `0x0041640D` | `C7 01 00 00 C7 01` → `C7 01 00 00 8F 0A` |

### 3.2 Repeated restricted-memory size/check value: `0x00EB0000` → `0x071F0000`

At every file offset below, replace the two bytes `EB 00` with `1F 07`. These are the changed high bytes of the same little-endian 32-bit value; the other two bytes of the immediate remain `00 00`.

| File offset | VA | File offset | VA | File offset | VA |
|---:|---:|---:|---:|---:|---:|
| `0x00075DAA` | `0x1400769AA` | `0x004121EF` | `0x140412DEF` | `0x0041241E` | `0x14041301E` |
| `0x00412431` | `0x140413031` | `0x004124DE` | `0x1404130DE` | `0x0041381A` | `0x14041441A` |
| `0x00413EEA` | `0x140414AEA` | `0x00413F39` | `0x140414B39` | `0x00414550` | `0x140415150` |
| `0x004146E9` | `0x1404152E9` | `0x004146FE` | `0x1404152FE` | `0x004152B4` | `0x140415EB4` |
| `0x004152EC` | `0x140415EEC` | `0x00415455` | `0x140416055` | `0x004155F3` | `0x1404161F3` |
| `0x0041562B` | `0x14041622B` | `0x00415656` | `0x140416256` | `0x00415A39` | `0x140416639` |
| `0x0073EB46` | `0x14073F746` | `0x0073ED55` | `0x14073F955` | `0x0165016B` | `0x141650D6B` |

### 3.3 Restricted-memory subregion and pointer arithmetic

These changes keep the size constants and their paired pointer/offset arithmetic aligned. Replace the full byte sequence at each offset.

| Purpose | VA | File offset | Original bytes → v13 bytes | Constant change |
|---|---:|---:|---|---|
| Subregion size | `0x1404167F5` | `0x00415BF5` | `41 B8 00 00 42 00` → `41 B8 00 00 87 04` | `0x420000` → `0x4870000` |
| Matching pointer step | `0x14041680A` | `0x00415C0A` | `48 81 C3 00 00 42 00` → `48 81 C3 00 00 87 04` | `0x420000` → `0x4870000` |
| Region-size value | `0x14041686E` | `0x00415C6E` | `41 B8 00 00 37 00` → `41 B8 00 00 DC 00` | `0x370000` → `0xDC0000` |
| Matching address calculation | `0x140416884` | `0x00415C84` | `8D 9F 00 00 37 00` → `8D 9F 00 00 DC 00` | `0x370000` → `0xDC0000` |
| Region-size value | `0x1404168AC` | `0x00415CAC` | `41 B8 00 00 6E 00` → `41 B8 00 00 B8 01` | `0x6E0000` → `0x1B80000` |
| Matching address calculation | `0x1404168C2` | `0x00415CC2` | `8D 9F 00 00 6E 00` → `8D 9F 00 00 B8 01` | `0x6E0000` → `0x1B80000` |
| Region-size value | `0x140416928` | `0x00415D28` | `41 B8 00 00 6E 00` → `41 B8 00 00 B8 01` | `0x6E0000` → `0x1B80000` |
| Matching pointer addition | `0x14041693E` | `0x00415D3E` | `81 C7 00 00 6E 00` → `81 C7 00 00 B8 01` | `0x6E0000` → `0x1B80000` |
| Region-size value | `0x140416960` | `0x00415D60` | `41 B8 00 00 6E 00` → `41 B8 00 00 B8 01` | `0x6E0000` → `0x1B80000` |

---

## 4. Havok-related capacity and allocator changes

### 4.1 Component counts and cleanup thresholds

| Purpose | Starting value → v13 value | VA / immediate location | File offset | Original bytes → v13 bytes |
|---|---|---:|---:|---|
| Havok component maximum check | 1,500 (`0x5DC`) → 3,000 (`0xBB8`) | `0x140504DEC` | `0x005041EC` | `41 81 7D 2C DC 05 00 00` → `41 81 7D 2C B8 0B 00 00` |
| Related Havok pool count | 1,500 (`0x5DC`) → 3,000 (`0xBB8`) | `0x14061CC7D` | `0x0061C07D` | `BA DC 05 00 00` → `BA B8 0B 00 00` |
| Havok body-count value | 1,696 (`0x6A0`) → 4,096 (`0x1000`) | immediate at `0x1405050F6` | `0x005044F6` | `A0 06` → `00 10` |
| Related body-count value | 1,696 (`0x6A0`) → 4,096 (`0x1000`) | immediate at `0x140505138` | `0x00504538` | `A0 06` → `00 10` |
| Related body-count value | 1,696 (`0x6A0`) → 4,096 (`0x1000`) | immediate at `0x140505168` | `0x00504568` | `A0 06` → `00 10` |
| Havok cleanup threshold | 1,700 (`0x6A4`) → 4,100 (`0x1004`) | immediate at `0x140507C15` | `0x00507015` | `A4 06` → `04 10` |

For the last four rows, edit just the listed two immediate bytes at the file offset; do not replace the surrounding bytes.

### 4.2 Havok contact-point pool

| Purpose | Starting value → v13 value | VA | File offset | Original bytes → v13 bytes |
|---|---|---:|---:|---|
| `havok contact points` pool count | 1,024 (`0x400`) → 4,096 (`0x1000`) | `0x14061CCCE` | `0x0061C0CE` | `BA 00 04 00 00` → `BA 00 10 00 00` |

The element size remains `0x50` bytes. Nominal entry storage therefore changes from `0x14000` bytes to `0x50000` bytes.

### 4.3 Havok allocator backing, budgets, and matching checks

| Purpose | VA | File offset | Original bytes → v13 bytes | Value change |
|---|---:|---:|---|---|
| Fixed allocator block size | `0x140A581CA` | `0x00A575CA` | `41 B8 00 00 38 00` → `41 B8 00 00 E0 00` | `0x380000` → `0xE00000` |
| Related allocator scalar | `0x140A5821B` | `0x00A5761B` | `BA 99 99 2F 00` → `BA 64 66 BE 00` | `0x2F9999` → `0xBE6664` |
| Allocation-size validation | `0x140A58627` | `0x00A57A27` | `81 FD 00 00 38 00` → `81 FD 00 00 E0 00` | `0x380000` → `0xE00000` |
| Allocation budget assignment | `0x140A5863F` | `0x00A57A3F` | `48 C7 40 28 00 10 27 00` → `48 C7 40 28 00 40 9C 00` | `0x271000` → `0x9C4000` |
| Deallocation-size validation | `0x140A586F0` | `0x00A57AF0` | `81 FE 00 00 38 00` → `81 FE 00 00 E0 00` | `0x380000` → `0xE00000` |
| Budget reset/release value | `0x140A58708` | `0x00A57B08` | `48 C7 40 28 00 00 38 00` → `48 C7 40 28 00 00 E0 00` | `0x380000` → `0xE00000` |
| Havok accounting/global limit | `0x14078351C` | `0x0078291C` | `C7 05 12 9E 46 04 00 10 27 00` → `C7 05 12 9E 46 04 00 40 9C 00` | `0x271000` → `0x9C4000` |
| Allocator self-test full-size request | `0x1407837D4` | `0x00782BD4` | `41 B8 00 10 27 00` → `41 B8 00 40 9C 00` | `0x271000` → `0x9C4000` |
| Allocator self-test half-size request A | `0x140783801` | `0x00782C01` | `41 B8 00 88 13 00` → `41 B8 00 20 4E 00` | `0x138800` → `0x4E2000` |
| Allocator self-test half-size request B | `0x140783826` | `0x00782C26` | `41 B8 00 88 13 00` → `41 B8 00 20 4E 00` | `0x138800` → `0x4E2000` |
| Allocator self-test near-limit request | `0x14078385D` | `0x00782C5D` | `41 B8 F0 0F 27 00` → `41 B8 F0 3F 9C 00` | `0x270FF0` → `0x9C3FF0` |
| Matching budget guard | `0x140783B5D` | `0x00782F5D` | `81 F9 00 10 27 00` → `81 F9 00 40 9C 00` | `0x271000` → `0x9C4000` |

---

## 5. Values that must remain unchanged

These are intentional non-changes and are **not** part of the route from the original file to v13.

| Purpose | VA | File offset | Bytes/value to preserve |
|---|---:|---:|---|
| Protected free-slot/diagnostic constant | `0x141092461` | `0x01091861` | `B8 00 08 00 00` — keep immediate `0x800` (2,048) |
| Command-script element/structure size | `0x1402AA009` | `0x002A9409` | Preserve `41 B8 EC 01 00 00` — size `0x1EC` |

**Do not change `0x141092461`.** Do not “correct” it to match the object pool, and do not change associated diagnostic arithmetic as part of this set.

## Scope note

This list documents the changes present in v13 compared with the exact starting EXE named at the top. It does not add a separate patch that disables rasterizer visibility, occlusion, or model draw-distance culling; those are not among the byte differences between these two files.
