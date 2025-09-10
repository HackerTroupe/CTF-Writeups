**Name:** passwordless

**Category:** Web

**Author:** Ciaran

**Solved By** `s0c41dgvns`

**Description:**

> Didn't have time to implement the email sending feature but that's ok, the site is 100% secure if nobody knows their password to sign in!
> 

**Challenge URL:** [http://passwordless.chal.imaginaryctf.org](http://passwordless.chal.imaginaryctf.org/)

**Attachment:** [passwordless.zip](https://github.com/ImaginaryCTF/ImaginaryCTF-2025-Challenges/blob/main/Web/passwordless/dist/passwordless.zip)

**Flag:** `ictf{8ee2ebc4085927c0dc85f07303354a05}`

**Solved By** `s0c41dgvns`
---

### 💡 Intuition

The description screamed **auth bypass**. If users never get their password, how can they log in? That implied a logic flaw in password handling.

Looking at the source:

- Registration creates `password = email + randomHex`.
- This is hashed with **bcrypt**.
- Problem: bcrypt truncates inputs longer than **72 bytes**.
- Additionally, **normalize-email** strips dots, `+tags`, and lowercases Gmail addresses.

That gave me the idea: if the email is long enough, the random hex part would be ignored due to truncation, leaving a deterministic password based only on the email prefix.

---

### 🛠️ Steps to Solve

**Step 1: Read the code**

- Normalize email → shorter version is stored.
- Password = `email + randomHex`.
- If `len(password) > 72`, bcrypt truncates at 72.

**Step 2: Craft email**

Register with something like:

```
test+aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa@gmail.com

```

- Normalizes to `test@gmail.com`.
- Password becomes long enough that random hex is truncated.

**Step 3: Login**

- Email: `test@gmail.com`
- Password: `test+aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa@gmail.com` (truncated prefix part).

This bypass worked and logged me in as the user, revealing the flag.

---

### ✅ Flag

```
ictf{8ee2ebc4085927c0dc85f07303354a05}

```
