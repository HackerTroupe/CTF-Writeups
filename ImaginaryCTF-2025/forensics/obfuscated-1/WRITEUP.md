## Obfuscated-1

**Name:** obfuscated-1

**Category:** Forensics

**Author:** Eth007

**Solved By** `s0c41dgvns`

**Description:**

> I installed every old software known to man... The flag is the VNC password, wrapped in ictf{}.
> 

**Attachment:** [Users.zip](https://github.com/ImaginaryCTF/ImaginaryCTF-2025-Challenges/blob/main/Forensics/obfuscated-1/dist/Users.zip)

---

### 💡 Thought Process

The mention of a *VNC password* instantly made me think of the Windows Registry, because TightVNC stores its server settings (including passwords) under the `NTUSER.DAT` hive.
My intuition was:

1. Load the registry hive.
2. Search for TightVNC keys.
3. Locate and decrypt the stored password.

This reasoning comes from past forensic challenges — legacy remote desktop tools often use weak, static encryption (like DES), making them easy targets once you know where to look.

---

### 🛠️ Steps to Solve

**Step 1: Load the Registry**

- Extracted `Users.zip`.
- Located the `NTUSER.DAT` file.
- Opened it using **Registry Explorer**.

**Step 2: Find TightVNC Settings**

- Path: `Root > Software > TightVNC > Server`
- Found a value named **Password**.

**Step 3: Decrypt the Password**

- Password was stored in DES-encrypted bytes.
- Used a custom Python script to decrypt:

```python
from Crypto.Cipher import DES

# TightVNC fixed DES key
KEY = bytes([0x17, 0x52, 0x6b, 0x20, 0x4b, 0x72, 0x41, 0x4b])

def decrypt_vnc(enc_hex: str) -> str:
    enc = bytes.fromhex(enc_hex)
    cipher = DES.new(KEY, DES.MODE_ECB)
    dec = cipher.decrypt(enc)
    return dec.decode(errors="ignore").rstrip("\\x00")

if __name__ == "__main__":
    encrypted = "CA8D9F..."  # replace with actual registry value
    print("Decrypted password:", decrypt_vnc(encrypted))
```

- Running it produced the password: `Slay4U!!`
- Wrapped in `ictf{}` → `ictf{Slay4U!!}`

---

### 🪞 Reflection

This was a classic registry-artifact challenge: knowing where software hides credentials can be more valuable than brute-force guessing. Forensics often rewards prior knowledge of *where data likes to live*.
