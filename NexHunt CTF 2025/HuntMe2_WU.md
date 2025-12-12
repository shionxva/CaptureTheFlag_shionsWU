# HuntMe2 Write-Up

## 1. File overview

```
┌─────────────────┐
│   User Input    │ (32 chars)
└────────┬────────┘
         │
┌────────▼────────┐
│  FUN_004013df   │
│  - Get input    │
│  - Sanitize     │
│  - Call checker │
└────────┬────────┘
         │
┌────────▼────────┐
│  FUN_0040132a   │◄───────┐
│  - Check len=32 │        │
│  - For i=0..31: │        │
│    • mask[i]    │        │
│    • input[i] ^ │        │
│      mask[i]    │        │
│    • Compare to │────────┘
│      target[i]  │
└────────┬────────┘
         │
     ┌───▼────┐
     │ Match? │
     └───┬────┘
   No    │    Yes
┌────────▼────────┐      ┌──────────────────────────────┐
│ "Forest remains │      │"You adapt. The hunt continues."│
│   closed."      │      └──────────────────────────────┘
└─────────────────┘
```
The program expects **exactly 32 characters** of input.  
The validation process consists of:

1. Reading and cleaning the user input  
2. Passing it into the main validator `FUN_0040132a`  
3. For each byte, computing a position-dependent mask  
4. Checking:

```
input[i] XOR mask[i] == target[i]
```

If all 32 bytes match, the program prints:

> “You adapt. The hunt continues.”

Otherwise:

> “Forest remains closed.”

![HuntMe2_1](https://github.com/shionxva/CaptureTheFlag_shionsWU/blob/main/NexHunt%20CTF%202025/Huntme2(1).png?raw=true)

---

## 2. Validation Logic (FUN_0040132a)

The main validation performs:

1. Check that the input length is exactly **32**
2. For each index `i` from `0` to `31`:
   - Compute a mask byte with:

```
mask[i] = FUN_00401239(i)
```

   - Compare:

```
input[i] ^ mask[i] == target[i]
```

From this, the correct flag byte is:

```
flag[i] = target[i] ^ mask[i]
```

---

## 4. Mask Generation (FUN_00401239)

### 4.1 Input Data

The function uses **five 7-byte strings**, referenced as:

```
local_48[0] = &DAT_00402020   // 7 bytes
local_48[1] = &DAT_00402027   // 7 bytes
local_48[2] = &DAT_0040202e   // 7 bytes
local_48[3] = &DAT_00402035   // 7 bytes
local_48[4] = &DAT_0040203c   // 7 bytes
```

### 4.2 Indexing Logic

For each string index `j = 0..4`, the byte selection index is:

```
idx = (j*j) + (j+1)*i + 3
idx = idx % 7
```

Meaning:
- `j*j` gives: 0, 1, 4, 9, 16  
- `(j+1)*i` scales according to position `i`  
- `+3` is a constant offset  
- `% 7` wraps into the valid range of each 7-byte string  

![HuntMe2_2](https://github.com/shionxva/CaptureTheFlag_shionsWU/blob/main/NexHunt%20CTF%202025/Huntme2(2).png?raw=true)

### 4.3 Per-Iteration Mix

For each chosen byte, accumulator `local_9` is updated with:

```
local_9 = local_9 ^ local_18[idx];
local_9 = ((local_9 >> 7) | (local_9 * 2)) & 0xFF;
```

This is effectively a **1-bit rotate-left**, because:

- `(x >> 7)` extracts the MSB  
- `(x << 1)` shifts left  
- OR combines them  

This process repeats 5 times (one per string).

---

## 5. Final Mask Transformation

After the rotation logic, the function performs another transformation:

```
bVar1 = local_9 ^ (local_9 << 3);
return bVar1 ^ (bVar1 >> 5) ^ (i * '=');
```

Where `'='` is ASCII `0x3D` (decimal **61**).

Thus the final mask byte is:

```
mask[i] =
    ( local_9 ^ (local_9 << 3) )
    ^ ( (local_9 ^ (local_9 << 3)) >> 5 )
    ^ ( i * 61 )
```

This makes the mask nonlinear and dependent on position `i`.

![HuntMe2_3](https://github.com/shionxva/CaptureTheFlag_shionsWU/blob/main/NexHunt%20CTF%202025/Huntme2(3).png?raw=true)

---

## 0. Conclusion

Combining everything:

```
for i = 0..31:
    if (input[i] ^ mask[i]) != target[i]:
        reject()
```

Thus the true 32-byte flag is reconstructed by:

```
flag[i] = target[i] ^ mask[i]
```

---
