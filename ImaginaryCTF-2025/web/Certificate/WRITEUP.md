**Name:** certificate  
**Category:** Web  
**Author:** Eth007  
**Solved by:** Y0RU1CH1  

**Description:**

> As a thank you for playing our CTF, we're giving out participation certificates! Each one comes with a custom flag, but I bet you can't get the flag belonging to Eth007!

**Challenge URL:** [http://codenames-1.chal.imaginaryctf.org/](http://codenames-1.chal.imaginaryctf.org/)

---

### 💡 Intuition

The challenge hinted at custom flags in certificates. The site asked for a hacker name and showed a “certificate preview.” 

That felt like client-side logic obfuscation — the frontend likely had the code to generate or fetch the flag, just not displaying it properly. So the trick was to bypass the UI and see how the flag was really derived.

### 🛠️ Steps to Solve

---

**Step 1: Inspect network requests**

Opened DevTools → saw a request to /cert/.

No useful parameters, just the certificate HTML served.

**Step 2: Check the page scripts**

Looked at the JavaScript used for certificate generation.

Found a function that took the hacker name, applied a hashing function, and appended it inside ictf{...}.

This meant the flag wasn’t stored anywhere — it was computed dynamically.

**Step 3: Reproduce the function locally**

Copied the hash function into a local JS/Python script.

Ran it with the target username Eth007.

---

### ✅ Flag

```
ictf{7b4b3965}

```
