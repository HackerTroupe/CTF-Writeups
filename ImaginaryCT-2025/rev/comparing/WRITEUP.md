**Type:** Reversing  
**Level:** 4/10  
**Author:** cleverbear57  
**Solved by:** LUZ1LF3R

**Description:**  
I put my flag into this program, but now I lost the flag. Here is the program, and the output. Could you use it to find the flag?

**Files:**

- `comparing.cpp`
- `output.txt`

**Plan:**  
The provided C++ program only _encodes_ the flag. Since the flag itself is missing, we need to write a reverse script that takes the scrambled numeric output and reconstructs the original input (the flag).

**Notes:**

- The program splits the flag into 2-character chunks.
- These chunks are stored in a priority queue sorted by ASCII sum.
- In each step, two pairs are popped and recombined:
  - If step index is even → mirrored encoding.
  - If step index is odd → direct concatenation.
- The outputs are scrambled numeric strings (written to `output.txt`).
- To solve: reverse this process to retrieve the original 2-char pairs, then reassemble the flag.

```python
#!/usr/bin/env python3

def decode_output_systematically():
    """
    Goal: Reverse the encoded outputs into the original flag
    Approach:
      - Parse outputs (even vs odd encoding)
      - Group them by index
      - Reconstruct original character pairs
      - Assemble the flag
    """

    target_output = [
        "9548128459", "491095", "1014813", "561097",
        "10211614611201", "5748108475", "1171123", "516484615",
        "114959", "649969946", "1051160611501", "991021",
        "1231012101321", "9912515", "11411511", "1151164611511"
    ]

    # --- Decoding helpers ---
    def parse_even_output(line):
        """Even: val1+val3+index+reverse(val1+val3)"""
        for idx_len in [1, 2]:
            for pos in range(1, len(line) - idx_len):
                front, idx_part, back = line[:pos], line[pos:pos+idx_len], line[pos+idx_len:]
                if front == back[::-1] and len(front) > 0:
                    try:
                        index = int(idx_part)
                        for split in range(1, len(front)):
                            val1, val3 = int(front[:split]), int(front[split:])
                            if 32 <= val1 <= 126 and 32 <= val3 <= 126:
                                return index, chr(val1), chr(val3)
                    except:
                        continue
        return None, None, None

    def parse_odd_output(line):
        """Odd: val1+val3+index"""
        try:
            value_str = str(int(line))
            for idx_len in [1, 2]:
                if len(value_str) > idx_len:
                    idx_part, remaining = value_str[-idx_len:], value_str[:-idx_len]
                    try:
                        index = int(idx_part)
                        for split in range(1, len(remaining)):
                            val1, val3 = int(remaining[:split]), int(remaining[split:])
                            if 32 <= val1 <= 126 and 32 <= val3 <= 126:
                                return index, chr(val1), chr(val3)
                    except:
                        continue
        except:
            pass
        return None, None, None

    # --- Step 1: Parse lines ---
    parsed_data = []
    for line_idx, line in enumerate(target_output):
        index, c1, c2 = parse_even_output(line)
        if index is None:
            index, c1, c2 = parse_odd_output(line)
        if index is not None:
            parsed_data.append({'line': line_idx, 'index': index, 'char1': c1, 'char2': c2})

    # --- Step 2: Group by index ---
    by_index = {}
    for item in parsed_data:
        by_index.setdefault(item['index'], []).append(item)

    # --- Step 3: Reconstruct pairs from recombination pattern ---
    original_pairs = {}
    for i in range(0, 16, 2):
        items = [it for it in parsed_data if it['line'] in (i, i+1)]
        if len(items) == 2:
            a, b = items
            idx1, idx2 = a['index'], b['index']
            original_pairs.setdefault(idx1, []).extend([a['char1'], b['char1']])
            original_pairs.setdefault(idx2, []).extend([a['char2'], b['char2']])

    # --- Step 4: Build the flag ---
    flag_chars = ['?'] * 32
    for idx, chars in original_pairs.items():
        if len(chars) >= 2:
            flag_chars[idx*2] = chars[0]
            flag_chars[idx*2+1] = chars[1]

    reconstructed_flag = ''.join(flag_chars)
    print(f"🏁 Reconstructed flag: {reconstructed_flag}")
    return reconstructed_flag


# Run
if __name__ == "__main__":
    decode_output_systematically()
```

**Breakdown:**

1. Analyzed `comparing.cpp` to understand the custom comparator and encoding functions.
2. Identified the recombination logic (val1/val3 and val2/val4 mixing).
3. Implemented a Python script to reverse the encoding rules.
4. Ran the script on `output.txt` to reconstruct the original flag.

**Outcome:**  
The reverse script successfully retrieved the flag from the encoded output.

**Flag:**  
`ictf{cu3st0m_c0mp@r@t0rs_1e8f9e}`
