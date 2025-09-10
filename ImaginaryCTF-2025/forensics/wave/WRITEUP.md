## Wave

**Name:** wave
**Category:** Forensics
**Author:** Eth007
**Solved By** `s0c41dgvns`

**Description:**

> not a steg challenge i promise
> 

**Attachment:** [wave.wav](https://github.com/ImaginaryCTF/ImaginaryCTF-2025-Challenges/blob/main/Forensics/wave/dist/wave.wav)

**Flag:** `ictf{obligatory_metadata_challenge}`

---

### 💡 Thought Process

When handed an audio file, my first suspicion was **steganography**. Common hiding techniques include:

- **LSBs of audio samples**
- **Spectrogram tricks** (drawings in frequency space)
- **Metadata tags** (artist/comment fields)
- **Appended raw data**

So I followed the typical forensic workflow:

1. **Check file type:**
    
    ```bash
    file wave.wav
    
    ```
    
    → Valid RIFF/WAV file.
    
2. **Strings search:**
    
    ```bash
    strings wave.wav | less
    
    ```
    
    → No obvious hits.
    
3. **Spectrogram analysis (Audacity / Sonic Visualizer):**
    - No unusual shapes or hidden messages.
4. **Check metadata:**
    - No suspicious tags.

At this point, I suspected hidden raw data.

---

### 🛠️ Solution

Dumped the hex:

```bash
xxd wave.wav | less

```

Scrolling revealed the flag plainly embedded in the data section:

```
... ictf{obligatory_metadata_challenge} ...

```

So the challenge wasn’t deep stego — just a subtle trick hidden in plain hex.

---

### ✅ Flag

```
ictf{obligatory_metadata_challenge}

```
