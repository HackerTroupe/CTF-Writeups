**Type:** Reversing  
**Level:** 3/3  
**Author:** Minerva-007  
**Solved by:** LUZ1LF3R

**Description:**  
Return oriented programming is one of the paradigms of all time. The garbled output is 94 7 d4 64 7 54 63 24 ad 98 45 72 35

**Files:** chal.out

**Plan:**  
We are given a ELF 64-bit binary. From the description, it has something to do with ROP and we are given a hexadecimal output. So our goal is to reverse the output to get the flag by analzying the chal.out. 

**Notes:**  
After trying out various commands and script I came to a conclusion that - The challenge says "Return oriented programming is one of the paradigms of all time" and gives us a specific output we need to achieve. Maybe the solution isn't about exploiting the binary, but about understanding how to arrange the operations to get the right result. So I crafted this solver:

```python
#!/usr/bin/env python3
"""
Solver for 'stacked' (ImaginaryCTF 2025).
The binary applies transformations (off/eor/rtr) on bytes of the flag,
storing outputs at each 'inc' step. We reverse those operations to 
reconstruct the original flag.
"""

# === Forward operations (from binary) ===
def eor(x): 
    return x ^ 0x69                 # XOR with 0x69

def rtr(x): 
    return ((x >> 1) | (x << 7)) & 0xFF  # Rotate right by 1 bit

def off(x): 
    return (x + 0x0f) & 0xFF        # Add offset (0x0f), wrap to 8-bit

# === Reverse operations (undo transformations) ===
def rev_eor(x): 
    return x ^ 0x69                 # XOR is symmetric, same as forward

def rev_rtr(x): 
    return ((x << 1) | (x >> 7)) & 0xFF  # Rotate left by 1 bit

def rev_off(x): 
    return (x - 0x0f) & 0xFF        # Subtract offset, wrap to 8-bit


# Sequence of operations extracted from the binary
operations = [
    'off', 'eor', 'rtr', 'inc', 'eor', 'eor', 'eor', 'inc',
    'rtr', 'off', 'rtr', 'inc', 'rtr', 'rtr', 'eor', 'inc',
    'eor', 'eor', 'eor', 'inc', 'rtr', 'off', 'rtr', 'inc',
    'rtr', 'eor', 'rtr', 'inc', 'rtr', 'rtr', 'eor', 'inc',
    'rtr', 'off', 'eor', 'inc', 'eor', 'eor', 'rtr', 'inc',
    'off', 'off', 'rtr', 'inc', 'rtr', 'rtr', 'eor', 'inc',
    'eor', 'off', 'rtr', 'inc', 'off', 'eor', 'off', 'inc'
]

# Target bytes we need to match (garbled output from challenge)
target = [0x94, 0x07, 0xd4, 0x64, 0x07, 0x54, 0x63, 0x24, 0xad, 0x98, 0x45, 0x72, 0x35]


def solve_system():
    """
    Work backwards through the transformation chain.
    Each 'inc' marks where a byte was stored → split into segments
    and reverse each segment independently to recover flag chars.
    """
    # Step 1: Find indices of 'inc' ops
    inc_positions = [i for i, op in enumerate(operations) if op == 'inc']
    print(f"Inc positions: {inc_positions}")

    # Step 2: Split operations into per-character segments
    segments, start = [], 0
    for pos in inc_positions:
        segments.append(operations[start:pos])  # segment before each inc
        start = pos + 1
    segments.append(operations[start:])  # leftover after last inc

    print(f"Number of segments: {len(segments)}")

    required_inputs = []  # original flag bytes

    # Step 3: Work backwards through each segment
    for i, segment in enumerate(segments):
        if i >= len(target):
            break

        target_value = target[i]
        print(f"\nSegment {i}: {segment}")
        print(f"Target output: {hex(target_value)}")

        current_value = target_value
        for op in reversed(segment):  # undo ops in reverse order
            if op == 'off':
                current_value = rev_off(current_value)
            elif op == 'eor':
                current_value = rev_eor(current_value)
            elif op == 'rtr':
                current_value = rev_rtr(current_value)

        required_inputs.append(current_value)
        char_repr = chr(current_value) if 32 <= current_value <= 126 else '?'
        print(f"Required input: {hex(current_value)} = {current_value} ('{char_repr}')")

    # Step 4: Construct potential flag string
    flag_chars = [
        chr(val) if 32 <= val <= 126 else f"\\x{val:02x}"
        for val in required_inputs
    ]
    potential_flag = ''.join(flag_chars)

    print(f"\nRecovered flag bytes: {[hex(x) for x in required_inputs]}")
    print(f"Potential flag: {potential_flag}")

    # Step 5: Verify by simulating forward
    print("\nVerification:")
    test_flag = required_inputs + [0] * (14 - len(required_inputs))
    result = simulate_program(test_flag)
    print(f"Our result: {[hex(x) for x in result[:len(target)]]}")
    print(f"Target:     {[hex(x) for x in target]}")
    matches = sum(1 for a, b in zip(result[:len(target)], target) if a == b)
    print(f"Matches: {matches}/{len(target)}")

    return required_inputs


def simulate_program(initial_flag):
    """
    Forward simulate the binary’s operations with given input flag.
    Used for verification step (make sure reversed inputs reproduce target).
    """
    flag_array = initial_flag[:]
    globalvar = 0
    current_byte = flag_array[0]
    stored_values = []

    for op in operations:
        if op == 'off':
            current_byte = off(current_byte)
        elif op == 'eor':
            current_byte = eor(current_byte)
        elif op == 'rtr':
            current_byte = rtr(current_byte)
        elif op == 'inc':
            # Store result, advance to next input
            flag_array[globalvar] = current_byte
            stored_values.append(current_byte)
            globalvar += 1
            current_byte = flag_array[globalvar] if globalvar < len(flag_array) else 0

    return stored_values


if __name__ == "__main__":
    print("=== Solving 'stacked' ===")
    solution = solve_system()
```
This script:
- Takes the known output bytes from the challenge binary.
- Splits the operations into per-character segments.
- Works backwards through each segment using reverse operations.
- Recovers the original input bytes (the flag).
- Verifies by re-simulating forwards.

**Breakdown:**  
1. Analyze the binary by using various commands like gdb, objdump, ropper etc.,
2. Crafted a python script to decrypt the given hexadecimal bytes.
3. Run the script, it will give us a valid flag

**Outcome:** 
Successfully decrypted the flag.

**Flag:**
`ictf{1n54n3_5k1ll2}`