**Name:** codenames-1  
**Category:** Web  
**Author:** Eth007  
**Solved by:** Y0RU1CH1

**Description:**

> I hear that multilingual codenames is all the rage these days. Flag is in /flag.txt.
> 

**Challenge URL:** [https://eth007.me/cert/](https://eth007.me/cert/)

**Attachment:** [codenames.zip](https://github.com/ImaginaryCTF/ImaginaryCTF-2025-Challenges/tree/main/Web/codenames-1/challenge)

**Flag:** `ictf{common_os_path_join_L_b19d35ca}`

---

### 💡 Intuition

The challenge statement mentioned that the flag was in `/flag.txt`.  
Looking at the source code, I noticed that when a game is created, the server loads the word list from a file specified by the `language` parameter.  

That means if I supply `/flag` as the value, the server will try to read `/flag.txt` and use its contents as the dictionary.  
This is a classic **arbitrary file read** vulnerability.

---

### 🛠️ Steps to Solve

**Step 1: Analyze the code**

In `app.py`:

```python
if request.form.get("language"):
    language = request.form.get("language")
    with open(language, "r") as f:
        words = f.read().splitlines()
```

The code directly opens whatever file path is passed via `language`.


**Step 2: Exploit file read**

- Go to the game lobby.  
- Set the `language` field to `/flag`.  
- Start the game.


**Step 3: Observe the board**

The server read `/flag.txt` and used it as the word list.  
The entire board displayed the flag.

---

**Result:**

```
ictf{common_os_path_join_L_b19d35ca}

```




