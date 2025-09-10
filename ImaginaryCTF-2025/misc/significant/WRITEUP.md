**Name:** significant
**Category:** Misc
**Author:** puzzler7
**Solved by:** Y0RU1CH1

**Description:**

The signpost knows where it is at all times. It knows this because it knows where it isn't, by subtracting where it is, from where it isn't, or where it isn't, from where it is, whichever is greater. Consequently, the position where it is, is now the position that it wasn't, and it follows that the position where it was, is now the position that it isn't.

Please find the coordinates (lat, long) of this signpost to the nearest 3 decimals, separated by a comma with no space. Ensure that you are rounding and not truncating before you make a ticket. Example flag: ictf{-12.345,6.789}

**Attachments:** [significant.jpg](https://github.com/ImaginaryCTF/ImaginaryCTF-2025-Challenges/blob/main/Misc/significant/dist/significant.jpg)

---

### 💡 Intuition

The challenge only provided an image, which hinted at an OSINT (Open-Source Intelligence) problem.
Since the picture looked like a public place, the most straightforward way was to use a reverse image search to track down the location.

### 🛠️ Steps to Solve

---

**Step 1: Reverse Image Search**

I uploaded the given image to Google Lens.

It matched with several Instagram reels and YouTube shorts.

**Step 2: Identify the Location**

From those sources, I confirmed that the place was Hallidie Plaza, San Francisco.

**Step 3: Verify on Google Maps**

I searched for Hallidie Plaza on Google Maps.

Found the exact location and copied its coordinates.

**Step 4: Submit Flag**

The challenge required the coordinates as the flag.

I used the copied coordinates as the final submission.
---

### ✅ Flag

```
ictf{37.785,-122.405}


```
