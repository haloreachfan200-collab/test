# Sapien v13 — selected Halo Reach DLL V16 renderer changes

## Important scope note

This executable is a **first-stage, partial port**, not a complete port of every applicable change in `haloreach.dll` V16. It should be treated as a controlled test candidate. It does **not** yet implement the central visible-render-list/mesh-record relocation (1,024 → 4,096), nor the coordinated visibility scratch/stack-frame, render-message queue, input queue, scheduler, physics/Havok, collision, state-bank/checkpoint, or deletion-recording changes. Therefore, there is no basis to promise that this version fixes the approximately 108-visible-biped problem.

The DLL manifest contains 642 overwrite rows across many coordinated subsystems. DLL addresses and bytes were not copied directly into the EXE.

## What this build changes

- CPU per-frame render-packet arena: `0x35C00` (220,160 bytes) → `0x100000` (1 MiB).
- Packet offset handle field: 18 → 20 bits (`0x3FFFF` → `0xFFFFF`); generation field is correspondingly narrowed from 14 to 12 bits. Allocator, capacity helpers, reset/clear size, encoder, decoder masks/shifts, and the two direct backing-store references were updated together.
- Adds a zero-initialized, read/write PE section `.v16pkt` at RVA `0x066A2000` / VA `0x1466A2000`, with virtual size 1 MiB and no raw file data. The executable file itself does not grow.
- Cached object render-state pool: `0x400` (1,024) → `0x1000` (4,096) entries.
- Associated render-state auxiliary allocation and registration: `0x60000` → `0x180000` bytes.
- Preserves the 20,480-byte render-packet priority reserve.
- Preserves Sapien's existing 20,480-object pool rather than lowering it to the DLL's smaller count.
- Verifies that VA `0x141092461` and the next 32 bytes remain unchanged.

The cached-state and auxiliary-allocation changes are included even though earlier isolated tests did not resolve visibility. Their value as part of the complete coordinated port remains unproven.

## Identity and validation

Input executable:

- `sapien_v13_obj20480_threads2024_havok4x_contactpoints4096_listheaders256(1).exe`
- Size: 40,796,912 bytes
- SHA-256: `051b5268d926f0b8cf28f55f83c34d8761eed6960631f676521971b58b7d7d08`

Output executable:

- `sapien_v13_DllV16_render_infrastructure_v1_partial.exe`
- Size: 40,796,912 bytes
- SHA-256: `6ddeea5d3cf73af667cdfc5449116cd642f9b5e72054d0c807c157529d7a46fb`
- PE section table was parsed by `objdump`; `.v16pkt` is present. The output file's patched operand bytes, packet-buffer RIP targets, section count, `SizeOfImage`, output hash, and protected AI bytes are checked by the patching script.

The input hash is intentionally hard-coded in the script. It refuses to patch a different executable and refuses to overwrite the input file.

## Rebuild the same patch

Keep the original executable and the Python script in the same folder, open a terminal in that folder, then run:

```powershell
python patch_sapien_dllv16_render_port.py "sapien_v13_obj20480_threads2024_havok4x_contactpoints4096_listheaders256(1).exe" "sapien_v13_DllV16_render_infrastructure_v1_partial.exe" --report "sapien_v13_DllV16_render_infrastructure_v1_report.json"
```

The script validates original bytes for every edit. If a check fails, it exits rather than forcing the patch onto an unknown build.

## Important next work

The next priority is still the DLL's visible render list and mesh-record arrays, then visibility scratch/bitmap frames and the render-message queue. Those changes relocate storage or change structure layouts; simply changing capacity immediates would risk overwriting adjacent fields or violating bounds. They need Sapien-specific reference mapping, including all readers/writers and any stack/unwind metadata, before they can be safely included.
