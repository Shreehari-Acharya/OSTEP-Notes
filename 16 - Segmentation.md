# Segmentation

## Virtual Memory Segmentation:

- Instead of having just one base and bounds pair for the entire address space, segmentation uses multiple base and bounds pairs - one for each logical segment of the address space (code, stack, heap, etc.)
- This allows the OS to place each segment in different parts of physical memory, avoiding wasting physical memory for unused virtual address space between segments.
- The hardware MMU has a set of segment registers, with each register containing a base address, size/bounds, and other information for a segment.

## Explicit Segment Selection:

- One approach is to use the top few bits of the virtual address to determine which segment it refers to.
- For example, if there are 3 segments (code, heap, stack), 2 bits can be used:
    - 00 = code segment
    - 01 = heap segment
    - 11 = stack segment
- The remaining bits represent the offset into that segment.

## Translating a Virtual Address:

1. Extract the segment selector bits from the virtual address
- Segment = (VirtualAddress & SEG_MASK) >> SEG_SHIFT
1. Get the offset into the segment
- Offset = VirtualAddress & OFFSET_MASK
1. Check if offset is within segment bounds
- if (Offset >= Bounds[Segment])
RaiseException(PROTECTION_FAULT)
1. If within bounds, calculate physical address
- PhysAddr = Base[Segment] + Offset

## Example:

- Virtual Address = 0x4200 (14-bit address space)
- It's in binary: 01 0000 1101 0000
- Top 2 bits (01) indicate heap segment
- Offset is the remaining 12 bits: 0000 1101 0000 = 0x0D0 = 208
- Assuming heap segment base is 0x8800:
    - PhysicalAddr = 0x8800 + 208 = 0x88D0

## Sharing and Protection:

- Protection bits can allow shared read-only code segments across processes
- Extra hardware checks if a segment access (read/write/execute) is permitted

## Stack Segment:

- Stack grows backwards (towards lower addresses)
- Negative offsets are calculated by:
    - Offset = MaxSegmentSize - PositiveOffset

## Issues with Segmentation:

- External fragmentation due to variable segment sizes
- Not flexible enough for sparse address spaces (entire sparsely-used segment must reside in memory)

## The key formulas/methods are:

1. Determining segment from virtual address bits
2. Calculating offset into segment
3. Checking if offset is in bounds
4. Adding base address and offset to get physical address
5. Handling negative offsets for stack segments

## Questions & Answers:
I couldn't do it :(
