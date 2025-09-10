**Type:** Reversing  
**Level:** 4/10  
**Author:** cleverbear57  
**Solved by:** LUZ1LF3R

**Description:**  
I made this weird Android app, but all it gave me was this .apk file. Can you get the flag from it?

**Files:**

- `weird.zip` → `app-debug.apk`

**Plan:**  
Analyze the APK file to recover the original flag. Use tools like `apktool` and `jadx` to decompile, then inspect the code for the flag transformation function. Finally, reverse the transformation logic with a custom script.

**Notes:**

- Installed the APK on Android; it displayed:
  ```txt
  Transformed flag: idvi+1{s6e3{)arg2zv[moqa905+
  ```
- This suggests the real flag is hidden and transformed inside the app.
- Located the string in MainActivityKt.smali using apktool + grep.
- Found a function transformFlag, which manipulates characters differently depending on type (letters, numbers, special chars).
- Wrote a Python script to reverse the transformation logic.

```python
def reverse_transform(transformed: str) -> str:
    # Alphabets, numbers, and special characters used in the transform
    alpha = "abcdefghijklmnopqrstuvwxyz"
    nums = "0123456789"
    spec = "!@#$%^&*()_+{}[]|"

    original = ""

    # Loop over each character with its index
    for i, ch in enumerate(transformed):
        if ch in alpha:
            # Rule (observed in smali):
            # transformed = alpha[(orig_index + i) % 26]
            t = alpha.index(ch)
            orig_index = (t - i) % len(alpha)
            original += alpha[orig_index]

        elif ch in nums:
            # Rule:
            # transformed = nums[(orig_index + 2*i) % 10]
            t = nums.index(ch)
            orig_index = (t - 2*i) % len(nums)
            original += nums[orig_index]

        elif ch in spec:
            # Rule:
            # transformed = spec[(orig_index + i*i) % len(spec)]
            t = spec.index(ch)
            orig_index = (t - i*i) % len(spec)
            original += spec[orig_index]

        else:
            # Fallback (probably never needed in a flag)
            original += ch

    return original


# === Test Run ===
transformed_flag = "idvi+1{s6e3{)arg2zv[moqa905+"
flag = reverse_transform(transformed_flag)
print("Flag:", flag)  # note: expect the flag

```

**Breakdown:**

1. Extract the APK from the provided ZIP.
2. Decompile using apktool.
3. Install and run the app → observe the transformed flag.
4. Search for the transformed string inside the decompiled code.
5. Identify the transformFlag function inside MainActivityKt.smali.
6. Analyze the transformation rules (letters shifted by index, numbers by 2\*i, special chars by i^2).
7. Implement a Python script to reverse these rules.
8. Run the script on the transformed string to recover the original flag.

**Outcome:**  
Successfully reversed the transformation logic and retrieved the original flag.

**Flag:**  
`ictf{1_l0v3_@ndr0id_stud103}`
