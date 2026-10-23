# Hashing in DSA: Complete Revision Notes

> Source: Striver's A to Z DSA course, "Basic Stuff" module (last lecture).
> The lecture uses pseudocode and C++ style code, but the ideas are the same in Java and Python.

---

## 0. Why hashing matters

- A huge number of DSA problems (frequency counting, duplicates, pairs, subarrays, strings) become easy or fast only if you know hashing.
- Core idea in one line: **Pre-store something, then fetch it when needed.**

---

## 1. The Initial Problem

Given an array, answer many queries of the form: **"How many times does number X appear in the array?"**

Example array: `[1, 3, 2, 1, 2]`

| Query | Answer |
|-------|--------|
| 1 | 2 |
| 3 | 1 |
| 4 | 0 (not present) |
| 2 | 2 |
| 10 | 0 |

Important point: you are asked **many times** (Q queries), for any number, and numbers can repeat across queries.

---

## 2. Brute Force Approach

For each query, loop through the whole array and count matches.

### Pseudocode

```
function countOccurrences(number, array):
    counter = 0
    for i = 0 to n-1:
        if array[i] == number:
            counter = counter + 1
    return counter
```

### How it works (dry run for number = 1)

- Start `counter = 0`.
- Compare with each element; every time `array[i] == 1`, increase counter.
- Return counter at the end.

### Time Complexity

- One query = O(n) (one full loop).
- Q queries = **O(Q * n)**.

### Why this is bad (big numbers)

- Suppose Q = 10^5 and n = 10^5.
- Operations = 10^5 * 10^5 = **10^10**.
- Rule of thumb: **10^8 operations is about 1 second**.
- So 10^10 operations = 10^10 / 10^8 = 10^2 = **about 100 seconds** (over 1.5 minutes). Way too slow. Most DSA problems allow 1 to 2 seconds.

That is why we need hashing.

---

## 3. What is Hashing?

**Hashing = pre-storing and fetching.**

You do **one pre-computation pass** over the data, store the results, and then every query is answered by just looking up the stored result (no loop).

---

## 4. Number Hashing using Arrays

### Idea

Suppose the problem says the array elements are at most 12.

1. Create another array called the **hash array** (also called **frequency array**) of size **13** (indexes 0 to 12). Fill with 0.
2. The **index** of the hash array represents the **number**.
3. The **value** at that index represents **how many times** that number appears.

### Pre-computation

Go through the original array. For each element `x`, do `hash[x] += 1`.

For `[1, 3, 2, 1, 2]`:

| Index | 0 | 1 | 2 | 3 | 4 | ... | 12 |
|-------|---|---|---|---|---|-----|----|
| Value | 0 | 2 | 2 | 1 | 0 | ... | 0 |

### Fetching

To answer "how many times does X appear?", just read `hash[X]`. That is **O(1)** per query.

- `hash[1] = 2`, `hash[3] = 1`, `hash[4] = 0`, `hash[2] = 2`, `hash[10] = 0`, `hash[12] = 0`.

### Full C++ code

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n;
    cin >> n;

    int arr[n];
    for (int i = 0; i < n; i++) {
        cin >> arr[i];
    }

    // PRE-COMPUTE
    int hash[13] = {0};          // size 13 because max element is 12
    for (int i = 0; i < n; i++) {
        hash[arr[i]] += 1;       // go to index arr[i] and add 1
    }

    int q;
    cin >> q;
    while (q--) {
        int number;
        cin >> number;

        // FETCH
        cout << hash[number] << endl;
    }
    return 0;
}
```

### Input format used in the lecture

1. `n` (size of array)
2. `n` array elements
3. `q` (number of queries)
4. `q` numbers, one per query

Sample: `n = 5`, array `1 3 2 1 3`, `q = 5`, queries `1 4 2 3 12` gives outputs `2 0 1 2 0`.

### Sizing rule

- If the maximum element is `M`, the hash array must have size **M + 1** (so index M exists).
- If the problem says max is 10^5, declare size 10^5 + 1. If 10^6, declare 10^6 + 1.

### Complexity

| Part | Time |
|------|------|
| Pre-computation | O(n) |
| Each query | O(1) |
| Total for Q queries | O(n + Q) |

Compare with brute force O(Q * n). Huge improvement.

---

## 5. Limits of Array Size (Very Important)

What if elements go up to 10^9, 10^12, 10^15? Can we make an array of size 10^9 + 1? **No.**

| Where declared | Type | Max practical size |
|----------------|------|--------------------|
| **Inside `main`** | `int` array | about **10^6** |
| **Globally** (outside main) | `int` array | about **10^7** |
| Inside `main` | `bool` array | about 10^7 |
| Globally | `bool` array | about 10^8 |

- Going beyond gives **segmentation fault** (memory cannot be allocated).
- Demonstrated in the lecture: size 10^7 inside main failed; one zero less worked; 10^7 globally worked.
- **Bonus fact:** globally declared arrays are **automatically initialized to 0**, so you do not need `= {0}`. Local arrays contain garbage values unless you initialize them.
- Name clash tip: naming your array `hash` globally can give "ambiguous" error in C++ because `hash` already exists in the standard library. Rename it (e.g. `hsh`).

**Conclusion:** array hashing for numbers works only up to about 10^6 (in main) or 10^7 (global). For bigger values we use **map / unordered_map** (Section 8 and 9).

---

## 6. Character Hashing

### Problem

Given a string like `abcdabefc`, answer queries: how many times does a character appear?

- `a` = 2, `c` = 2, `z` = 0.

### Brute force

For each query loop through the string and count. Time = **O(Q * n)** again.

```
function countChar(c, s):
    counter = 0
    for i = 0 to n-1:
        if s[i] == c: counter++
    return counter
```

### Hashing idea

Map each character to an array index.

**Case 1: only lowercase letters (a to z)**

- There are 26 letters, so use array of size **26**.
- Map: `a -> 0, b -> 1, c -> 2, ... z -> 25`.
- Formula: **index = character - 'a'**

### Why does `character - 'a'` work? ASCII!

- ASCII = American Standard Code for Information Interchange. Each character has an integer code (256 characters in total).
- Key values to remember:

| Character | ASCII |
|-----------|-------|
| digit `'0'` | 48 |
| digit `'1'` | 49 |
| uppercase `'A'` | 65 |
| lowercase `'a'` | 97 |
| lowercase `'z'` | 122 |

- When you subtract characters, the computer subtracts their ASCII integers.
- Example: `'f' - 'a' = 102 - 97 = 5`, `'b' - 'a' = 1`, `'c' - 'a' = 2`.
- `int x = 'a';` stores 97 in `x` (auto typecast, no error).

### Code (lowercase only)

```cpp
string s;
cin >> s;

// PRE-COMPUTE
int hash[26] = {0};
for (int i = 0; i < s.size(); i++) {
    hash[s[i] - 'a']++;
}

int q;
cin >> q;
while (q--) {
    char c;
    cin >> c;

    // FETCH
    cout << hash[c - 'a'] << endl;
}
```

**Case 2: uppercase, digits, symbols may also be present (or not stated)**

- Use array of size **256** and **do not subtract anything**.
- `hash[s[i]]++` works because a char used as an index is **auto converted to its ASCII integer**.

```cpp
int hash[256] = {0};
for (int i = 0; i < s.size(); i++) {
    hash[s[i]]++;
}
// fetch:
cout << hash[c] << endl;
```

- This also works for lowercase-only input, it just uses more memory than needed.

### Rules to remember

| Situation | Array size | Index formula |
|-----------|-----------|---------------|
| Only lowercase | 26 | `c - 'a'` |
| Only uppercase | 26 | `c - 'A'` |
| Anything / not stated | 256 | `c` (direct) |

### Is there any size problem in character hashing?

**No.** There are at most 256 characters, so arrays always work. **For characters, always prefer arrays.**

---

## 7. Why we need something more than arrays

For numbers up to 10^9, 10^12, 10^18 an array cannot be created. This is where **C++ STL** (`map`, `unordered_map`) comes in. In Java, use **`HashMap`** / Collections. The concepts are the same, only the syntax differs.

---

## 8. `map` in C++ STL

### Concept

A map stores **key -> value** pairs.

- **Key** = the number (what you are hashing).
- **Value** = its frequency (how many times it appears).

### Behavior

- `mpp[x]` for a key that does not exist gives **0** (and creates it).
- `mpp[x]++` therefore turns a missing key into 1, and an existing key increases by 1.

Example with `[1, 2, 3, 1, 3, 2]`:

| Step | Element | Map after step |
|------|---------|----------------|
| 1 | 1 | {1:1} |
| 2 | 2 | {1:1, 2:1} |
| 3 | 3 | {1:1, 2:1, 3:1} |
| 4 | 1 | {1:2, 2:1, 3:1} |
| 5 | 3 | {1:2, 2:1, 3:2} |
| 6 | 2 | {1:2, 2:2, 3:2} |

### Code

```cpp
int main() {
    int n;
    cin >> n;
    int arr[n];
    for (int i = 0; i < n; i++) cin >> arr[i];

    // PRE-COMPUTE
    map<int, int> mpp;                 // key type, value type
    for (int i = 0; i < n; i++) {
        mpp[arr[i]]++;
    }

    int q;
    cin >> q;
    while (q--) {
        int number;
        cin >> number;

        // FETCH
        cout << mpp[number] << endl;
    }
    return 0;
}
```

- If numbers are as large as 10^9 to 10^12 or more, use `long long` as key type: `map<long long, int>`.
- You can do the pre-computation while taking input (saves one loop). It does not change complexity: O(n + n) = O(2n) = O(n), constants are ignored.

### Advantage over array hashing (memory)

- Map stores **only the keys that actually appear**.
- Example: if max element is 12 and elements are `1, 2, 3, 12`, array hashing needs 13 slots, map stores only 4 entries.
- Looking up a missing key (e.g. 4) just returns 0 and does not store anything useful.

### Map stores keys in SORTED order

- Iterating over a `map` gives keys in **ascending order**.

```cpp
for (auto it : mpp) {
    cout << it.first << " -> " << it.second << endl;
}
```

- `it.first` = key, `it.second` = value.

### Map with characters

Key can be a character, no need to convert to integer:

```cpp
map<char, int> mpp;
for (int i = 0; i < s.size(); i++) {
    mpp[s[i]]++;
}
```

### Time complexity of `map`

- Both **storing and fetching** take **O(log n)** in **all cases** (best, average, worst).
- n here means number of elements in the map.

---

## 9. `unordered_map`

- Same syntax as `map`; just write `unordered_map<int, int> mpp;`
- Keys are stored in **no particular order**. The order can even differ between compilers or runs. Do not depend on it.

### Time complexity

| Case | Store / Fetch |
|------|---------------|
| Best | **O(1)** |
| Average | **O(1)** |
| Worst | **O(n)** (n = number of elements in the map) |

- The worst case is rare ("once in a blue moon"). It happens due to **collisions** (Section 10).
- If the loop runs n times and each operation is O(n), total becomes O(n^2). With O(1) it stays O(n).

### Which one should I use?

1. **First preference: `unordered_map`** (because worst case almost never happens).
2. If you get **Time Limit Exceeded (TLE)**, switch to `map` (guaranteed O(log n)).
3. Interview answer: "Average and best case O(1), worst case O(n) for unordered_map."

### Key type differences

| | `map` | `unordered_map` |
|--|-------|-----------------|
| Order | Sorted | No order |
| Time | O(log n) always | O(1) avg/best, O(n) worst |
| Allowed key types | Almost anything: int, double, char, string, `pair<int,int>`, vector, etc. | Only simple types: int, double, char, string. **No** `pair<int,int>`, **no** vector as key |

---

## 10. How hashing works internally (Division Method, Collisions)

Interviewers rarely ask you to implement this. Understand the **concept** and vocabulary only.

### Problem it solves

Numbers can be huge (e.g. 10^18), but memory is limited. We need to **squeeze big numbers into a small array**.

### Division method (simple example)

Suppose we may only use an array of size **10** (indexes 0 to 9). Take numbers `2, 5, 16, 28, 139`.

- Hash function: **index = number % 10**

| Number | Index |
|--------|-------|
| 2 | 2 |
| 5 | 5 |
| 16 | 6 |
| 28 | 8 |
| 139 | 9 |

To query "how many times does 139 appear?": compute `139 % 10 = 9`, go to index 9, read the value.

### What if two numbers land on the same index?

Example: 28 and 18 and 38 all give `% 10 = 8`.

- Solution: **chaining**. At each index, keep a **linked list** (chain) of all numbers that landed there. (Kept in sorted order so searching, even binary search, is fast.)
- To find 28: go to index 8, then search inside that chain.
- Chaining is done with linked lists (you will learn it later). No need to implement it now.

### Collision

**Collision = many different keys end up at the same hash index.**

- If all keys (e.g. 18, 28, 38, 48, ...) end in 8, every one goes to index 8. The chain becomes huge and searching becomes **O(n)**. This is the **worst case** of unordered_map.
- Real libraries use a smarter mix of methods (not just plain division), so collisions are very rare. A test-case setter would have to design input specifically to break the hash, which is extremely hard.

### Other methods (names only)

- **Folding method**
- **Mid-square method**

Not required in depth. Just know that they exist. Division method + collision is what to understand.

---

## 11. Homework from the lecture

**Find the highest and lowest frequency element(s) in an array.**

Hints:
1. Build a frequency map (`map` or `unordered_map`, or an array if values are small).
2. Iterate over the map and keep track of max frequency and min frequency using `if / else`.

Sketch:

```cpp
unordered_map<int,int> mpp;
for (int i = 0; i < n; i++) mpp[arr[i]]++;

int maxFreq = 0, minFreq = n;
int maxEle = -1, minEle = -1;

for (auto it : mpp) {
    int count = it.second;
    int element = it.first;
    if (count > maxFreq) { maxFreq = count; maxEle = element; }
    if (count < minFreq) { minFreq = count; minEle = element; }
}
```

---

## 12. One Page Cheat Sheet

**Definition:** Hashing = pre-store (pre-compute) + fetch.

**Why:** Brute force for Q queries is O(Q * n). Hashing makes it O(n + Q).

**Decision guide**

| Data | Range | Use |
|------|-------|-----|
| Numbers | up to 10^6 (in main) or 10^7 (global) | Array hashing, size = max + 1 |
| Numbers | larger than that | `unordered_map` (or `map`) |
| Characters | any | Array: size 26 with `c - 'a'`, or size 256 with `c` |
| Keys like pairs / vectors | any | `map` only |
| Need sorted keys | any | `map` |

**Complexities**

| Structure | Store / Fetch |
|-----------|---------------|
| Array hashing | O(1) |
| `map` | O(log n) in all cases |
| `unordered_map` | O(1) avg and best, O(n) worst |

**Key facts to remember**

- 10^8 operations is about 1 second.
- Hash array size = max element + 1.
- Global arrays are auto-initialized to 0; local arrays are not.
- Max array size: about 10^6 in `main`, 10^7 globally (`int`).
- `'a'` = 97, `'A'` = 65, `'0'` = 48. Char used as an index auto converts to ASCII.
- `mpp[key]` on a missing key gives 0.
- `map` = sorted; `unordered_map` = random order.
- Collision = many keys landing on the same index. It is the cause of unordered_map's worst case.
- Java equivalent: `HashMap`.
- Default choice: `unordered_map`; if TLE, use `map`.

---

## 13. Quick Self-Test (Interview style)

1. **What is hashing?** Pre-storing data (like frequencies) so later lookups are fast.
2. **Why is hashing faster than looping for every query?** One O(n) precompute, then O(1) per query, instead of O(n) per query.
3. **Can we hash a number as big as 10^9 with an array?** No, memory limit. Use a map.
4. **How do you hash lowercase letters?** Array of 26, index `c - 'a'`.
5. **Difference between `map` and `unordered_map`?** Sorted vs unsorted, O(log n) vs O(1) average, key types allowed.
6. **What is a collision?** Multiple keys mapped to the same index.
7. **When is unordered_map O(n)?** When heavy collisions push all keys into one chain (worst case).
8. **What is chaining?** Storing colliding keys in a linked list at the same index.
