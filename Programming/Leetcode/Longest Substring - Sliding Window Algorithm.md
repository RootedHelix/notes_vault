Related to [[Leetcode]]

#leetcode


- We take an array of characters and use a certain iterator to identify index at start flag.
- then we set the iterator i of the for the loop to be what we use to expand the window through the array
Here’s a **clean, mental-model summary** of what that sliding-window algorithm is doing — no fluff, just the mechanics.

---

## What problem this solves

> **Find the length of the longest substring with no repeated characters**  
> (e.g. `"abcabcbb"` → `3` for `"abc"`)

---

## The core idea (one sentence)

> Maintain a **moving window** `[start … i]` that always contains **unique characters**, and expand it greedily while fixing duplicates when they appear.

---

## What each piece represents

### `last[POSSIBLE_CHARS]`

- An array indexed by character value (0–255)
    
- Stores the **most recent index** where each character appeared
    
- Initialized to `-1` = “never seen”
    

```c
last['a'] = 5;   // 'a' last appeared at index 5
```

---

### `start`

- The **left boundary** of the current window
    
- Always points to the first character in the current “valid” substring
    

---

### `i`

- The **right boundary** of the window
    
- Moves forward one character at a time
    

---

## What happens on each character

### Step 1: Read the current character

```c
unsigned char c = (unsigned char)s[i];
```

- Cast ensures valid indexing (0–255)
    

---

### Step 2: Check for a duplicate _inside the window_

```c
if (last[c] >= start)
{
    start = last[c] + 1;
}
```

This means:

- We’ve seen this character before
    
- And it occurred **inside the current window**
    
- So the window is no longer valid
    

👉 **Fix:** Move `start` _just past_ the previous occurrence

This is the “slide” in _sliding window_.

---

### Step 3: Record this character’s position

```c
last[c] = i;
```

So future duplicates know where it last appeared.

---

### Step 4: Compute current window length

```c
curr = i - start + 1;
```

Window is always valid here (no duplicates).

---

### Step 5: Update the maximum

```c
max_len = max(max_len, curr);
```

---

## Why this is efficient (the key insight)

- `i` only moves **forward**
    
- `start` only moves **forward**
    
- Each character is processed once
    

👉 **O(n) time**, **O(1) space**

No nested loops. No backtracking.

---

## Example walk-through (quick)

For `"abba"`:

|i|char|action|
|---|---|---|
|0|a|window = `"a"`|
|1|b|window = `"ab"`|
|2|b|duplicate → move `start` to `2`|
|3|a|valid → window = `"ba"`|

Max length = `2`

---

## One-line takeaway

> This algorithm keeps a **maximal, duplicate-free window** by jumping the left pointer forward when duplicates appear, ensuring linear time.

If you want, next I can:

- Draw this visually
    
- Compare it to the naive O(n²) approach
    
- Show how this pattern generalizes to other sliding window problems
