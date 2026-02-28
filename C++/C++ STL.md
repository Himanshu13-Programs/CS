# 📘 Day 1 — C++ STL: Complete Notes

---

## 📌 Table of Contents
1. [vector](#1-vector)
2. [map & unordered_map](#2-map--unordered_map)
3. [set & unordered_set](#3-set--unordered_set)
4. [sort](#4-sort)
5. [lower_bound & upper_bound](#5-lower_bound--upper_bound)
6. [Quick Reference Cheatsheet](#6-quick-reference-cheatsheet)
7. [Common Patterns & Tricks](#7-common-patterns--tricks)

---

## 1. `vector`

### 🧠 What is it?
A **dynamic array**. Think of it as an array that can grow and shrink. Under the hood it allocates memory in chunks (doubles capacity when full), so **amortized O(1)** for push_back.

### 🔧 Declaration
```cpp
vector<int> v;                    // empty vector
vector<int> v(5);                 // size 5, all 0s
vector<int> v(5, 10);            // size 5, all 10s
vector<int> v = {1, 2, 3, 4};   // initializer list
vector<vector<int>> grid(n, vector<int>(m, 0)); // 2D grid n×m filled with 0
```

### ⚙️ Core Operations & Complexities
| Operation | Syntax | Time |
|---|---|---|
| Add to end | `v.push_back(x)` | O(1) amortized |
| Remove from end | `v.pop_back()` | O(1) |
| Access element | `v[i]` or `v.at(i)` | O(1) |
| Size | `v.size()` | O(1) |
| Check empty | `v.empty()` | O(1) |
| Insert at pos | `v.insert(v.begin()+i, x)` | O(n) |
| Erase at pos | `v.erase(v.begin()+i)` | O(n) |
| Clear all | `v.clear()` | O(n) |
| First element | `v.front()` | O(1) |
| Last element | `v.back()` | O(1) |
| Resize | `v.resize(k)` | O(k) |

### 💡 Key Things to Know
- `v[i]` does **no bounds checking** — crashes silently if out of range
- `v.at(i)` throws `std::out_of_range` — safer for debugging
- **Always use `int i = 0; i < v.size()`** carefully — `v.size()` returns `size_t` (unsigned), so `v.size() - 1` when size=0 wraps to a huge number → use `(int)v.size()`

### 📝 Example — Prefix Sum using vector
```cpp
vector<int> nums = {1, 2, 3, 4, 5};
int n = nums.size();

vector<int> prefix(n + 1, 0);
for (int i = 0; i < n; i++) {
    prefix[i+1] = prefix[i] + nums[i];
}
// prefix = [0, 1, 3, 6, 10, 15]
// Sum from index l to r = prefix[r+1] - prefix[l]
int rangeSum = prefix[4] - prefix[1]; // sum of index 1..3 = 2+3+4 = 9
```

### 📝 Example — 2D Grid
```cpp
int n = 3, m = 4;
vector<vector<int>> grid(n, vector<int>(m, 0));

// Fill diagonal
for (int i = 0; i < min(n, m); i++) grid[i][i] = 1;

// Print
for (auto& row : grid) {
    for (int x : row) cout << x << " ";
    cout << "\n";
}
```

### ⚡ Iteration Patterns
```cpp
// Index-based (use when you need index)
for (int i = 0; i < (int)v.size(); i++) cout << v[i];

// Range-based (cleanest)
for (int x : v) cout << x;

// Reference (use when modifying)
for (int& x : v) x *= 2;

// With iterator
for (auto it = v.begin(); it != v.end(); it++) cout << *it;
```

---

## 2. `map` & `unordered_map`

### 🧠 What is it?
A **key-value store**.
- `map` → internally a **Red-Black Tree** → keys always **sorted** → O(log n) operations
- `unordered_map` → internally a **Hash Table** → keys **unsorted** → O(1) average operations

### 🔧 Declaration
```cpp
map<string, int> mp;                         // empty ordered map
map<int, int> freq;                          // frequency map
unordered_map<string, int> ump;              // hash map (faster, no order)
map<string, vector<int>> mp;                 // map of vectors (valid!)
```

### ⚙️ Core Operations
| Operation | map | unordered_map |
|---|---|---|
| Insert / Update | `mp[key] = val` | same |
| Access | `mp[key]` | same |
| Check exists | `mp.count(key)` or `mp.find(key) != mp.end()` | same |
| Delete | `mp.erase(key)` | same |
| Size | `mp.size()` | same |
| Iterate | sorted by key | arbitrary order |
| Time per op | O(log n) | O(1) avg, O(n) worst |

### ⚠️ Critical Gotcha
```cpp
map<string, int> mp;
cout << mp["hello"]; // This INSERTS "hello" with value 0 !!
                     // Use mp.count("hello") to check existence safely
```

### 💡 `count` vs `find`
```cpp
// count: returns 0 or 1, cleaner for existence check
if (mp.count(key)) { /* exists */ }

// find: returns iterator, use when you need the value too
auto it = mp.find(key);
if (it != mp.end()) {
    cout << it->second; // use value without double lookup
}
```

### 📝 Example — Frequency Count
```cpp
vector<int> nums = {1, 2, 2, 3, 3, 3, 4};
unordered_map<int, int> freq;

for (int x : nums) freq[x]++;
// freq = {1:1, 2:2, 3:3, 4:1}

// Find most frequent
int maxFreq = 0, maxVal = -1;
for (auto& [val, cnt] : freq) {  // structured binding (C++17)
    if (cnt > maxFreq) {
        maxFreq = cnt;
        maxVal = val;
    }
}
cout << maxVal; // 3
```

### 📝 Example — Two Sum using map
```cpp
vector<int> nums = {2, 7, 11, 15};
int target = 9;
unordered_map<int, int> seen; // val -> index

for (int i = 0; i < (int)nums.size(); i++) {
    int complement = target - nums[i];
    if (seen.count(complement)) {
        cout << seen[complement] << " " << i; // 0 1
        break;
    }
    seen[nums[i]] = i;
}
```

### 📝 Example — Group Anagrams
```cpp
vector<string> words = {"eat","tea","tan","ate","nat","bat"};
map<string, vector<string>> groups;

for (string& w : words) {
    string key = w;
    sort(key.begin(), key.end()); // sorted word = key
    groups[key].push_back(w);
}
// groups["aet"] = ["eat","tea","ate"]
// groups["ant"] = ["tan","nat"]
```

### 📝 Example — Ordered map for range queries
```cpp
// When you need keys in sorted order → use map not unordered_map
map<int, int> timeline; // time -> event_count
timeline[5]++;
timeline[3]++;
timeline[8]++;

// Iterate in sorted time order automatically
for (auto& [time, cnt] : timeline) {
    cout << time << ": " << cnt << "\n";
}
// 3: 1
// 5: 1
// 8: 1
```

---

## 3. `set` & `unordered_set`

### 🧠 What is it?
Stores **unique elements only**.
- `set` → Red-Black Tree → **sorted** → O(log n)
- `unordered_set` → Hash Table → **unsorted** → O(1) average

### 🔧 Declaration
```cpp
set<int> s;
set<int> s = {3, 1, 4, 1, 5, 9, 2, 6}; // duplicates removed automatically
unordered_set<string> us;
```

### ⚙️ Core Operations
| Operation | Syntax | Time (set) |
|---|---|---|
| Insert | `s.insert(x)` | O(log n) |
| Delete | `s.erase(x)` | O(log n) |
| Check exists | `s.count(x)` or `s.find(x) != s.end()` | O(log n) |
| Smallest element | `*s.begin()` | O(1) |
| Largest element | `*s.rbegin()` | O(1) |
| Size | `s.size()` | O(1) |

### 💡 Key Difference from map
- `set` stores only **keys** (no value). Use it when you only care about **existence** or **unique sorted elements**.

### 📝 Example — Remove Duplicates
```cpp
vector<int> nums = {4, 2, 7, 2, 4, 1, 7};
set<int> unique(nums.begin(), nums.end());
vector<int> result(unique.begin(), unique.end());
// result = [1, 2, 4, 7] (sorted!)
```

### 📝 Example — Sliding Window with set (Longest Substring No Repeat)
```cpp
string s = "abcabcbb";
unordered_set<char> window;
int left = 0, maxLen = 0;

for (int right = 0; right < (int)s.size(); right++) {
    while (window.count(s[right])) {
        window.erase(s[left++]); // shrink from left
    }
    window.insert(s[right]);
    maxLen = max(maxLen, right - left + 1);
}
cout << maxLen; // 3 ("abc")
```

### 📝 Example — set for "Next Greater in Sorted Order"
```cpp
set<int> s = {1, 3, 5, 7, 9};
int target = 4;

// Find smallest element > target
auto it = s.upper_bound(4); // points to 5
if (it != s.end()) cout << *it; // 5

// Find largest element < target
auto it2 = s.lower_bound(4); // points to 5 (first >= 4)
if (it2 != s.begin()) {
    --it2;
    cout << *it2; // 3
}
```

### ⚠️ multiset — when duplicates matter
```cpp
multiset<int> ms = {1, 2, 2, 3, 3, 3};
ms.erase(ms.find(2)); // erase ONE occurrence of 2
// ms = {1, 2, 3, 3, 3}

ms.erase(3); // erases ALL 3s !!
// ms = {1, 2}
```

---

## 4. `sort`

### 🧠 What is it?
`std::sort` uses **IntroSort** (hybrid of QuickSort + HeapSort + InsertionSort). Always **O(n log n)**, never degrades.

### 🔧 Basic Usage
```cpp
#include <algorithm>

vector<int> v = {5, 2, 8, 1, 9, 3};

sort(v.begin(), v.end());              // ascending: [1,2,3,5,8,9]
sort(v.begin(), v.end(), greater<int>()); // descending: [9,8,5,3,2,1]

// Sort array
int arr[] = {5, 2, 8};
sort(arr, arr + 3);
```

### 🔧 Custom Comparator
The comparator must return `true` if `a` should come **before** `b`.

```cpp
// Sort by second element of pair
vector<pair<int,int>> v = {{1,3},{2,1},{3,2}};
sort(v.begin(), v.end(), [](auto& a, auto& b){
    return a.second < b.second;
});
// v = {{2,1},{3,2},{1,3}}
```

### 📝 Example — Sort strings by length
```cpp
vector<string> words = {"banana", "kiwi", "apple", "fig"};
sort(words.begin(), words.end(), [](const string& a, const string& b){
    return a.size() < b.size(); // shorter first
});
// words = ["fig", "kiwi", "apple", "banana"]
```

### 📝 Example — Sort by multiple criteria
```cpp
// Sort by frequency ascending, then by value descending
vector<int> nums = {1,1,2,3,3,3,2};
unordered_map<int,int> freq;
for (int x : nums) freq[x]++;

sort(nums.begin(), nums.end(), [&](int a, int b){
    if (freq[a] != freq[b]) return freq[a] < freq[b]; // lower freq first
    return a > b;                                       // higher val first if same freq
});
```

### 📝 Example — Sort intervals by start time
```cpp
vector<vector<int>> intervals = {{1,3},{2,6},{8,10},{15,18}};
sort(intervals.begin(), intervals.end()); // sorts by first element by default

// Custom: sort by end time (greedy interval problems)
sort(intervals.begin(), intervals.end(), [](auto& a, auto& b){
    return a[1] < b[1];
});
```

### 💡 `stable_sort` — preserves relative order of equal elements
```cpp
stable_sort(v.begin(), v.end(), comparator);
// Use when equal elements must maintain original order
```

### 📝 Example — partial_sort (only need top K)
```cpp
vector<int> v = {5, 2, 8, 1, 9, 3, 7};
partial_sort(v.begin(), v.begin() + 3, v.end()); // sort only first 3
// v = [1, 2, 3, 9, 8, 5, 7] — first 3 are sorted, rest are in any order
```

---

## 5. `lower_bound` & `upper_bound`

### 🧠 What is it?
Binary search functions on **sorted sequences**. O(log n).

| Function | Returns iterator to... |
|---|---|
| `lower_bound(begin, end, x)` | First element **≥ x** |
| `upper_bound(begin, end, x)` | First element **> x** |

> ⚠️ **MUST be used on sorted data.** Results are undefined on unsorted arrays.

### 🔧 On vectors
```cpp
vector<int> v = {1, 2, 4, 4, 4, 6, 8};
//               0  1  2  3  4  5  6

auto lb = lower_bound(v.begin(), v.end(), 4); // iterator to index 2 (first 4)
auto ub = upper_bound(v.begin(), v.end(), 4); // iterator to index 5 (first 6)

// Convert to index
int lb_idx = lb - v.begin(); // 2
int ub_idx = ub - v.begin(); // 5

// Count occurrences of 4
int count = ub - lb; // 5 - 2 = 3 ✓

// Check if element exists
if (lb != v.end() && *lb == 4) cout << "Found!";

// Find first element >= 5
auto it = lower_bound(v.begin(), v.end(), 5);
cout << *it; // 6

// Find last element <= 5
auto it2 = upper_bound(v.begin(), v.end(), 5);
if (it2 != v.begin()) {
    --it2;
    cout << *it2; // 4
}
```

### 🔧 On set/map (use member functions — faster!)
```cpp
set<int> s = {1, 3, 5, 7, 9};

// Member function versions — O(log n) using tree structure
auto it = s.lower_bound(4); // points to 5
auto it2 = s.upper_bound(5); // points to 7

// ⚠️ std::lower_bound(s.begin(), s.end(), x) also works but is O(n) on sets!
// Always use s.lower_bound(x) for set/map — it's the member function
```

### 📝 Example — Search Insert Position (LC 35)
```cpp
// Given sorted array and target, find index to insert target
vector<int> nums = {1, 3, 5, 6};
int target = 5;

auto it = lower_bound(nums.begin(), nums.end(), target);
cout << (it - nums.begin()); // 2 (index of 5, or where to insert)

target = 2;
it = lower_bound(nums.begin(), nums.end(), target);
cout << (it - nums.begin()); // 1 (insert between index 0 and 1)
```

### 📝 Example — Count elements in range [lo, hi]
```cpp
vector<int> v = {1, 2, 4, 4, 4, 6, 8, 9};
int lo = 3, hi = 7;

auto left  = lower_bound(v.begin(), v.end(), lo);  // first >= 3 → index 2
auto right = upper_bound(v.begin(), v.end(), hi);  // first > 7  → index 7

cout << right - left; // 5 (elements: 4,4,4,6 → wait, let me recount)
// elements in [3,7]: 4,4,4,6 = 4 elements
```

### 📝 Example — Binary Search on Answer Space
```cpp
// "Find minimum maximum subarray sum when split into k parts"
// Binary search on answer [max_element, total_sum]

vector<int> nums = {7, 2, 5, 10, 8};
int k = 2;

auto canSplit = [&](int maxSum) {
    int parts = 1, curr = 0;
    for (int x : nums) {
        if (curr + x > maxSum) { parts++; curr = 0; }
        curr += x;
    }
    return parts <= k;
};

int lo = *max_element(nums.begin(), nums.end()); // 10
int hi = accumulate(nums.begin(), nums.end(), 0); // 32
int ans = hi;

while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;
    if (canSplit(mid)) { ans = mid; hi = mid - 1; }
    else lo = mid + 1;
}
cout << ans; // 18
```

### 📝 Example — lower_bound on map
```cpp
map<int, string> schedule = {{8,"breakfast"},{12,"lunch"},{18,"dinner"}};
int query = 10;

// First event at or after query time
auto it = schedule.lower_bound(query);
if (it != schedule.end())
    cout << it->first << ": " << it->second; // 12: lunch

// Last event before or at query time
if (it != schedule.begin()) {
    --it;
    cout << it->first << ": " << it->second; // 8: breakfast
}
```

---

## 6. Quick Reference Cheatsheet

```
┌─────────────────────────────────────────────────────────────────┐
│                     STL QUICK REFERENCE                         │
├──────────────┬───────────────┬─────────────────────────────────┤
│  Container   │  Use When     │  Key Operations                  │
├──────────────┼───────────────┼─────────────────────────────────┤
│ vector<T>    │ dynamic array │ push_back, [], size, resize      │
│ map<K,V>     │ sorted KV     │ [key], count, find, erase        │
│ unord_map    │ fast KV       │ same, O(1) avg                   │
│ set<T>       │ unique sorted │ insert, erase, count, begin/end  │
│ unord_set    │ fast unique   │ same, O(1) avg                   │
├──────────────┼───────────────┼─────────────────────────────────┤
│ lower_bound  │ first >= x    │ returns iterator                 │
│ upper_bound  │ first > x     │ returns iterator                 │
│ sort         │ sort range    │ O(n log n), custom comparator    │
└──────────────┴───────────────┴─────────────────────────────────┘

COMPLEXITY SUMMARY:
  vector  → access O(1), insert/delete end O(1), middle O(n)
  map     → all ops O(log n)
  unordered_map → all ops O(1) avg, O(n) worst (hash collision)
  set     → all ops O(log n)
  sort    → O(n log n) always
  lower/upper_bound → O(log n) on sorted data
```

---

## 7. Common Patterns & Tricks

### 🔥 Pattern 1 — Frequency Map (use everywhere)
```cpp
unordered_map<int, int> freq;
for (int x : nums) freq[x]++;
// Then query: freq[x] gives count of x
```

### 🔥 Pattern 2 — Coordinate Compression
```cpp
// When values are huge but you need indices
vector<int> vals = {100, 500, 200, 100, 500};
vector<int> sorted_unique = vals;
sort(sorted_unique.begin(), sorted_unique.end());
sorted_unique.erase(unique(sorted_unique.begin(), sorted_unique.end()), sorted_unique.end());

// Map original value to compressed index
for (int& x : vals) {
    x = lower_bound(sorted_unique.begin(), sorted_unique.end(), x) - sorted_unique.begin();
}
// vals = [0, 2, 1, 0, 2]
```

### 🔥 Pattern 3 — Pair sorting trick
```cpp
// To sort indices by value without losing index info
vector<int> nums = {5, 2, 8, 1};
vector<pair<int,int>> indexed;
for (int i = 0; i < nums.size(); i++) indexed.push_back({nums[i], i});
sort(indexed.begin(), indexed.end()); // sorted by value
// indexed = [{1,3},{2,1},{5,0},{8,2}] — value,originalIndex
```

### 🔥 Pattern 4 — Set as sliding window
```cpp
// Detect if window has all unique elements
set<char> window;
int left = 0;
for (int right = 0; right < s.size(); right++) {
    while (window.count(s[right])) window.erase(s[left++]);
    window.insert(s[right]);
    // window is now guaranteed to have all unique chars
}
```

### 🔥 Pattern 5 — Binary search on sorted + rotated
```cpp
// Always check which half is sorted first
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;
    if (nums[mid] == target) return mid;
    if (nums[lo] <= nums[mid]) { // left half sorted
        if (nums[lo] <= target && target < nums[mid]) hi = mid - 1;
        else lo = mid + 1;
    } else { // right half sorted
        if (nums[mid] < target && target <= nums[hi]) lo = mid + 1;
        else hi = mid - 1;
    }
}
```

### 🔥 Pattern 6 — Map for "last seen index"
```cpp
unordered_map<int, int> lastSeen; // val -> last index
for (int i = 0; i < nums.size(); i++) {
    if (lastSeen.count(nums[i])) {
        int dist = i - lastSeen[nums[i]]; // distance to last occurrence
    }
    lastSeen[nums[i]] = i;
}
```

---

## ⚠️ Common Mistakes to Avoid

| Mistake | Why it's wrong | Fix |
|---|---|---|
| `v.size() - 1` when v is empty | size_t underflow → huge number | `(int)v.size() - 1` |
| `mp["key"]` to check existence | Creates entry with 0 | `mp.count("key")` |
| `lower_bound` on unsorted data | Undefined behavior | Sort first |
| `s.erase(val)` on multiset | Erases ALL copies | `s.erase(s.find(val))` |
| `std::lower_bound` on set | O(n) not O(log n) | Use `s.lower_bound(val)` |
| Forgetting `&` in range-for | Copies each element | `for (auto& x : v)` |

---

## 🎯 Problems to Practice These Concepts

| Problem | LC # | Concept Used |
|---|---|---|
| Two Sum | 1 | unordered_map |
| Contains Duplicate | 217 | unordered_set |
| Longest Consecutive Sequence | 128 | unordered_set |
| Valid Anagram | 242 | map / sort |
| Group Anagrams | 49 | map + sort |
| Search Insert Position | 35 | lower_bound |
| Find Minimum in Rotated Array | 153 | binary search |
| Count of Range Sum | 327 | lower_bound on sorted |
| Top K Frequent Elements | 347 | map + sort |
| Merge Intervals | 56 | sort by start |

---

*Notes by: Version2026 Prep | Next: Day 2 — OS Basics (Processes, Memory, Scheduling)*