# Patch inventory — `sapien_stable_object_ai256_v4_stim512_mem2_firingowners256.exe`

**Scope:** This inventory reconstructs the known patch chain from the original/stable Sapien build to the named final executable: inherited manual object/script/thread changes, actor/LOD expansion, stimulus-data increase, the second game-state-memory expansion, and the final firing-position-owner table adjustment.

**Address format:** Virtual addresses (VAs) for the x64 PE with image base `0x140000000`. “Original” means the vanilla/unmodified value recorded in the patch history; “intermediate” records the 128-actor / first-memory-expansion stage where applicable; “final” describes the intended final output.

**Runtime status:** You reported that 256 actors using Guard behavior and firing positions now work without the earlier `debug_index == actor_index` crash. That validates the tested scenario, not every edge case in this executable.

## Important caveats

- `0x141092461` is **not patched** in the final build. It remains `MOV EAX,0x800` (`B8 00 08 00 00`) because changing this diagnostic to `0x1000` was observed to make AI agents die during combat.
- The `mem2` values are experimental: this build changed one subregion size to `0x420000`, total game-state allocation to `0x1C70000`, and persistent game-state size to `0xEB0000`. This inventory documents those edits but does not prove every later fixed region offset was relocated consistently. The successful Guard test is encouraging, but memory-layout integrity has not been independently proven.
- The older experimental change at `0x1402AA009` to `MOV R8D,0x4C` is **not** included as an intended final patch. The command-script pool element size in the decompilation is `0x1EC`; please keep that size intact. The current patch scripts did not independently verify `0x1402AA009` in the finished EXE.
- This report is based on the patch scripts, READMEs, earlier Ghidra notes, and test reports in the conversation. The final `.exe` was not provided as a local file in this turn, so this is a historical patch inventory, not an independent byte-for-byte audit of the complete binary.

## 1. Inherited manual changes: object, command-script, and thread pools

| VA | Purpose | Original → final | Byte / patch detail |
|---|---|---|---|
| `0x14109255C` | Main object-slot pool count | 0x800 (2048) → 0x1000 (4096) | BA 00 08 00 00 -> BA 00 10 00 00 |
| `0x141092561` | Object-pool capacity global DAT_146381A94 | 0x800 (2048) → 0x1000 (4096) | Immediate dword: 00 08 00 00 -> 00 10 00 00 |
| `0x1410925B7` | Object-memory pool size | 0x226000 → 0x452000 | BA 00 60 22 00 -> BA 00 20 45 00 |
| `0x141092621` | Render-data pool sizing | 0x2000 → 0x4000 | 41 B9 00 20 00 00 -> 41 B9 00 40 00 00 |
| `0x141092461` | Free-slot diagnostic constant | 0x800 → 0x800 unchanged | B8 00 08 00 00 (unchanged) |
| `0x1402AA014` | Command-script pool entry count | 0x28 (40) → 0x100 (256) | 41 8D 51 28 -> 31 D2 B6 01 (XOR EDX,EDX; MOV DH,1) |
| `0x14045E262` | Deterministic HS thread pool | 0x140 (320) → 0x280 (640) | BA 40 01 00 00 -> BA 80 02 00 00 |

## 2. Actor capacity, actor-debug storage, and firing-position owners

| VA | Purpose | Original → intermediate → final | Byte / patch detail |
|---|---|---|---|
| `0x1402809AF` | Main actor pool maximum | 0x60 (96) → 0x80 (128) → 0x100 (256) | 41 8D 51 60 -> 31 D2 B6 01 |
| `0x140280AB3` | Firing-position-owner hash bucket count | 0x180 (384) → 0x200 (512), then reduced → 0x100 (256) | 41 B8 80 01 00 00 -> 41 B8 00 02 00 00 -> 41 B8 00 01 00 00 |
| `0x140280ABE` | Firing-position-owner hash table maximum elements | 0x60 (96) → 0x80 (128) → 0x100 (256) | 44 8D 4A 5C -> 44 8D 4A 7C -> 45 8B C8 90 |
| `0x14021ECD1` | Actor-debug info allocation bytes | 0x3D800 (96 * 0xA40) → 0x52000 (128 * 0xA40) → 0xA4000 (256 * 0xA40) | B9 00 D8 03 00 -> B9 00 20 05 00 -> B9 00 40 0A 00 |
| `0x14021DD29` | Actor-debug info clear bytes | 0x3D800 → 0x52000 → 0xA4000 | B9 00 D8 03 00 -> B9 00 20 05 00 -> B9 00 40 0A 00 |

## 3. 256-entry actor LOD stack/frame changes

| VA | Purpose | Original → intermediate → final | Bytes |
|---|---|---|---|
| `0x14036FF08` | LOD stack frame base (RBP) | -0x480 → -0x600 → -0xC00 | 48 8D AC 24 80 FB FF FF -> 48 8D AC 24 00 FA FF FF -> 48 8D AC 24 00 F4 FF FF |
| `0x14036FF10` | LOD stack allocation (RSP) | 0x580 → 0x700 → 0xD00 | 48 81 EC 80 05 00 00 -> 48 81 EC 00 07 00 00 -> 48 81 EC 00 0D 00 00 |
| `0x14036FF21` | LOD security-cookie store offset | +0x430 → +0x5B0 → +0xBB0 | 48 89 85 30 04 00 00 -> 48 89 85 B0 05 00 00 -> 48 89 85 B0 0B 00 00 |
| `0x14036FF5B` | LOD save XMM6 stack slot | 0x570 → 0x6F0 → 0xCF0 | 0F 29 B4 24 70 05 00 00 -> 0F 29 B4 24 F0 06 00 00 -> 0F 29 B4 24 F0 0C 00 00 |
| `0x14036FF69` | LOD save XMM7 stack slot | 0x560 → 0x6E0 → 0xCE0 | 0F 29 BC 24 60 05 00 00 -> 0F 29 BC 24 E0 06 00 00 -> 0F 29 BC 24 E0 0C 00 00 |
| `0x14036FF7C` | LOD save XMM8 stack slot | 0x550 → 0x6D0 → 0xCD0 | 44 0F 29 84 24 50 05 00 00 -> 44 0F 29 84 24 D0 06 00 00 -> 44 0F 29 84 24 D0 0C 00 00 |
| `0x14036FF8E` | LOD save XMM9 stack slot | 0x540 → 0x6C0 → 0xCC0 | 44 0F 29 8C 24 40 05 00 00 -> 44 0F 29 8C 24 C0 06 00 00 -> 44 0F 29 8C 24 C0 0C 00 00 |
| `0x1403704BA` | LOD security-cookie load offset | +0x430 → +0x5B0 → +0xBB0 | 48 8B 8D 30 04 00 00 -> 48 8B 8D B0 05 00 00 -> 48 8B 8D B0 0B 00 00 |
| `0x1403704C9` | LOD epilogue stack restore | 0x580 → 0x700 → 0xD00 | 4C 8D 9C 24 80 05 00 00 -> 4C 8D 9C 24 00 07 00 00 -> 4C 8D 9C 24 00 0D 00 00 |
| `0x14036FFAA` | LOD actor-rating array count guard | 96 (0x60) → 128 (0x80) → 256 (0x100) | 66 0F 1F 44 00 00 66 41 83 FE 60 -> 0F 1F 44 00 00 66 41 81 FE 00 01 |
| `0x140370339` | LOD actor-loop back-edge displacement | Original conditional branch ends with 71 FC FF FF → Retargeted by one byte (70 FC FF FF) → 0F 85 70 FC FF FF | 0F 85 71 FC FF FF -> 0F 85 70 FC FF FF |
| `0x140370345` | Restore XMM9 from matching save slot | 0x540 → 0x6C0 → 0xCC0 | 44 0F 28 8C 24 40 05 00 00 -> 44 0F 28 8C 24 C0 06 00 00 -> 44 0F 28 8C 24 C0 0C 00 00 |
| `0x14037034E` | Restore XMM8 from matching save slot | 0x550 → 0x6D0 → 0xCD0 | 44 0F 28 84 24 50 05 00 00 -> 44 0F 28 84 24 D0 06 00 00 -> 44 0F 28 84 24 D0 0C 00 00 |
| `0x140370357` | Restore XMM7 from matching save slot | 0x560 → 0x6E0 → 0xCE0 | 0F 28 BC 24 60 05 00 00 -> 0F 28 BC 24 E0 06 00 00 -> 0F 28 BC 24 E0 0C 00 00 |
| `0x14037035F` | Restore XMM6 from matching save slot | 0x570 → 0x6F0 → 0xCF0 | 0F 28 B4 24 70 05 00 00 -> 0F 28 B4 24 F0 06 00 00 -> 0F 28 B4 24 F0 0C 00 00 |
| `0x141F2DE66` | Unwind metadata: RDI saved-register slot | 0x5C0 → 0x740 → 0xD40 | B8 00 -> E8 00 -> A8 01 |
| `0x141F2DE6A` | Unwind metadata: RSI saved-register slot | 0x5B8 → 0x738 → 0xD38 | B7 00 -> E7 00 -> A7 01 |
| `0x141F2DE6E` | Unwind metadata: RBX saved-register slot | 0x5B0 → 0x730 → 0xD30 | B6 00 -> E6 00 -> A6 01 |
| `0x141F2DE72` | Unwind metadata: stack allocation size | 0x580 → 0x700 → 0xD00 | B0 00 -> E0 00 -> A0 01 |
| `0x141F2DE8E` | Unwind metadata: XMM9 save slot | 0x540 → 0x6C0 → 0xCC0 | 54 00 -> 6C 00 -> CC 00 |
| `0x141F2DE92` | Unwind metadata: XMM8 save slot | 0x550 → 0x6D0 → 0xCD0 | 55 00 -> 6D 00 -> CD 00 |
| `0x141F2DE96` | Unwind metadata: XMM7 save slot | 0x560 → 0x6E0 → 0xCE0 | 56 00 -> 6E 00 -> CE 00 |
| `0x141F2DE9A` | Unwind metadata: XMM6 save slot | 0x570 → 0x6F0 → 0xCF0 | 57 00 -> 6F 00 -> CF 00 |

### LOD assertion/bounds branches widened for actor indices 128–255

The following R14-index sites formerly compared against 95, then were updated to 127 in the 128-actor patch, and finally changed to an unconditional branch over the assertion sequence with NOP padding in the 256-actor patch:

| VA | Final patch behavior / bytes |
|---|---|
| `0x14036FFC3` | 66 41 83 FE 5F 76 0C -> 66 41 83 FE 7F 76 0C -> EB 11 90 90 90 90 90 |
| `0x14036FFE5` | 66 41 83 FE 5F 76 0C -> 66 41 83 FE 7F 76 0C -> EB 11 90 90 90 90 90 |
| `0x140370000` | 66 41 83 FE 5F 76 0C -> 66 41 83 FE 7F 76 0C -> EB 11 90 90 90 90 90 |
| `0x140370038` | 66 41 83 FE 5F 76 0C -> 66 41 83 FE 7F 76 0C -> EB 11 90 90 90 90 90 |
| `0x140370087` | 66 41 83 FE 5F 76 0C -> 66 41 83 FE 7F 76 0C -> EB 11 90 90 90 90 90 |
| `0x1403700C0` | 66 41 83 FE 5F 76 0C -> 66 41 83 FE 7F 76 0C -> EB 11 90 90 90 90 90 |
| `0x140370232` | 66 41 83 FE 5F 76 0C -> 66 41 83 FE 7F 76 0C -> EB 11 90 90 90 90 90 |
| `0x14037024D` | 66 41 83 FE 5F 76 10 -> 66 41 83 FE 7F 76 10 -> EB 15 90 90 90 90 90 |
| `0x140370266` | 66 41 83 FE 5F 76 0C -> 66 41 83 FE 7F 76 0C -> EB 11 90 90 90 90 90 |
| `0x1403702B0` | 66 41 83 FE 5F 76 0C -> 66 41 83 FE 7F 76 0C -> EB 11 90 90 90 90 90 |
| `0x1403702C7` | 66 41 83 FE 5F 76 0C -> 66 41 83 FE 7F 76 0C -> EB 11 90 90 90 90 90 |
| `0x140370303` | 66 41 83 FE 5F 76 0C -> 66 41 83 FE 7F 76 0C -> EB 11 90 90 90 90 90 |

The five DI-index sites received the same type of assertion bypass:

| VA | Final patch behavior / bytes |
|---|---|
| `0x1403703B1` | 66 83 FF 5F 76 0C -> 66 83 FF 7F 76 0C -> EB 10 90 90 90 90 |
| `0x1403703D8` | 66 83 FF 5F 76 0C -> 66 83 FF 7F 76 0C -> EB 10 90 90 90 90 |
| `0x140370433` | 66 83 FF 5F 76 0C -> 66 83 FF 7F 76 0C -> EB 10 90 90 90 90 |
| `0x140370460` | 66 83 FF 5F 76 0C -> 66 83 FF 7F 76 0C -> EB 10 90 90 90 90 |
| `0x14037047C` | 66 83 FF 5F 76 0C -> 66 83 FF 7F 76 0C -> EB 10 90 90 90 90 |

### Experimental observer guard

The inherited actor-128 v3 patch also modifies `0x140565205` with a same-length 27-byte guard around the reciprocal used by observer velocity. It was carried into the final build, but it was not independently shown to be necessary for the camera repair; the XMM save/restore correction was the proven camera fix.

| VA | Original → final bytes |
|---|---|
| `0x140565205` | `F3 0F 10 00 0F 5A C0 F2 0F 5E C8 66 44 0F 5A D1 66 66 66 0F 1F 84 00 00 00 00 00 -> F3 0F 10 00 45 0F 57 D2 41 0F 2E C7 76 0D 0F 5A C0 F2 0F 5E C8 66 44 0F 5A D1 90` |

## 4. Stimulus-data pool

| VA | Purpose | Original → final | Bytes |
|---|---|---|---|
| `0x14032A43F` | stimulus-data pool count | 0x100 (256) → 0x200 (512) | BA 00 01 00 00 -> BA 00 02 00 00 |

The separate `stimulus-ref` pool at `0x14032A49D` remains `0x400` (1,024 entries); it was checked by the patch scripts but not modified.

## 5. Game-state / restricted-memory constants

### Subregion (2 addresses)

| VA | Purpose | Original → first expansion → final | Bytes |
|---|---|---|---|
| `0x1404167F5` | Subregion allocation size | 0x220000 → 0x320000 → 0x420000 | 41 B8 00 00 22 00 -> 41 B8 00 00 32 00 -> 41 B8 00 00 42 00 |
| `0x14041680A` | Pointer/region arithmetic addition | 0x220000 → 0x320000 → 0x420000 | 48 81 C3 00 00 22 00 -> 48 81 C3 00 00 32 00 -> 48 81 C3 00 00 42 00 |

### Total game-state allocation (7 addresses)

| VA | Purpose | Original → first expansion → final | Bytes |
|---|---|---|---|
| `0x1404140BB` | Total game-state allocation constant | 0x1A70000 → 0x1B70000 → 0x1C70000 | BA 00 00 A7 01 -> BA 00 00 B7 01 -> BA 00 00 C7 01 |
| `0x140414BA7` | Total game-state allocation constant | 0x1A70000 → 0x1B70000 → 0x1C70000 | BA 00 00 A7 01 -> BA 00 00 B7 01 -> BA 00 00 C7 01 |
| `0x14041663B` | Total game-state allocation constant | 0x1A70000 → 0x1B70000 → 0x1C70000 | B9 00 00 A7 01 -> B9 00 00 B7 01 -> B9 00 00 C7 01 |
| `0x140416676` | Total game-state allocation constant | 0x1A70000 → 0x1B70000 → 0x1C70000 | 81 C1 00 00 A7 01 -> 81 C1 00 00 B7 01 -> 81 C1 00 00 C7 01 |
| `0x140416B7A` | Total game-state allocation constant | 0x1A70000 → 0x1B70000 → 0x1C70000 | 41 B8 00 00 A7 01 -> 41 B8 00 00 B7 01 -> 41 B8 00 00 C7 01 |
| `0x140416FFA` | Total game-state allocation constant | 0x1A70000 → 0x1B70000 → 0x1C70000 | C7 03 00 00 A7 01 -> C7 03 00 00 B7 01 -> C7 03 00 00 C7 01 |
| `0x14041700D` | Total game-state allocation constant | 0x1A70000 → 0x1B70000 → 0x1C70000 | C7 01 00 00 A7 01 -> C7 01 00 00 B7 01 -> C7 01 00 00 C7 01 |

### Persistent game-state size (21 addresses)

| VA | Purpose | Original → first expansion → final | Bytes |
|---|---|---|---|
| `0x1400769A6` | CMP RAX against persistent size | 0xCB0000 → 0xDB0000 → 0xEB0000 | 48 3D 00 00 CB 00 -> 48 3D 00 00 DB 00 -> 48 3D 00 00 EB 00 |
| `0x140412DE9` | Stack argument persistent size | 0xCB0000 → 0xDB0000 → 0xEB0000 | C7 44 24 20 00 00 CB 00 -> C7 44 24 20 00 00 DB 00 -> C7 44 24 20 00 00 EB 00 |
| `0x14041301A` | Store persistent size | 0xCB0000 → 0xDB0000 → 0xEB0000 | C7 03 00 00 CB 00 -> C7 03 00 00 DB 00 -> C7 03 00 00 EB 00 |
| `0x14041302D` | Store persistent size | 0xCB0000 → 0xDB0000 → 0xEB0000 | C7 01 00 00 CB 00 -> C7 01 00 00 DB 00 -> C7 01 00 00 EB 00 |
| `0x1404130DA` | R9D persistent size | 0xCB0000 → 0xDB0000 → 0xEB0000 | 41 B9 00 00 CB 00 -> 41 B9 00 00 DB 00 -> 41 B9 00 00 EB 00 |
| `0x140414416` | R8D persistent size | 0xCB0000 → 0xDB0000 → 0xEB0000 | 41 B8 00 00 CB 00 -> 41 B8 00 00 DB 00 -> 41 B8 00 00 EB 00 |
| `0x140414AE7` | CMP EAX against persistent size | 0xCB0000 → 0xDB0000 → 0xEB0000 | 3D 00 00 CB 00 -> 3D 00 00 DB 00 -> 3D 00 00 EB 00 |
| `0x140414B35` | R9D persistent size | 0xCB0000 → 0xDB0000 → 0xEB0000 | 41 B9 00 00 CB 00 -> 41 B9 00 00 DB 00 -> 41 B9 00 00 EB 00 |
| `0x14041514D` | EDX persistent size | 0xCB0000 → 0xDB0000 → 0xEB0000 | BA 00 00 CB 00 -> BA 00 00 DB 00 -> BA 00 00 EB 00 |
| `0x1404152E5` | R8D persistent size | 0xCB0000 → 0xDB0000 → 0xEB0000 | 41 B8 00 00 CB 00 -> 41 B8 00 00 DB 00 -> 41 B8 00 00 EB 00 |
| `0x1404152FA` | R9D persistent size | 0xCB0000 → 0xDB0000 → 0xEB0000 | 41 B9 00 00 CB 00 -> 41 B9 00 00 DB 00 -> 41 B9 00 00 EB 00 |
| `0x140415EB1` | EDX persistent size | 0xCB0000 → 0xDB0000 → 0xEB0000 | BA 00 00 CB 00 -> BA 00 00 DB 00 -> BA 00 00 EB 00 |
| `0x140415EE9` | EDX persistent size | 0xCB0000 → 0xDB0000 → 0xEB0000 | BA 00 00 CB 00 -> BA 00 00 DB 00 -> BA 00 00 EB 00 |
| `0x140416051` | R9D persistent size | 0xCB0000 → 0xDB0000 → 0xEB0000 | 41 B9 00 00 CB 00 -> 41 B9 00 00 DB 00 -> 41 B9 00 00 EB 00 |
| `0x1404161F0` | EDX persistent size | 0xCB0000 → 0xDB0000 → 0xEB0000 | BA 00 00 CB 00 -> BA 00 00 DB 00 -> BA 00 00 EB 00 |
| `0x140416228` | EDX persistent size | 0xCB0000 → 0xDB0000 → 0xEB0000 | BA 00 00 CB 00 -> BA 00 00 DB 00 -> BA 00 00 EB 00 |
| `0x140416253` | EDX persistent size | 0xCB0000 → 0xDB0000 → 0xEB0000 | BA 00 00 CB 00 -> BA 00 00 DB 00 -> BA 00 00 EB 00 |
| `0x140416636` | EDX persistent size | 0xCB0000 → 0xDB0000 → 0xEB0000 | BA 00 00 CB 00 -> BA 00 00 DB 00 -> BA 00 00 EB 00 |
| `0x14073F742` | CMP EDX against persistent size | 0xCB0000 → 0xDB0000 → 0xEB0000 | 81 FA 00 00 CB 00 -> 81 FA 00 00 DB 00 -> 81 FA 00 00 EB 00 |
| `0x14073F952` | ECX persistent size | 0xCB0000 → 0xDB0000 → 0xEB0000 | B9 00 00 CB 00 -> B9 00 00 DB 00 -> B9 00 00 EB 00 |
| `0x141650D68` | ECX persistent size | 0xCB0000 → 0xDB0000 → 0xEB0000 | B9 00 00 CB 00 -> B9 00 00 DB 00 -> B9 00 00 EB 00 |

## 6. Counts and summary

- Inventory rows: **84** address-level modifications or explicitly preserved sites.
- Main actor pool: **96 → 128 → 256**.
- Actor firing-position-owner table: original **384 buckets / 96 elements**, intermediate **512 / 128**, final **256 / 256**.
- Actor-debug storage: **0x3D800 → 0x52000 → 0xA4000** bytes at allocation and clear sites.
- Stimulus-data pool: **256 → 512**; stimulus-ref remains **1,024**.
- Command-script pool: **40 → 256** (inherited manual change). HS script threads: **320 → 640** (inherited manual change).
- Object pool: **2,048 → 4,096**; object-memory pool **0x226000 → 0x452000**; render-data pool **0x2000 → 0x4000**.
- Game-state memory: subregion **0x220000 → 0x320000 → 0x420000**; total **0x1A70000 → 0x1B70000 → 0x1C70000**; persistent **0xCB0000 → 0xDB0000 → 0xEB0000**.

## 7. Sites that are not changes in the final build

- `0x141092461`: preserved `B8 00 08 00 00` (`MOV EAX,0x800`).
- `0x14032A49D`: stimulus-ref count remains `BA 00 04 00 00` (`MOV EDX,0x400`, 1,024 entries).
- `0x1402AA009`: not considered a final modification; do not leave an experimental `MOV R8D,0x4C` here without checking the current EXE, because this argument is the command-script element-size argument in the pool initializer.

## Files

- This Markdown file is the readable narrative inventory.
- The companion CSV has one row per address, with original/intermediate/final values, byte changes, status, and notes, so it can be filtered/sorted.
