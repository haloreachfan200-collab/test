# Halo Reach MCC V16 DLL vs. Sapien v13: renderer/visibility comparison

## Files verified

- `haloreach.dll` is the **V16-patched DLL**, not the stock input described in the guide. Its SHA-256 is `b8810c3b9822f5d6563439fc6b8b16c2def6cd7a6f660dd3775c42edcd0a5f24`, matching `GUIDE.md` exactly.
- Sapien v13 executable: `sapien_v13_obj20480_threads2024_havok4x_contactpoints4096_listheaders256(1).exe`, 40,796,912 bytes, image base `0x140000000`, `SizeOfImage 0x066A2000`, SHA-256 `5e567ee7680936b2bc087262c1a94cdae707d493742ebf67cd706ae5f246fcdf`.
- Sapien v13 decompilation: `sapien_v13_obj20480_threads2024_havok4x_contactpoints4096_listheaders256.c`, SHA-256 `b262a7df9efb62ef2b979bba0b12ca3366770c76330a15890c1b8524f4a22389`.
- DLL image base is `0x180000000`, with `SizeOfImage 0x1B0A2000`. **DLL RVAs/file offsets are not Sapien EXE addresses.** The correspondence below is semantic and structural, not a byte-for-byte cross-module address match.

## Strongest match: visible-list and render-mesh arrays

This is the clearest cross-file match found so far.

### Sapien v13

`FUN_140841840` at VA `0x140841840` initializes two adjacent arrays in one structure:

| Field | Relative location in the structure | Current capacity | Record size | Record start |
|---|---:|---:|---:|---:|
| First list | count at `+0x18` | `0x400` = 1,024 | `0x10` = 16 bytes | `+0x1C` |
| Second list (mesh-shaped records) | count at `+0x4038` | `0x400` = 1,024 | `0xA8` = 168 bytes | `+0x4040` |

Confirmed by the v13 binary disassembly:

- `0x14084185C`: `MOV R15D,0x400`
- `0x140841879`: writes that count to `[RCX+0x18]`
- `0x140841885–0x140841895`: initializes 1,024 entries, advancing by `0x10`
- `0x1408418B3`: writes the same count to `[RBX+0x4038]`
- `0x1408418C2–0x1408418D5`: initializes 1,024 entries, advancing by `0xA8`

### DLL V16

The V16 guide describes the corresponding **visible list and mesh records** as 1,024 → 4,096 entries each. The DLL constructor code at VA `0x18014EA9C` confirms the same two record layouts:

- Capacity is set to `0x1000` (4,096).
- First record array advances by `0x10` per entry.
- Second record array advances by `0xA8` per entry.
- The constructor then jumps to appended code so the rest of initialization remains coordinated.

The DLL relocates the backing storage rather than just enlarging the inline arrays:

| Structure | DLL stock RVA | DLL V16 RVA |
|---|---:|---:|
| Visible-list header | `0x0291C7B0` | `0x12BE9000` |
| Mesh header | `0x029207C0` | `0x12BF9020` |

Those new bases preserve the exact header/entry geometry: the second header begins `0x10020` bytes after the first (`0x20` header spacing plus `4096 × 0x10` bytes of visible-list entries). This is the same layout relationship as the Sapien v13 constructor, whose second header begins at structure offset `0x4020` (`0x20 + 1024 × 0x10`).

**Assessment:** high-confidence structural match. The v13 EXE still has 1,024-entry arrays with the same entry sizes and header offsets. This is a much more relevant candidate for the visible-biped problem than the earlier render-state-cache-only test. The DLL change is not just one capacity immediate: its backing arrays are moved, many references are redirected, and a constructor continuation hook handles the expanded layout.

A direct change of `0x400` to `0x1000` in the EXE would be unsafe: the present second header and numerous following fields are located directly after the original 1,024-entry arrays. They would be overwritten by expanded entries unless the arrays and references are relocated together.

## Confirmed match: cached object render states

Sapien v13 `FUN_140B78ED0` and DLL V16 use the same named pool and the same record size:

| Setting | Sapien v13 | DLL V16 |
|---|---:|---:|
| `cached object render states` count | `0x400` = 1,024 | `0x1000` = 4,096 |
| Record size | `0x270` = 624 bytes | `0x270` = 624 bytes |
| Manager pointer TLS offset | `0x970` | Same manager pattern in the corresponding initialization path |

Sapien v13's instructions are at `0x140B78EF9` (`MOV EDX,0x400`) and `0x140B78F03` (`MOV R8D,0x270`). This is the same pool already increased in the v18 experiment. Since that test did not improve visibility, the count alone is unlikely to explain the symptom.

The DLL patch list also grows a nearby 0x60000-byte render-initialization allocation to 0x180000. Sapien v13's `FUN_140B4B670` has a 0x60000-byte `VirtualAlloc` and restricted-section registration immediately before it calls `FUN_140B78ED0`; its embedded debug marker names `c_render_primitive_list::initialize_for_game()`. Thus, these allocations are in the same rendering initialization sequence, although the DLL guide's label “render-state auxiliary heap” should not be treated as a definitive class-name match without tracing the DLL's call targets. The user already tested a larger version of the corresponding 0x60000-byte Sapien allocation in v20 without a visibility improvement.

## Other DLL V16 changes that matter for the rendering symptom

These are strong leads, but I have not yet established a one-to-one Sapien function/address match for every one:

1. **Primary visibility scratch:** 2,048 → 32,768 slots. The DLL also expands object-indexed spatial visited bitmaps from 256 to 4,096 bytes. Its patch updates multiple stack frames, indexed offsets, clear lengths, saved-register locations, GS-cookie locations, page probes and unwind metadata. This is likely relevant to visibility traversal/culling and cannot be ported by changing a bitmap length alone.
2. **Per-frame CPU render packets:** 220,160 bytes (`0x35C00`) → 1,048,576 bytes (`0x100000`). The packet backing is relocated. Offset masks change from `0x3FFFF` to `0xFFFFF` at ten decoders, and the generation shift changes from 18 to 20 bits. The mask, encoders/decoders, generation format, backing, and allocation/reset sizes must remain coordinated.
3. **Render-message queue:** 2,048 → 32,768 physical records. The visited bitmap grows from 256 to 4,096 bytes, the record/data offset changes from `0x10C` to `0x100C`, and producers, consumers, merge/reset, and delete-time invalidation paths are updated. This could affect high-volume render-state updates; it is not equivalent to the cached-state pool.
4. **Object pool and render data:** DLL V16 raises shared object header slots 2,048 → 32,767, body heap 2,252,800 → 36,044,800 bytes, and object render-data array 8,192 → 131,072 bytes. Sapien v13 already has a 20,480-slot object pool, so this is not the closest explanation for disappearing models at roughly 100 visible bipeds, though render-data and invalidation structures remain worth comparing.
5. **Physics, scheduler and other arenas:** V16 has extensive Havok, collision, scheduler, checkpoint, and memory-arena changes. These may be needed for large-scale simulation, but they are not the first candidates for a rendering-only disappearance issue. They should not be copied wholesale into the EXE without separately mapping their layouts.

## How this changes the next test

The experiments already run in Sapien tested a render-state count expansion and a larger 0x60000-byte rendering allocation; neither fixed de-rendering. They did **not** test the DLL's coordinated 1,024 → 4,096 visible-list and mesh-record relocation described above.

The most evidence-backed next port to investigate is therefore:

1. Map the Sapien v13 `FUN_140841840` instance and all readers/writers of both arrays.
2. Allocate new backing for 4,096 16-byte visible records and 4,096 168-byte mesh records, keeping the headers and following fields separate from the old inline storage.
3. Update the constructor and every accessor/reference to the two arrays, plus relevant capacity/bounds checks. Preserve the current v13 object/AI/script/Havok changes and avoid changing `0x141092461`.
4. In a separate experiment, map the DLL's primary visibility scratch and render-packet changes to their Sapien counterparts; these may be complementary limits.

The V16 DLL's success with 800+ visible bipeds is evidence that its **combined patch set** supports that workload. It does not prove the list/mesh expansion alone is responsible, but the exact record-layout match makes it the best first candidate to port. Do not copy DLL RVAs, file offsets, or appended code directly into the EXE; the images have different bases, section layouts, and surrounding code.

## Later Sapien runtime evidence and porting constraint

The user subsequently tested the V3-derived packet-only and render-state-only isolation builds. Neither fixed the repeated null-write crash at Sapien VA `0x14152D3C9`; the game-tick-2,150 report is specifically from the render-state-only build, which retains the V3 1 MiB packet arena and 20/12 handle encoding. The combined V4 build also crashed, so the DLL's coordinated packet/cache changes (and their 2x V4 versions) are not by themselves a demonstrated crash fix. This does not show those changes are unnecessary: V1/V3 already carry the 1 MiB packet, 4,096 cached states, and `0x180000` paired auxiliary allocation/registration, while the isolation tests only establish that doubling either group did not fix this observed fault.

The user's standing instruction is broader than diagnosing this crash: map each behavior in the DLL V16 patch set to a corresponding Sapien implementation and carry over an equivalent change unless it would reduce a Sapien value below the current V3 value. This ledger is evidence of the DLL-side coordinated change set, not proof that every DLL overwrite should be copied literally or that every subsystem change causes the Sapien fault. In particular:

- `ALL_OVERWRITES.md` rows 278–281 and 302 include packet-backing references, the reset/allocation size, and generation shift; rows 282–283 are the paired auxiliary allocation and registration; row 284 is the cached-state capacity. V1/V3 already port these groups to Sapien.
- The visible-list/mesh relocation is structurally matched to `FUN_140841840`, but the v17/v21 Sapien attempts produced all-model invisibility. Do not reuse those implementations. A fresh mapping must audit every initializer, writer, reader, index bound, and relocated reference before producing another test build.
- The render-message queue and visibility scratch are distinct structures from the cached-state pool. Their DLL layouts and coordinated edits are documented above and in the subsystem notes; a Sapien equivalent still needs function-by-function confirmation.
- Preserve Sapien's higher current values and the protected VA `0x141092461`; do not transplant DLL addresses or lower a capacity just to match DLL V16.

Accordingly, the next evidence-based step is to map remaining DLL renderer structures to Sapien and obtain crash-time caller/record/tag context. The direct null-store trap at `FUN_14152D360` identifies the invalid record tag but not which producer or dependency corrupted it. The game-tick-2,150 build identity is now known at the variant level; its file hash and visible-biped count remain unavailable.

The follow-up mapping pass is documented in [`docs/reverse-engineering/RENDERER_GROUP_MAPPING.md`](docs/reverse-engineering/RENDERER_GROUP_MAPPING.md). It separates the render-message queue from the input object-update queue, global scheduler queues, and per-call object-job scratch. It also records that the supplied `FUN_14152D360` excerpt's later opcode switch (codes through `0x1F`) is distinct from the earlier packet-state scan (states 0–3).
