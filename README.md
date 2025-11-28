# Windows Storage Pool — SPCACHE Cache Structure Analysis

> Purpose: To organize the analysis of SPCACHE-related structures in Microsoft Storage Spaces, making it easy to share and collaborate.

---

## Overview

This document includes:

- Original C structure definitions (provided by contributors)
- Parsing examples and diagnostic methods (how to locate SPSLOT, SPCACHE, and cache records)
- Collaboration suggestions and contribution guidelines (how to continue improving on GitHub)

> Note: All offsets and values are assumed to be **little-endian**, as is common in Windows disk structures. Please attach hexdumps if you have different samples.

---

## Original Structures (Provided by Contributors)

```c
typedef struct tag_WindowsStoragePoolSPCache
{
    char cSignature[8];                        // SPCACHE
    DT_GUID gStorageSpaceGUID;                 // Storage space GUID
    dt_u32 u32Version;                         // ?? Version, usually 0x00000001
    dt_u32 u32HeaderSize;                      // SPCACHE header size (0x00000060)
    dt_u32 u32Unknown_1;                       // Unknown1 00 00 00 00
    dt_u32 u32CRC32;                           // Structure CRC32 checksum
    dt_u64 u64Index;                           // Index?
    dt_u32 u32SPSlotOffice;                    // First offset of SPSlot (bytes)
    dt_u32 u32Unknown_2;                       // Unknown2 00 00 00 00
    dt_u32 u32SlotSize;                        // Size of each slot (block) in bytes
    dt_u32 u32UnknownSize1;                    // Unknown size1 1024
    dt_u64 u64Unknown_3;                       // Unknown3 00 20 40 00 00 00 00 00
    dt_u32 u32UnknownSize2;                    // Unknown size2
    dt_u32 u32Unknown_4;                        // Unknown4 02 00 00 00
    dt_u32 u32CacheBlockStartOffset;            // Cache block start offset (bytes)
    dt_u32 u32Unknown_5;                        // Unknown5 00 00 00 00
    dt_u32 u32SPCacheSingleBlockSizeByte;        // Size of a single SPCache block (bytes)
    dt_u32 u32UnknownSize3;                    // Unknown size3
} WINDOWSSTORAGEPOOLSPCACHE, *LPWINDOWSSTORAGEPOOLSPCACHE;

typedef struct tag_WindowsStoragePoolSPSlot
{
    char cSignature[8];                    // SPSLOT
    DT_GUID gStorageSpaceGUID;            // Storage space GUID
    dt_u32 u32Version;                    // 0x00000001
    dt_u32 u32Size;                        // SPSLOT structure size (0x00000040)
    dt_u32 u32Unknown_1;
    dt_u32 u32CRC32;                        // Structure CRC32 checksum
    dt_u64 u64Index;                        // Index?
    dt_u64 u64TotalCacheBlockRecords;    // Total number of cache block records
} WINDOWSSTORAGEPOOLSPSLOT, *LPWINDOWSSTORAGEPOOLSPSLOT;

typedef struct tag_WindowsStoragePool_CacheBlockRecord
{
    dt_u64 u64OffsetByte;                // Offset of the cache block in the storage space
    dt_u32 u32ID;                        // Cache block record ID
    dt_u16 u16Type;                        // 03/02. If 02, skip additional 8 unknown bytes
    dt_u16 u16Unknown;
} WINDOWSSTORAGEPOOL_CACHEBLOCKRECORD, *LPWINDOWSSTORAGEPOOL_CACHEBLOCKRECORD;
```

---

## Parsing Examples and Debugging Tips

1. **Find signatures**: Use `xxd`/`hexdump`/`grep -a` to search for `SPCACHE` and `SPSLOT` signatures (may include NUL terminators).
2. **Associate GUIDs**: Multiple files of the same storage space should contain the same GUID to correlate related entries.
3. **CRC check**: Use tools to verify CRC32.
4. **Handle Type==0x0002 records**: If `u16Type==2`, the following 8 bytes are extension fields.
5. **Sample comparison**: Collect SPCACHE/SPSLOT samples across different Windows versions and pool configurations to analyze variations in unknown fields, gradually deducing their purpose (e.g., UnknownSize1 is often 1024, possibly related to internal bitmap or counters).

---

## Collaboration Suggestions

- Recommended repository structure:
  - `README.md` (overview)
  - `spec/SPCACHE.md` (detailed structure)
  - `samples/` (store sanitized hexdump samples)
  - `CONTRIBUTING.md` (guidelines for contributions and sample format)

- Submit samples **sanitized**, keeping only raw binary or hexdump.
- Provide Windows version, Storage Spaces configuration, and sample offset in issue templates.

---

## License and Disclaimer

For research and reverse engineering collaboration only. Ensure you have legal access to the target data.
