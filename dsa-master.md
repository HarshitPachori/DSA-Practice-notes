# DSA Master Patterns Reference

Built on top of the Rising Brain pattern-wise sheet. Structure follows the sheet exactly: **Topic → Pattern → Sub-pattern**, so you can map every problem you pick directly to a trigger and template below.

# How to actually use this sheet

Work topic by topic in the order above — it's sequenced so each topic's mental tools (two-pointer, then sliding window, then prefix sums) get reused by the next one, rather than jumping around and re-learning fundamentals mid-stream.

For each pattern: read the trigger, attempt one Easy and one Medium problem from the list cold, using only the trigger as a hint. If you solve it, move to the next sub-pattern. If you get stuck past 15-20 minutes, look at the template, understand *why* it's shaped that way, then close it and re-attempt from memory. Come back to any pattern you struggled with after finishing the topic, not immediately — the gap is what tests whether it actually stuck.

Cross-check tricky spots against a second source when the trigger and template alone don't fully click — NeetCode's videos are strong for visual intuition on Trees/Graphs/DP, takeUforward is strong for the binary-search-on-answers and DP-on-strings templates specifically, and Aditya Verma's DP playlist remains one of the clearest walkthroughs of the Knapsack family if the state-transition reasoning above doesn't land on first read.


---

# 1. Array

Fundamental collection of elements stored at contiguous memory locations. Almost every other pattern in this sheet builds on array traversal fundamentals, so treat this topic as the foundation, not just another checkbox.

## Pattern: Two-Pointer

**Trigger.** The array is sorted, or can be sorted without losing the answer. You're looking for a pair, triplet, or a way to shrink a search range from both ends at once. If you find yourself writing a nested loop to check every pair, stop — ask whether sorting first turns that O(n²) into O(n) with two pointers.

**Constraints signal.** n up to 10⁵–10⁶ almost always means the intended solution is O(n) or O(n log n). A brute-force O(n²) pair check will time out past n ~ 10⁴.

```java
public int[] twoPointerSandwich(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1;

    while (left < right) {
        int currentSum = nums[left] + nums[right];
        if (currentSum == target) {
            return new int[]{left, right};
        }
        if (currentSum < target) {
            left++;
        } else {
            right--;
        }
    }
    return new int[]{-1, -1};
}
```

**Triplet variant (3Sum shape) — fix one, two-pointer the rest:**

```java
public List<List<Integer>> threeSum(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> result = new ArrayList<>();

    for (int i = 0; i < nums.length - 2; i++) {
        if (i > 0 && nums[i] == nums[i - 1]) continue; // skip duplicate anchors
        int left = i + 1, right = nums.length - 1;
        while (left < right) {
            int sum = nums[i] + nums[left] + nums[right];
            if (sum == 0) {
                result.add(List.of(nums[i], nums[left], nums[right]));
                while (left < right && nums[left] == nums[left + 1]) left++;
                while (left < right && nums[right] == nums[right - 1]) right--;
                left++;
                right--;
            } else if (sum < 0) {
                left++;
            } else {
                right--;
            }
        }
    }
    return result;
}
```

**Edge cases.**
- Empty array or single element — return early, no pair possible.
- All identical elements — duplicate-skip logic must still be correct, don't just rely on "it probably works."
- Target achievable only using the same index twice — explicitly disallowed unless the problem says otherwise; `left < right` guards this.
- Array not sorted going in, but the problem needs sortedness — sorting changes original indices, so if the answer needs original positions (like classic Two Sum, not Two Sum II), two-pointer on a sorted copy won't work; use a HashMap instead.

**Problems in this sub-pattern:** Move Zeroes, Two Sum II, 3Sum, Sort Colors, Container With Most Water, Trapping Rain Water.

---

## Pattern: Sliding Window

**Trigger.** You need something about a *contiguous* subarray — a sum, a count, a max, a length — and re-scanning the whole subarray from scratch at every position would be wasteful. The window either has a fixed size k, or a size that grows and shrinks based on a condition.

**Constraints signal.** n up to 10⁵, and the naive approach recomputes a sum or count for every window from scratch, giving O(n·k) or O(n²). Sliding window brings this to O(n) because you update the window incrementally instead of recomputing.

### Sub-pattern: Fixed Size

```java
public int fixedSlidingWindow(int[] nums, int k) {
    int windowSum = 0;
    int maxVal = Integer.MIN_VALUE;

    for (int right = 0; right < nums.length; right++) {
        windowSum += nums[right];
        if (right >= k - 1) {
            maxVal = Math.max(maxVal, windowSum);
            windowSum -= nums[right - k + 1];
        }
    }
    return maxVal;
}
```

### Sub-pattern: Variable Size (grow until invalid, shrink until valid again)

```java
public int minSubArrayLen(int target, int[] nums) {
    int left = 0;
    long currentSum = 0;
    int minLen = Integer.MAX_VALUE;

    for (int right = 0; right < nums.length; right++) {
        currentSum += nums[right];
        while (currentSum >= target) {
            minLen = Math.min(minLen, right - left + 1);
            currentSum -= nums[left];
            left++;
        }
    }
    return minLen == Integer.MAX_VALUE ? 0 : minLen;
}
```

**Edge cases.**
- k larger than array length — return early or handle explicitly, don't let the loop underflow.
- Negative numbers in the array break the "sum grows monotonically as window grows" assumption that variable sliding window depends on. If negatives are allowed, this pattern doesn't apply — fall back to prefix sum + HashMap.
- "At most K distinct" vs "exactly K distinct" is a classic trap — exactly-K is usually solved as atMost(K) − atMost(K−1), not as a direct window condition.
- Sliding Window Maximum needs a monotonic deque, not a plain running max, because you need O(1) access to the max as elements leave the window from the left — see Monotonic Stack/Deque pattern.

**Problems:** Maximum Sum Subarray of Size K, Max Consecutive Ones, Max Consecutive Ones III, Subarray Product Less Than K, Fruits Into Baskets, Minimum Size Subarray Sum, Sliding Window Maximum, Subarray with K Distinct Integers.

---

## Pattern: Prefix Sum

**Trigger.** Repeated range-sum queries on a static array, or a subarray-sum problem where two-pointer fails because the array contains negative numbers.

**Constraints signal.** Q queries against an array of size n — brute force is O(n) per query, O(nQ) total. Prefix sum preprocesses in O(n), then answers each query in O(1).

```java
public int[] buildPrefixSum(int[] nums) {
    int[] prefix = new int[nums.length + 1];
    for (int i = 0; i < nums.length; i++) {
        prefix[i + 1] = prefix[i] + nums[i];
    }
    return prefix; // range sum [L, R] inclusive = prefix[R + 1] - prefix[L]
}
```

**2D variant (matrix block sum):**

```java
public int[][] buildMatrixPrefixSum(int[][] mat) {
    int rows = mat.length, cols = mat[0].length;
    int[][] prefix = new int[rows + 1][cols + 1];
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            prefix[r + 1][c + 1] = mat[r][c] + prefix[r][c + 1] + prefix[r + 1][c] - prefix[r][c];
        }
    }
    return prefix;
}
```

**Edge cases.**
- Integer overflow on large cumulative sums — use `long[]` when values or array size are large.
- 1-indexed prefix array is deliberate — makes `prefix[R+1] - prefix[L]` clean without special-casing L=0. Don't collapse it back to 0-indexed and reintroduce off-by-one bugs.
- Modulo arithmetic (Subarray Sum Divisible by K) needs Java's `%` handled carefully — Java's modulo can return negative values for negative dividends; normalize with `((x % k) + k) % k`.

**Problems:** Find Pivot Index, Subarray Sum Equals K, Matrix Block Sum, Product of Array Except Self, Continuous Subarray Sum, Subarray Sum Divisible by K.

---

## Pattern: Kadane's Algorithm

**Trigger.** Maximum (or minimum) sum of a contiguous subarray. Signature phrase: "contiguous subarray" plus "maximum sum" or "maximum product."

**Constraints signal.** O(n) time, O(1) space is achievable and expected — if you're reaching for DP with an array, you've overcomplicated it.

```java
public int maxSubArray(int[] nums) {
    int maxSoFar = nums[0];
    int currentMax = nums[0];

    for (int i = 1; i < nums.length; i++) {
        currentMax = Math.max(nums[i], currentMax + nums[i]);
        maxSoFar = Math.max(maxSoFar, currentMax);
    }
    return maxSoFar;
}
```

**Product variant — track both max and min, since a negative times a negative flips the sign:**

```java
public int maxProduct(int[] nums) {
    int maxSoFar = nums[0], curMax = nums[0], curMin = nums[0];

    for (int i = 1; i < nums.length; i++) {
        int candidateMax = curMax * nums[i];
        int candidateMin = curMin * nums[i];
        curMax = Math.max(nums[i], Math.max(candidateMax, candidateMin));
        curMin = Math.min(nums[i], Math.min(candidateMax, candidateMin));
        maxSoFar = Math.max(maxSoFar, curMax);
    }
    return maxSoFar;
}
```

**Edge cases.**
- All-negative array — don't reset `currentMax` to 0 on negative; that silently produces 0 as an answer when the correct answer is the least-negative element.
- Circular subarray variant (Maximum Sum Circular Subarray) needs total sum minus the *minimum* subarray sum as a second case, then take the max of that and the normal Kadane result — and you must separately guard against the case where the whole array is negative (minSubarray == total sum), which would wrongly return 0.
- Single-element array — should just return that element.

**Problems:** Maximum Subarray, Maximum Product Subarray, Maximum Sum Circular Subarray, Maximum Absolute Sum of Any Subarray, Largest Sum Contiguous Subarray.

---

# 2. Strings

Sequence of characters. Most string patterns are array patterns wearing a costume — same two-pointer and sliding-window mechanics, but character comparisons and frequency maps replace numeric sums.

## Pattern: Two-Pointer (Palindrome)

**Trigger.** Checking or building palindromes, or comparing a string against its reverse without actually allocating the reversed string.

```java
public boolean isPalindrome(String s) {
    int left = 0, right = s.length() - 1;
    while (left < right) {
        while (left < right && !Character.isLetterOrDigit(s.charAt(left))) left++;
        while (left < right && !Character.isLetterOrDigit(s.charAt(right))) right--;
        if (Character.toLowerCase(s.charAt(left)) != Character.toLowerCase(s.charAt(right))) {
            return false;
        }
        left++;
        right--;
    }
    return true;
}
```

**Expand-around-center (Longest Palindromic Substring / Palindromic Substrings):**

```java
public String longestPalindrome(String s) {
    int start = 0, maxLen = 0;

    for (int center = 0; center < s.length(); center++) {
        int len1 = expandFromCenter(s, center, center);       // odd length
        int len2 = expandFromCenter(s, center, center + 1);   // even length
        int len = Math.max(len1, len2);
        if (len > maxLen) {
            maxLen = len;
            start = center - (len - 1) / 2;
        }
    }
    return s.substring(start, start + maxLen);
}

private int expandFromCenter(String s, int left, int right) {
    while (left >= 0 && right < s.length() && s.charAt(left) == s.charAt(right)) {
        left--;
        right++;
    }
    return right - left - 1;
}
```

**Edge cases.**
- Empty string or single character — trivially a palindrome.
- Case sensitivity and non-alphanumeric characters (Valid Palindrome) — normalize before comparing, don't just compare raw chars.
- Valid Palindrome II (allow removing exactly one character) — when the first mismatch is found, you must try skipping *either* the left or the right character and check if the remainder is a palindrome; don't just always skip one side.

**Problems:** Reverse a String, Valid Palindrome, Valid Palindrome II, Longest Palindromic Substring, Palindromic Substrings.

---

## Pattern: Sliding Window (String)

**Trigger.** Substring problems involving character constraints — no repeats, exactly K unique characters, matching a target frequency map (anagram-style).

```java
public int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> lastSeen = new HashMap<>();
    int left = 0, maxLen = 0;

    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        if (lastSeen.containsKey(c) && lastSeen.get(c) >= left) {
            left = lastSeen.get(c) + 1;
        }
        lastSeen.put(c, right);
        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

**Frequency-map window (Minimum Window Substring / Find All Anagrams):**

```java
public String minWindow(String s, String t) {
    if (s.isEmpty() || t.isEmpty()) return "";

    Map<Character, Integer> need = new HashMap<>();
    for (char c : t.toCharArray()) need.merge(c, 1, Integer::sum);

    Map<Character, Integer> window = new HashMap<>();
    int have = 0, needCount = need.size();
    int left = 0, resStart = 0, resLen = Integer.MAX_VALUE;

    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        window.merge(c, 1, Integer::sum);
        if (need.containsKey(c) && window.get(c).intValue() == need.get(c).intValue()) {
            have++;
        }

        while (have == needCount) {
            if (right - left + 1 < resLen) {
                resLen = right - left + 1;
                resStart = left;
            }
            char leftChar = s.charAt(left);
            window.merge(leftChar, -1, Integer::sum);
            if (need.containsKey(leftChar) && window.get(leftChar) < need.get(leftChar)) {
                have--;
            }
            left++;
        }
    }
    return resLen == Integer.MAX_VALUE ? "" : s.substring(resStart, resStart + resLen);
}
```

**Edge cases.**
- t longer than s — no valid window exists, return early.
- Duplicate characters in the target pattern (t) — frequency comparison must be exact-count, not just presence/absence.
- Fixed-size window variant (Permutation in String, Find All Anagrams) can use a simpler fixed window with array-based counts (size 26) instead of a HashMap, which is faster in practice.

**Problems:** Find All Anagrams in a String, Longest Substring Without Repeating Characters, Longest Substring with K Uniques, Permutation in String, Minimum Window Substring, Substring with Concatenation of All Words.

---

# 3. Binary Search

Efficient search over a monotonic space. The core discipline here isn't the loop itself — it's correctly identifying what's monotonic. Once you know what property is monotonic, the search is mechanical.

## Pattern: Classic Binary Search

**Trigger.** Sorted array, or a search space where "if X works, does X+1 also work" (or the reverse) holds true — that's what monotonic really means, and it doesn't require the raw values to be sorted, just the *feasibility* of an answer.

```java
public int classicBinarySearch(int[] nums, int target) {
    int low = 0, high = nums.length - 1;

    while (low <= high) {
        int mid = low + (high - low) / 2; // avoids overflow
        if (nums[mid] == target) return mid;
        if (nums[mid] < target) {
            low = mid + 1;
        } else {
            high = mid - 1;
        }
    }
    return -1;
}
```

**Rotated sorted array — figure out which half is properly ordered, then decide:**

```java
public int searchRotated(int[] nums, int target) {
    int low = 0, high = nums.length - 1;

    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (nums[mid] == target) return mid;

        if (nums[low] <= nums[mid]) { // left half is sorted
            if (nums[low] <= target && target < nums[mid]) {
                high = mid - 1;
            } else {
                low = mid + 1;
            }
        } else { // right half is sorted
            if (nums[mid] < target && target <= nums[high]) {
                low = mid + 1;
            } else {
                high = mid - 1;
            }
        }
    }
    return -1;
}
```

**Edge cases.**
- `(low + high) / 2` overflows for large arrays — always use `low + (high - low) / 2`.
- Duplicates in a rotated array (not in the base problem, but in variants) break the "which half is sorted" check when `nums[low] == nums[mid]` — you may need to shrink `low` by 1 in that specific tie case.
- Empty array — return -1 immediately, don't let the loop even start incorrectly.

**Problems:** Binary Search, Sqrt(x), Search Insert Position, Search in Rotated Sorted Array, Find Minimum in Rotated Sorted Array, Find Peak Element.

---

## Pattern: Lower / Upper Bound

**Trigger.** "First occurrence," "last occurrence," "smallest index where condition holds," or "insertion point" — anywhere you need a boundary rather than an exact match.

```java
public int findLowerBound(int[] nums, int target) {
    int low = 0, high = nums.length - 1;
    int ans = nums.length; // default: insert at end

    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (nums[mid] >= target) {
            ans = mid;
            high = mid - 1; // keep looking left for an earlier valid index
        } else {
            low = mid + 1;
        }
    }
    return ans;
}

public int findUpperBound(int[] nums, int target) {
    int low = 0, high = nums.length - 1;
    int ans = nums.length;

    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (nums[mid] > target) {
            ans = mid;
            high = mid - 1;
        } else {
            low = mid + 1;
        }
    }
    return ans;
}
```

**Edge cases.**
- Target absent entirely — `ans` may equal `nums.length` (past the last index) or point to a non-matching value; always verify `nums[ans] == target` before treating it as a real match, not just an insertion point.
- First-and-last-position problems are just lowerBound and upperBound-minus-one called back to back — don't write a separate scan for each.

**Problems:** Find Kth Rotation, Count Occurrences, Ceiling in a Sorted Array, Floor in a Sorted Array, Find First and Last Position of Element.

---

## Pattern: Binary Search on Answers

**Trigger.** The problem asks you to "minimize the maximum" or "maximize the minimum" of something, and checking whether a *candidate answer* is feasible is easy (usually a linear scan), even though computing the optimal answer directly isn't obvious. This is the single highest-leverage pattern to internalize — it shows up disguised in a lot of "greedy-looking" problems.

```java
public int minEatingSpeed(int[] piles, int h) {
    int low = 1, high = Arrays.stream(piles).max().getAsInt();
    int ans = high;

    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (isFeasible(piles, h, mid)) {
            ans = mid;
            high = mid - 1; // try a smaller (slower) speed
        } else {
            low = mid + 1;
        }
    }
    return ans;
}

private boolean isFeasible(int[] piles, int h, int speed) {
    long hoursNeeded = 0;
    for (int pile : piles) {
        hoursNeeded += (pile + speed - 1) / speed; // ceiling division
    }
    return hoursNeeded <= h;
}
```

**Edge cases.**
- Search range must start no lower than what's actually achievable — for Koko, `low` must be at least 1, and for capacity/allocation-style problems, `low` must be at least the maximum single element (you can never ship/allocate less than the heaviest single item).
- Off-by-one in "minimize" vs "maximize" direction — when minimizing, shrink `high` on success; when maximizing, raise `low` on success. Mixing these up is the most common bug in this pattern.
- Feasibility function itself needs to be correct and monotonic — if `isFeasible(x)` being true doesn't guarantee `isFeasible(x+1)` is also true, binary search on answers doesn't apply at all.

**Problems:** Koko Eating Bananas, Capacity To Ship Packages Within D Days, Min Speed to Arrive on Time, Aggressive Cows, Minimum Days to Make M Bouquets, Magnetic Force Between Two Balls, Allocate Minimum Number of Pages, Split Array Largest Sum.

---

## Pattern: Search in 2D Matrix

**Trigger.** A matrix that's sorted either fully (as if flattened) or row-wise and column-wise, and you need to locate a target or the kth smallest efficiently.

```java
public boolean searchMatrix(int[][] matrix, int target) {
    int rows = matrix.length, cols = matrix[0].length;
    int low = 0, high = rows * cols - 1;

    while (low <= high) {
        int mid = low + (high - low) / 2;
        int midVal = matrix[mid / cols][mid % cols];
        if (midVal == target) return true;
        if (midVal < target) low = mid + 1; else high = mid - 1;
    }
    return false;
}
```

**Staircase search (Matrix II — sorted rows and columns independently, not fully sorted):**

```java
public boolean searchMatrixII(int[][] matrix, int target) {
    int row = 0, col = matrix[0].length - 1;

    while (row < matrix.length && col >= 0) {
        if (matrix[row][col] == target) return true;
        if (matrix[row][col] > target) col--; else row++;
    }
    return false;
}
```

**Edge cases.**
- Don't confuse "fully sorted as if flattened" (Matrix I — treat as 1D, binary search directly) with "sorted per row and per column but not globally" (Matrix II — needs the staircase walk from a corner, not flattened binary search).
- Empty matrix or empty row — guard before indexing.

**Problems:** Search a 2D Matrix, Search a 2D Matrix II, Kth Smallest Element in Sorted Matrix, Matrix Median.

---

# 4. Stack

LIFO structure. The recurring insight across every sub-pattern here: a stack lets you defer a decision about an earlier element until you have enough information from later elements to make it correctly.

## Pattern: Monotonic Stack

**Trigger.** "Next greater element," "previous smaller element," histogram-style area problems, or anything about the nearest boundary to the left/right that satisfies an ordering condition.

```java
public int[] nextGreaterElement(int[] nums) {
    int[] result = new int[nums.length];
    Arrays.fill(result, -1);
    Deque<Integer> stack = new ArrayDeque<>(); // stores indices

    for (int i = 0; i < nums.length; i++) {
        while (!stack.isEmpty() && nums[stack.peek()] < nums[i]) {
            result[stack.pop()] = nums[i];
        }
        stack.push(i);
    }
    return result;
}
```

**Largest Rectangle in Histogram — the canonical hard version of this pattern:**

```java
public int largestRectangleArea(int[] heights) {
    Deque<Integer> stack = new ArrayDeque<>();
    int maxArea = 0;

    for (int i = 0; i <= heights.length; i++) {
        int currentHeight = (i == heights.length) ? 0 : heights[i];
        while (!stack.isEmpty() && heights[stack.peek()] > currentHeight) {
            int height = heights[stack.pop()];
            int width = stack.isEmpty() ? i : i - stack.peek() - 1;
            maxArea = Math.max(maxArea, height * width);
        }
        stack.push(i);
    }
    return maxArea;
}
```

**Edge cases.**
- Push indices, not values — you need to map back to original positions and compute distances/widths.
- Circular array variants (Next Greater Element II) need two passes over the array (`i % n`) instead of one, since elements can wrap around.
- Sentinel value (the `i == heights.length` trick above) simplifies flushing the stack at the end — without it you need a separate cleanup loop after the main one.

**Problems:** Next Greater Element I & II, Daily Temperatures, Online Stock Span, Asteroid Collision, Largest Rectangle in Histogram, Maximal Rectangle.

---

## Pattern: Expression Evaluation

**Trigger.** Calculator-style string parsing, postfix/infix evaluation, or nested decoding (like `3[a2[c]]` style strings) where operator precedence or nesting needs explicit tracking.

```java
public int calculate(String s) {
    Deque<Integer> stack = new ArrayDeque<>();
    int currentNum = 0;
    char sign = '+';

    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        if (Character.isDigit(c)) {
            currentNum = currentNum * 10 + (c - '0');
        }
        if ((!Character.isDigit(c) && c != ' ') || i == s.length() - 1) {
            switch (sign) {
                case '+' -> stack.push(currentNum);
                case '-' -> stack.push(-currentNum);
                case '*' -> stack.push(stack.pop() * currentNum);
                case '/' -> stack.push(stack.pop() / currentNum);
            }
            sign = c;
            currentNum = 0;
        }
    }

    int total = 0;
    for (int val : stack) total += val;
    return total;
}
```

**Decode String (nested bracket expansion):**

```java
public String decodeString(String s) {
    Deque<Integer> countStack = new ArrayDeque<>();
    Deque<StringBuilder> stringStack = new ArrayDeque<>();
    StringBuilder current = new StringBuilder();
    int k = 0;

    for (char c : s.toCharArray()) {
        if (Character.isDigit(c)) {
            k = k * 10 + (c - '0');
        } else if (c == '[') {
            countStack.push(k);
            stringStack.push(current);
            current = new StringBuilder();
            k = 0;
        } else if (c == ']') {
            StringBuilder decoded = stringStack.pop();
            int repeat = countStack.pop();
            decoded.append(String.valueOf(current).repeat(repeat));
            current = decoded;
        } else {
            current.append(c);
        }
    }
    return current.toString();
}
```

**Edge cases.**
- Multi-digit numbers inside the expression — accumulate digits before hitting an operator, don't process one digit at a time.
- Trailing token at end of string — the last number/operator has no following operator character to trigger processing, handle it with an explicit end-of-string check (`i == s.length() - 1`).
- Division truncation toward zero vs. floor division — Java's `/` truncates toward zero for integers, which matches most calculator problems, but double-check the problem statement.

**Problems:** Basic Calculator I & II, Evaluate Reverse Polish Notation, Decode String.

---

## Pattern: Stack Simulation / Undo Operation

**Trigger.** The problem describes a sequential process with an "undo," "cancel," or "collapse adjacent" behavior — you push forward progress and pop it back out when a cancellation condition is hit.

```java
public boolean backspaceCompare(String s, String t) {
    return buildFinalString(s).equals(buildFinalString(t));
}

private String buildFinalString(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : s.toCharArray()) {
        if (c == '#') {
            if (!stack.isEmpty()) stack.pop();
        } else {
            stack.push(c);
        }
    }
    StringBuilder sb = new StringBuilder();
    while (!stack.isEmpty()) sb.append(stack.pop());
    return sb.reverse().toString();
}
```

**Edge cases.**
- Popping from an empty stack — always guard with `!stack.isEmpty()` before popping; an empty backspace is a no-op, not an error.
- Multiple consecutive cancellations in a row need to chain correctly — simulate character by character rather than trying to shortcut with regex-style thinking.

**Problems:** Backspace String Compare, Remove All Adjacent Duplicates, Make the String Great, Minimum String Length After Removing Substrings.

---

## Pattern: Parenthesis & Scoring

**Trigger.** Validating bracket sequences, or computing a numeric score based on nested bracket depth.

```java
public boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    Map<Character, Character> pairs = Map.of(')', '(', ']', '[', '}', '{');

    for (char c : s.toCharArray()) {
        if (pairs.containsValue(c)) {
            stack.push(c);
        } else if (pairs.containsKey(c)) {
            if (stack.isEmpty() || stack.pop() != pairs.get(c)) return false;
        }
    }
    return stack.isEmpty();
}
```

**Score of Parentheses — depth-aware scoring:**

```java
public int scoreOfParentheses(String s) {
    Deque<Integer> stack = new ArrayDeque<>();
    stack.push(0);

    for (char c : s.toCharArray()) {
        if (c == '(') {
            stack.push(0);
        } else {
            int inner = stack.pop();
            int outer = stack.pop();
            stack.push(outer + Math.max(2 * inner, 1));
        }
    }
    return stack.pop();
}
```

**Edge cases.**
- Popping from an empty stack on a closing bracket means immediate invalidity — check before popping, don't let it throw.
- Non-empty stack after the full scan also means invalidity — an unmatched opening bracket is still unmatched even if nothing crashed.
- Longest Valid Parentheses is trickier than plain validation — track *indices* on the stack (not just characters), seeded with a base index of -1, so you can compute the length of valid spans directly.

**Problems:** Valid Parentheses, Minimum Add to Make Parentheses Valid, Score of Parentheses, Longest Valid Parentheses.

---

## Pattern: Stack-Based Design

**Trigger.** You're asked to build a data structure (queue from stacks, min-stack, max-stack) that needs O(1) or amortized O(1) operations, and the auxiliary structure choice affects whether that's achievable.

```java
public class MinStack {
    private final Deque<Long> stack = new ArrayDeque<>();
    private long min;

    public void push(int val) {
        if (stack.isEmpty()) {
            stack.push(0L);
            min = val;
        } else {
            stack.push((long) val - min); // store the offset from current min
            if (val < min) min = val;
        }
    }

    public void pop() {
        long top = stack.pop();
        if (top < 0) min = min - top; // this was the min; restore previous min
    }

    public int top() {
        long top = stack.peek();
        return (int) (top < 0 ? min : top + min);
    }

    public int getMin() {
        return (int) min;
    }
}
```

**Edge cases.**
- Long overflow when storing offsets against min — use `long`, not `int`, since the offset can exceed int range for adversarial inputs.
- Implementing a queue with two stacks — one stack absorbs pushes, the other serves pops; only transfer elements from the input stack to the output stack when the output stack is empty, not on every operation, or you lose the amortized O(1) guarantee.

**Problems:** Implement Queue using Stacks, Implement Stack using Queues, Min Stack, Max Stack, Design Stack with Increment Operation.

---

## Pattern: Stack + Greedy

**Trigger.** Building the smallest or largest possible string/number by removing characters, where each decision to keep or discard depends on what's already been committed and what's still ahead.

```java
public String removeKdigits(String num, int k) {
    Deque<Character> stack = new ArrayDeque<>();

    for (char c : num.toCharArray()) {
        while (!stack.isEmpty() && k > 0 && stack.peek() > c) {
            stack.pop();
            k--;
        }
        stack.push(c);
    }

    while (k > 0 && !stack.isEmpty()) { // still have removals left, trim from the end
        stack.pop();
        k--;
    }

    StringBuilder sb = new StringBuilder();
    while (!stack.isEmpty()) sb.append(stack.pop());
    sb.reverse();

    while (sb.length() > 1 && sb.charAt(0) == '0') sb.deleteCharAt(0); // strip leading zeros

    return sb.isEmpty() ? "0" : sb.toString();
}
```

**Edge cases.**
- Leading zeros in the result — strip them, but leave at least one digit if the whole number reduces to zero.
- k not fully exhausted by the main loop (the number was already non-decreasing) — must trim the remaining k digits from the end of the stack.
- Result becomes empty after all removals — return `"0"`, not an empty string.

**Problems:** Remove K Digits, Remove Duplicate Letters, Smallest Subsequence of Distinct Characters, Minimum Remove to Make Valid Parentheses, Create Maximum Number.

---

## Pattern: Recursive Stack

**Trigger.** You need to reach into the "bottom" of a stack (or the "tail" of a similarly LIFO-processed structure) without extra data structures, and unwinding the call stack itself becomes your auxiliary storage.

```java
public void insertAtBottom(Deque<Integer> stack, int val) {
    if (stack.isEmpty()) {
        stack.push(val);
        return;
    }
    int top = stack.pop();
    insertAtBottom(stack, val);
    stack.push(top);
}

public void reverseStack(Deque<Integer> stack) {
    if (stack.isEmpty()) return;
    int top = stack.pop();
    reverseStack(stack);
    insertAtBottom(stack, top);
}
```

**Edge cases.**
- Stack depth equals recursion depth — for very large stacks this risks a `StackOverflowError`; know this is a real limitation of the pattern, not just a theoretical one.
- Order of operations matters: pop-then-recurse-then-push-back is what lets you process from the bottom without a second data structure.

**Problems:** Delete Middle Element of Stack, Reverse a Stack (Recursive), Insert at Bottom of Stack.

---

# 5. Recursion

Solving problems by reducing them to smaller instances of the same problem. Every sub-pattern below is the same discipline applied at a different scale: define the base case first, trust the recursive call to solve the smaller version correctly, then combine.

## Pattern: Linear Recursion

**Trigger.** The problem reduces cleanly to "solve for n-1, then do one more step" — a single recursive call per level, no branching.

```java
public long factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}

public double myPow(double x, int n) {
    long exp = n; // widen to long to safely negate Integer.MIN_VALUE
    if (exp < 0) {
        x = 1 / x;
        exp = -exp;
    }
    return fastPow(x, exp);
}

private double fastPow(double x, long n) {
    if (n == 0) return 1.0;
    double half = fastPow(x, n / 2);
    return (n % 2 == 0) ? half * half : half * half * x;
}
```

**Edge cases.**
- `Integer.MIN_VALUE` negated overflows `int` — widen to `long` before negating, as shown above.
- Base case must be reachable — an off-by-one in the base case (`n <= 1` vs `n == 0`) is the most common bug in this pattern.
- Fast power (halving `n` each call) turns O(n) into O(log n) — always prefer this over a linear multiply loop when `n` can be large.

**Problems:** Factorial of N, Print 1 to N / N to 1, Check Palindrome (Recursive), Pow(x, n).

---

## Pattern: Non-Linear Recursion

**Trigger.** Multiple recursive calls branch out from each step (typically two), and you combine their results — classic tree-shaped recursion, often re-expressible as DP once you notice overlapping subproblems.

```java
public int climbStairs(int n) {
    return climb(n, new HashMap<>());
}

private int climb(int n, Map<Integer, Integer> memo) {
    if (n <= 2) return n;
    if (memo.containsKey(n)) return memo.get(n);
    int result = climb(n - 1, memo) + climb(n - 2, memo);
    memo.put(n, result);
    return result;
}
```

**Edge cases.**
- Without memoization, naive non-linear recursion is exponential (O(2^n)) — always ask whether overlapping subproblems exist, and memoize if they do.
- Recursion depth for large n can overflow the call stack even with memoization — for n in the tens of thousands, an iterative bottom-up version is safer.

**Problems:** Fibonacci Number, Climbing Stairs, Unique Paths, House Robber / Stickler Thief.

---

## Pattern: Divide & Conquer

**Trigger.** The problem splits cleanly into independent halves whose solutions combine in linear (or better) time — sorting, and problems that ask for something across two separately-processed collections.

```java
public void mergeSort(int[] arr, int left, int right) {
    if (left >= right) return;
    int mid = left + (right - left) / 2;
    mergeSort(arr, left, mid);
    mergeSort(arr, mid + 1, right);
    merge(arr, left, mid, right);
}

private void merge(int[] arr, int left, int mid, int right) {
    int[] temp = new int[right - left + 1];
    int i = left, j = mid + 1, k = 0;

    while (i <= mid && j <= right) {
        temp[k++] = (arr[i] <= arr[j]) ? arr[i++] : arr[j++];
    }
    while (i <= mid) temp[k++] = arr[i++];
    while (j <= right) temp[k++] = arr[j++];

    System.arraycopy(temp, 0, arr, left, temp.length);
}
```

**Median of Two Sorted Arrays — the hard-tier divide and conquer, via binary search on the partition point:**

```java
public double findMedianSortedArrays(int[] nums1, int[] nums2) {
    if (nums1.length > nums2.length) return findMedianSortedArrays(nums2, nums1);

    int m = nums1.length, n = nums2.length;
    int low = 0, high = m;

    while (low <= high) {
        int partition1 = low + (high - low) / 2;
        int partition2 = (m + n + 1) / 2 - partition1;

        int maxLeft1 = (partition1 == 0) ? Integer.MIN_VALUE : nums1[partition1 - 1];
        int minRight1 = (partition1 == m) ? Integer.MAX_VALUE : nums1[partition1];
        int maxLeft2 = (partition2 == 0) ? Integer.MIN_VALUE : nums2[partition2 - 1];
        int minRight2 = (partition2 == n) ? Integer.MAX_VALUE : nums2[partition2];

        if (maxLeft1 <= minRight2 && maxLeft2 <= minRight1) {
            if ((m + n) % 2 == 0) {
                return (Math.max(maxLeft1, maxLeft2) + Math.min(minRight1, minRight2)) / 2.0;
            }
            return Math.max(maxLeft1, maxLeft2);
        } else if (maxLeft1 > minRight2) {
            high = partition1 - 1;
        } else {
            low = partition1 + 1;
        }
    }
    throw new IllegalArgumentException("Input arrays are not sorted");
}
```

**Edge cases.**
- Merge sort needs a temp array per merge call, or an equivalently careful in-place merge — sloppy in-place merging is a common source of subtle bugs.
- Median of Two Sorted Arrays: always binary search on the *smaller* array to keep the partition bounds valid, and use `Integer.MIN_VALUE`/`MAX_VALUE` sentinels for out-of-range partition edges.
- Quick sort's worst case is O(n^2) on already-sorted or adversarial input unless you randomize the pivot — mention this if asked about complexity.

**Problems:** Binary Search (Recursive), Merge Sort, Quick Sort, Power (x^n), Median of Two Sorted Arrays.

---

## Pattern: Recursion on LinkedList/Stack

**Trigger.** Processing a linked list or stack by handling the head/top element and recursing on "the rest," letting the call stack implicitly reverse or defer processing order.

```java
public ListNode reverseList(ListNode head) {
    if (head == null || head.next == null) return head;
    ListNode newHead = reverseList(head.next);
    head.next.next = head;
    head.next = null;
    return newHead;
}

public ListNode mergeTwoLists(ListNode list1, ListNode list2) {
    if (list1 == null) return list2;
    if (list2 == null) return list1;

    if (list1.val <= list2.val) {
        list1.next = mergeTwoLists(list1.next, list2);
        return list1;
    } else {
        list2.next = mergeTwoLists(list1, list2.next);
        return list2;
    }
}
```

**Edge cases.**
- Recursive linked list solutions use O(n) call stack space even though they look elegant — for very long lists, prefer the iterative version if space matters.
- Forgetting to set `head.next = null` in recursive reversal creates a cycle — this is the single most common bug in recursive list reversal.

**Problems:** Reverse Linked List, Merge 2 Sorted Lists, Delete Middle of Stack, Reverse Stack.

---

## Pattern: Subsequences

**Trigger.** "Generate all subsets" or "explore all include/exclude choices for each element" — the include/exclude recursion tree, distinct from Backtracking mainly in that you're usually just enumerating rather than pruning against a complex constraint.

```java
public List<List<Integer>> generateSubsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    buildSubsets(0, nums, new ArrayList<>(), result);
    return result;
}

private void buildSubsets(int index, int[] nums, List<Integer> current, List<List<Integer>> result) {
    if (index == nums.length) {
        result.add(new ArrayList<>(current));
        return;
    }
    buildSubsets(index + 1, nums, current, result); // exclude nums[index]
    current.add(nums[index]);
    buildSubsets(index + 1, nums, current, result); // include nums[index]
    current.remove(current.size() - 1);
}
```

**Edge cases.**
- 2^n total subsets — this pattern is only viable when n is small (typically n <= 20), otherwise it won't finish in time.
- Always clone the current list before adding to results — storing a live reference means every future mutation corrupts already-stored answers.

**Problems:** Generate All Subsets, Subset Sum, Count Subsequences with Given Sum.

---

# 6. Linked List

Linear structure without contiguous memory. The absence of random access is the whole story here — every pattern exists to compensate for the fact that you can't jump to index i in O(1).

## Pattern: Basic Operations

**Trigger.** Direct pointer manipulation — insert, delete, search, or compute length by walking the list once.

```java
public class ListNode {
    int val;
    ListNode next;
    ListNode(int val) { this.val = val; }
}

public ListNode deleteNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode fast = dummy, slow = dummy;

    for (int i = 0; i < n; i++) fast = fast.next;

    while (fast.next != null) {
        fast = fast.next;
        slow = slow.next;
    }
    slow.next = slow.next.next;
    return dummy.next;
}
```

**Edge cases.**
- Deleting the head node — always use a dummy node pointing to head so head-deletion doesn't need special-case code.
- Single-node list — verify your logic doesn't dereference a null `next` when the list has only one element.
- Off-by-one when counting "nth from end" — walking `fast` exactly `n` steps ahead before starting the second pointer is what makes the gap correct.

**Problems:** Search in Linked List, Insert at Head/Tail/Nth Position, Delete Head/Tail/Nth Node, Intersection of Two Linked Lists, Design Linked List, Odd-Even Linked List.

---

## Pattern: Fast and Slow Pointers

**Trigger.** Cycle detection, finding the middle node, or anything needing O(1) extra space on a single-direction list.

```java
public boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}

public ListNode detectCycleStart(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) {
            ListNode ptr = head;
            while (ptr != slow) {
                ptr = ptr.next;
                slow = slow.next;
            }
            return ptr;
        }
    }
    return null;
}
```

**Edge cases.**
- Always check both `fast != null && fast.next != null` — checking only one leaves a null-pointer gap when the list has an even number of nodes.
- Finding the cycle *start* (not just detecting a cycle exists) requires the second phase shown above — resetting one pointer to head and advancing both one step at a time. This works because of the mathematical relationship between the distance to the cycle start and the cycle length; know the "why" if asked to explain it, not just the code.

**Problems:** Middle of the Linked List, Linked List Cycle, Linked List Cycle II, Remove Nth Node From End.

---

## Pattern: Reversal Pattern

**Trigger.** Reversing the whole list, a sub-segment, or fixed-size groups within the list.

```java
public ListNode reverseList(ListNode head) {
    ListNode prev = null;
    ListNode current = head;
    while (current != null) {
        ListNode next = current.next;
        current.next = prev;
        prev = current;
        current = next;
    }
    return prev;
}

public ListNode reverseKGroup(ListNode head, int k) {
    ListNode node = head;
    int count = 0;
    while (node != null && count < k) {
        node = node.next;
        count++;
    }
    if (count < k) return head; // fewer than k nodes remain, leave as-is

    ListNode newHead = reverseKGroup(node, k); // recurse on the rest first
    ListNode prev = newHead;
    ListNode current = head;
    for (int i = 0; i < k; i++) {
        ListNode next = current.next;
        current.next = prev;
        prev = current;
        current = next;
    }
    return prev;
}
```

**Edge cases.**
- Reversing a sub-segment (between positions m and n) requires caching the node just before position m and the node at position n+1 before touching any links, or you'll lose the rest of the list permanently.
- Reverse in k-groups: if fewer than k nodes remain at the end, leave that final partial group unreversed — check the count before committing to reverse.
- Always keep a `dummy` node before head when the head itself might change, so you have a stable reference to return.

**Problems:** Reverse a Linked List, Palindrome Linked List, Reverse Linked List II, Maximum Twin Sum, Swap Nodes in Pairs, Rotate List, Reverse Nodes in k-Group.

---

## Pattern: Merge / Sort

**Trigger.** Combining sorted lists, sorting a list in O(n log n), or reordering using a combination of middle-finding, reversal, and merging.

```java
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0);
    ListNode tail = dummy;

    while (l1 != null && l2 != null) {
        if (l1.val <= l2.val) {
            tail.next = l1;
            l1 = l1.next;
        } else {
            tail.next = l2;
            l2 = l2.next;
        }
        tail = tail.next;
    }
    tail.next = (l1 != null) ? l1 : l2;
    return dummy.next;
}

public ListNode sortList(ListNode head) {
    if (head == null || head.next == null) return head;

    ListNode slow = head, fast = head, prev = null;
    while (fast != null && fast.next != null) {
        prev = slow;
        slow = slow.next;
        fast = fast.next.next;
    }
    prev.next = null; // split into two halves

    ListNode left = sortList(head);
    ListNode right = sortList(slow);
    return mergeTwoLists(left, right);
}
```

**Edge cases.**
- Splitting the list in half needs the `prev.next = null` cut — forgetting this creates an infinite loop in the recursive call.
- Merge K Sorted Lists should use a min-heap of list heads (O(N log k)), not repeated pairwise merging (O(Nk)) — pairwise merging technically works but is asymptotically worse and often too slow for the given constraints.

**Problems:** Merge Two Sorted Lists, Remove Duplicates from Sorted List, Sort List, Reorder List, Remove Duplicates from Sorted List II, Partition List, Merge K Sorted Lists.

---

## Pattern: LinkedList with Stack/HashMap

**Trigger.** Backward-dependent processing on a forward-only list — carrying digits, finding the next greater node, or needing O(1) lookup of arbitrary nodes (random pointer copying).

```java
public Node copyRandomList(Node head) {
    if (head == null) return null;
    Map<Node, Node> oldToNew = new HashMap<>();

    Node current = head;
    while (current != null) {
        oldToNew.put(current, new Node(current.val));
        current = current.next;
    }

    current = head;
    while (current != null) {
        oldToNew.get(current).next = oldToNew.get(current.next);
        oldToNew.get(current).random = oldToNew.get(current.random);
        current = current.next;
    }
    return oldToNew.get(head);
}
```

**Edge cases.**
- `random` pointer can point to `null` or to a node later in the list — `HashMap.get(null)` correctly returns `null`, so no extra check is needed there, but verify your map is fully populated before the second pass.
- Add Two Numbers: lists represent digits in *reverse* order in the classic version — don't reverse them again by mistake; carry propagation happens naturally left to right through the existing order.

**Problems:** Add Two Numbers, Add Two Numbers II, Next Greater Node in Linked List, Remove Nodes From Linked List, Copy List with Random Pointer.

---

# 7. Double Linked List

Navigation in both directions. The extra `prev` pointer trades memory for O(1) removal of an arbitrary known node — that trade is the entire reason this topic exists as distinct from singly linked lists.

## Pattern: Basic DLL Operations

**Trigger.** You need O(1) insert/delete at both ends, or an LRU/LFU-style cache where recency ordering must update in O(1) per access.

```java
public class LRUCache {
    private final int capacity;
    private final Map<Integer, Node> cache = new HashMap<>();
    private final Node head = new Node(0, 0);
    private final Node tail = new Node(0, 0);

    private static class Node {
        int key, val;
        Node prev, next;
        Node(int key, int val) { this.key = key; this.val = val; }
    }

    public LRUCache(int capacity) {
        this.capacity = capacity;
        head.next = tail;
        tail.prev = head;
    }

    public int get(int key) {
        if (!cache.containsKey(key)) return -1;
        Node node = cache.get(key);
        remove(node);
        insertAtFront(node);
        return node.val;
    }

    public void put(int key, int value) {
        if (cache.containsKey(key)) {
            remove(cache.get(key));
        }
        Node node = new Node(key, value);
        cache.put(key, node);
        insertAtFront(node);

        if (cache.size() > capacity) {
            Node lru = tail.prev;
            remove(lru);
            cache.remove(lru.key);
        }
    }

    private void remove(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private void insertAtFront(Node node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }
}
```

**Edge cases.**
- Sentinel `head`/`tail` nodes eliminate null checks at the boundaries — without them, inserting or removing at the very front or back needs special-case branches, which is where bugs live.
- `put` on an existing key must update its value *and* move it to the front — a common mistake is updating the value but forgetting to touch its position.
- LFU Cache needs a second layer of bookkeeping (frequency buckets, each itself a DLL) — it's a genuinely harder design problem than LRU, budget more time for it.

**Problems:** Implement Doubly Linked List, Insert/Delete a Node in a DLL, Reverse Doubly Linked List, LRU Cache, LFU Cache.

---

## Pattern: Merge / Sort / Reorder

**Trigger.** Using the `prev`/`next` structure of a DLL to merge, flatten, or convert between DLL and another structure (like a balanced BST).

```java
public Node flatten(Node head) {
    if (head == null) return head;

    Node current = head;
    while (current != null) {
        if (current.child != null) {
            Node next = current.next;
            Node childHead = flatten(current.child);

            current.next = childHead;
            childHead.prev = current;
            current.child = null;

            Node tail = childHead;
            while (tail.next != null) tail = tail.next;

            tail.next = next;
            if (next != null) next.prev = tail;
        }
        current = current.next;
    }
    return head;
}
```

**Edge cases.**
- Flattening a multilevel DLL: process depth-first, and remember to null out the `child` pointer after flattening it, or the structure remains ambiguous.
- Converting DLL to a balanced BST needs the middle node as root at each level, same as sorted-array-to-BST, but walked via slow/fast pointers since there's no direct indexing.

**Problems:** Merge Two Sorted DLLs, Flatten Multilevel DLL, Convert DLL to Binary Tree.

---

# 8. HashMap

Key-value structure with O(1) average lookup. The pattern here is less about the map itself and more about *what you choose to use as the key* — that choice is usually the entire solution.

## Pattern: Frequency Map / Counting

**Trigger.** Counting occurrences to find a majority element, the top-k frequent items, or grouping by frequency.

```java
public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int num : nums) freq.merge(num, 1, Integer::sum);

    List<Integer>[] buckets = new List[nums.length + 1]; // bucket sort by frequency
    for (Map.Entry<Integer, Integer> entry : freq.entrySet()) {
        int count = entry.getValue();
        if (buckets[count] == null) buckets[count] = new ArrayList<>();
        buckets[count].add(entry.getKey());
    }

    int[] result = new int[k];
    int idx = 0;
    for (int count = buckets.length - 1; count >= 0 && idx < k; count--) {
        if (buckets[count] == null) continue;
        for (int num : buckets[count]) {
            if (idx == k) break;
            result[idx++] = num;
        }
    }
    return result;
}
```

**Edge cases.**
- Bucket sort by frequency gives O(n) instead of O(n log n) from a heap or full sort — worth knowing both approaches, but default to bucket sort when k and n make the difference meaningful.
- Majority Element (appearing more than n/2 times) has an O(1)-space solution via Boyer-Moore voting — don't reach for a HashMap there if the interviewer is probing for the optimal-space answer.

**Problems:** Majority Element, Top K Frequent Elements, Sort Characters By Frequency, Task Scheduler.

---

## Pattern: Prefix-Sum with Map

**Trigger.** Subarray sum problems where two-pointer fails because of negative numbers — the map stores the *first occurrence* of a prefix sum, letting you detect a valid subarray in O(1) per position.

```java
public int subarraySum(int[] nums, int k) {
    Map<Integer, Integer> prefixCount = new HashMap<>();
    prefixCount.put(0, 1); // empty prefix, needed for subarrays starting at index 0

    int count = 0, sum = 0;
    for (int num : nums) {
        sum += num;
        count += prefixCount.getOrDefault(sum - k, 0);
        prefixCount.merge(sum, 1, Integer::sum);
    }
    return count;
}
```

**Edge cases.**
- Forgetting to seed the map with `(0, 1)` silently drops every valid subarray that starts at index 0 — this is the single most common bug in this pattern, worth memorizing as a checklist item.
- This counts *number of subarrays*, not their positions — if the problem wants indices, you need to store lists of indices per prefix sum instead of just counts.

**Problems:** Count Subarrays with Sum K.

---

# 9. Tree

Hierarchical structure. The organizing question for this whole topic is: does the answer at a node depend on information from its children (bottom-up / postorder-flavored), or does it depend on information passed down from its ancestors (top-down)? Get that right first, and the traversal choice follows naturally.

## Pattern: DFS Traversals

**Trigger.** Standard depth-first recursion — used for max depth, path sums, subtree comparisons, and diameter-style problems where children need to report information up to their parent.

```java
public class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int val) { this.val = val; }
}

public int maxDepth(TreeNode root) {
    if (root == null) return 0;
    return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}

public int diameterOfBinaryTree(TreeNode root) {
    int[] diameter = new int[1];
    height(root, diameter);
    return diameter[0];
}

private int height(TreeNode node, int[] diameter) {
    if (node == null) return 0;
    int leftHeight = height(node.left, diameter);
    int rightHeight = height(node.right, diameter);
    diameter[0] = Math.max(diameter[0], leftHeight + rightHeight);
    return 1 + Math.max(leftHeight, rightHeight);
}

public int maxPathSum(TreeNode root) {
    int[] maxSum = { Integer.MIN_VALUE };
    maxGain(root, maxSum);
    return maxSum[0];
}

private int maxGain(TreeNode node, int[] maxSum) {
    if (node == null) return 0;
    int leftGain = Math.max(maxGain(node.left, maxSum), 0);  // ignore negative contributions
    int rightGain = Math.max(maxGain(node.right, maxSum), 0);
    maxSum[0] = Math.max(maxSum[0], node.val + leftGain + rightGain);
    return node.val + Math.max(leftGain, rightGain); // can only extend one side upward
}
```

**Edge cases.**
- Empty tree (`root == null`) — always the first base case; most bugs come from forgetting this returns something sensible (0, true, empty list) rather than crashing.
- Binary Tree Maximum Path Sum: negative node values need `Math.max(gain, 0)` to avoid a negative branch dragging the total down — and the value returned up to the parent is *not* the same as the value used to update the global max, since a path passing through a node can use both children, but a path extending to the parent can only use one.
- Deeply skewed (effectively linked-list-shaped) trees can overflow the recursion stack for very large n — know this limitation even if you don't rewrite the solution iteratively.

**Problems:** Inorder/Preorder/Postorder Traversal, Same Tree, Symmetric Tree, Diameter of Binary Tree, Balanced Binary Tree, Maximum/Minimum Depth, Subtree of Another Tree, Path Sum I/II/III, Print All Nodes at Distance K, Boundary Traversal, Count Complete Tree Nodes, Binary Tree Maximum Path Sum, Binary Tree Cameras.

---

## Pattern: BFS / Level-Order

**Trigger.** Anything level-by-level — level sums or averages, zigzag ordering, side views, or connecting nodes across the same depth.

```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;

    Queue<TreeNode> queue = new ArrayDeque<>();
    queue.offer(root);

    while (!queue.isEmpty()) {
        int levelSize = queue.size(); // capture before mutating the queue
        List<Integer> level = new ArrayList<>(levelSize);

        for (int i = 0; i < levelSize; i++) {
            TreeNode node = queue.poll();
            level.add(node.val);
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        result.add(level);
    }
    return result;
}

public List<Integer> rightSideView(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    if (root == null) return result;

    Queue<TreeNode> queue = new ArrayDeque<>();
    queue.offer(root);

    while (!queue.isEmpty()) {
        int levelSize = queue.size();
        for (int i = 0; i < levelSize; i++) {
            TreeNode node = queue.poll();
            if (i == levelSize - 1) result.add(node.val); // last node seen at this level
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
    }
    return result;
}
```

**Edge cases.**
- Capture `queue.size()` into a local variable at the top of each level's loop — checking `queue.size()` mid-loop gives the wrong boundary since you're actively adding children to the same queue.
- Vertical Order Traversal needs a `(column, row, value)` triple sorted by column then row then value — plain BFS level order alone is not enough; you need coordinate tracking alongside the queue.
- Zigzag traversal is level order with alternating reversal — don't rebuild the whole traversal logic, just flip a boolean each level and reverse that level's list before adding it.

**Problems:** Binary Tree Level Order Traversal, Zigzag Level Order, Average of Levels, Cousins in Binary Tree, Left/Right Side View, Populating Next Right Pointers, Vertical Order Traversal, Top View, Bottom View, Maximum Width of Binary Tree.

---

## Pattern: Lowest Common Ancestor

**Trigger.** Finding the deepest node that is an ancestor of two given nodes — DFS recursion returning "found" signals up, or parent-pointer mapping when repeated LCA queries are needed.

```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null || root == p || root == q) return root;

    TreeNode left = lowestCommonAncestor(root.left, p, q);
    TreeNode right = lowestCommonAncestor(root.right, p, q);

    if (left != null && right != null) return root; // p and q found in different subtrees
    return (left != null) ? left : right;
}
```

**Edge cases.**
- Assumes both `p` and `q` exist in the tree — if that's not guaranteed, you need an extra pass to verify both were actually found, not just trust the recursion's return value.
- Kth Ancestor of a Tree Node with many repeated queries benefits from binary lifting (precomputing 2^i-th ancestors) rather than walking up one step at a time per query — mention this if the problem specifies many queries.

**Problems:** Lowest Common Ancestor of Binary Tree, Find Distance Between Nodes, Kth Ancestor of a Tree Node.

---

## Pattern: Serialization / Construction

**Trigger.** Encoding a tree to a string and decoding it back, or reconstructing a tree from traversal arrays (preorder + inorder, etc.).

```java
public String serialize(TreeNode root) {
    StringBuilder sb = new StringBuilder();
    serializeHelper(root, sb);
    return sb.toString();
}

private void serializeHelper(TreeNode node, StringBuilder sb) {
    if (node == null) {
        sb.append("#,");
        return;
    }
    sb.append(node.val).append(",");
    serializeHelper(node.left, sb);
    serializeHelper(node.right, sb);
}

public TreeNode deserialize(String data) {
    Deque<String> tokens = new ArrayDeque<>(Arrays.asList(data.split(",")));
    return deserializeHelper(tokens);
}

private TreeNode deserializeHelper(Deque<String> tokens) {
    String val = tokens.poll();
    if (val.equals("#")) return null;
    TreeNode node = new TreeNode(Integer.parseInt(val));
    node.left = deserializeHelper(tokens);
    node.right = deserializeHelper(tokens);
    return node;
}

public TreeNode buildTree(int[] preorder, int[] inorder) {
    Map<Integer, Integer> inorderIndex = new HashMap<>();
    for (int i = 0; i < inorder.length; i++) inorderIndex.put(inorder[i], i);
    return build(preorder, 0, preorder.length - 1, inorder, 0, inorder.length - 1, inorderIndex, new int[]{0});
}

private TreeNode build(int[] preorder, int preStart, int preEnd, int[] inorder, int inStart, int inEnd,
                        Map<Integer, Integer> inorderIndex, int[] preIndex) {
    if (preStart > preEnd) return null;

    int rootVal = preorder[preIndex[0]++];
    TreeNode root = new TreeNode(rootVal);
    int inRoot = inorderIndex.get(rootVal);
    int leftSize = inRoot - inStart;

    root.left = build(preorder, preStart + 1, preStart + leftSize, inorder, inStart, inRoot - 1, inorderIndex, preIndex);
    root.right = build(preorder, preStart + leftSize + 1, preEnd, inorder, inRoot + 1, inEnd, inorderIndex, preIndex);
    return root;
}
```

**Edge cases.**
- Preorder+Inorder reconstruction requires *distinct* element values, or the inorder-index lookup becomes ambiguous — check the problem's constraints on duplicates before assuming this works.
- Serialization must explicitly encode null children (the `#` sentinel above) — without that, you can't distinguish a tree's actual shape during deserialization.
- Precomputing the inorder value-to-index map avoids an O(n) linear scan per recursive call, dropping the whole build from O(n²) to O(n).

**Problems:** Invert Binary Tree, Flatten Binary Tree to Linked List, Construct Binary Tree from Preorder & Inorder, Construct from Inorder & Postorder, Construct from Preorder & Postorder, Serialize and Deserialize Binary Tree.

---

# 10. Binary Search Tree

Left child < root < right child. Every pattern here exploits that ordering to prune half the tree at each step, exactly like binary search on an array — except the "array" is implicit in the tree's shape.

## Pattern: BST Operations

**Trigger.** Search, insert, delete, or range queries where the BST ordering property lets you avoid visiting both children at every node.

```java
public boolean isValidBST(TreeNode root) {
    return validate(root, null, null);
}

private boolean validate(TreeNode node, Long lower, Long upper) {
    if (node == null) return true;
    if (lower != null && node.val <= lower) return false;
    if (upper != null && node.val >= upper) return false;
    return validate(node.left, lower, (long) node.val) && validate(node.right, (long) node.val, upper);
}

public TreeNode deleteNode(TreeNode root, int key) {
    if (root == null) return null;

    if (key < root.val) {
        root.left = deleteNode(root.left, key);
    } else if (key > root.val) {
        root.right = deleteNode(root.right, key);
    } else {
        if (root.left == null) return root.right;
        if (root.right == null) return root.left;

        TreeNode successor = root.right;
        while (successor.left != null) successor = successor.left; // smallest in right subtree
        root.val = successor.val;
        root.right = deleteNode(root.right, successor.val);
    }
    return root;
}

public int kthSmallest(TreeNode root, int k) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode current = root;

    while (current != null || !stack.isEmpty()) {
        while (current != null) {
            stack.push(current);
            current = current.left;
        }
        current = stack.pop();
        if (--k == 0) return current.val;
        current = current.right;
    }
    throw new IllegalArgumentException("k is out of bounds");
}
```

**Edge cases.**
- BST validation with `Integer.MIN_VALUE`/`MAX_VALUE` node values breaks int-based bound checks — use `Long` bounds (nullable, as shown) instead.
- Deletion with two children: replace with either the inorder successor (smallest in right subtree) or inorder predecessor (largest in left subtree) — either is correct, just be consistent.
- Kth Smallest via iterative inorder traversal avoids building the full sorted list in memory — prefer it over "inorder into an ArrayList then index" for large trees.

**Problems:** Convert Sorted Array to BST, Search in a BST, Two Sum IV, Insert into a BST, Validate BST, Delete Node in a BST, Recover BST, Merge 2 BST, Maximum Sum BST in Binary Tree, Kth Smallest Element, Convert BST to Greater Tree, BST Iterator, Inorder Successor/Predecessor, Construct BST from Preorder, Convert Binary Tree to DLL.

---

## Pattern: LCA & Range Queries

**Trigger.** Using BST ordering specifically (not general tree DFS) to find the lowest common ancestor in O(h) by walking from the root toward the split point.

```java
public TreeNode lowestCommonAncestorBST(TreeNode root, TreeNode p, TreeNode q) {
    TreeNode current = root;
    while (current != null) {
        if (p.val < current.val && q.val < current.val) {
            current = current.left;
        } else if (p.val > current.val && q.val > current.val) {
            current = current.right;
        } else {
            return current; // split point found
        }
    }
    return null;
}
```

**Edge cases.**
- This is strictly faster than the general binary-tree LCA (O(h) vs O(n)) precisely because it uses BST ordering instead of searching both subtrees — don't fall back to the general-tree recursive LCA out of habit when the tree is explicitly a BST.
- Closest BST Value needs to track the running best answer while walking down, comparing absolute differences at each step, rather than doing a full traversal.

**Problems:** Closest Binary Search Tree Value, Lowest Common Ancestor of BST, Closest Leaf in BST.

---

# 11. Graph

Nodes and edges. The single most important decision at the start of every graph problem: is it directed or undirected, weighted or unweighted, and does it need shortest path, connectivity, or ordering? That classification points directly at which sub-pattern below applies.

## Pattern: BFS (Unweighted Path)

**Trigger.** Shortest path or minimum steps in a graph or grid where every edge has equal weight (weight 1). BFS guarantees the first time you reach a node is via the shortest path, which is exactly why it's preferred over DFS here.

```java
public int shortestPathBinaryMatrix(int[][] grid) {
    int n = grid.length;
    if (grid[0][0] != 0 || grid[n - 1][n - 1] != 0) return -1;

    Queue<int[]> queue = new ArrayDeque<>();
    queue.offer(new int[]{0, 0, 1});
    grid[0][0] = 1; // reuse grid as visited marker

    int[][] directions = {{-1,-1},{-1,0},{-1,1},{0,-1},{0,1},{1,-1},{1,0},{1,1}};

    while (!queue.isEmpty()) {
        int[] cell = queue.poll();
        if (cell[0] == n - 1 && cell[1] == n - 1) return cell[2];

        for (int[] dir : directions) {
            int r = cell[0] + dir[0], c = cell[1] + dir[1];
            if (r >= 0 && c >= 0 && r < n && c < n && grid[r][c] == 0) {
                grid[r][c] = 1;
                queue.offer(new int[]{r, c, cell[2] + 1});
            }
        }
    }
    return -1;
}
```

**Multi-source BFS (Rotting Oranges shape) — seed the queue with every source before the first step, not just one:**

```java
public int orangesRotting(int[][] grid) {
    int rows = grid.length, cols = grid[0].length;
    Queue<int[]> queue = new ArrayDeque<>();
    int freshCount = 0;

    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            if (grid[r][c] == 2) queue.offer(new int[]{r, c});
            if (grid[r][c] == 1) freshCount++;
        }
    }

    int minutes = 0;
    int[][] directions = {{1,0},{-1,0},{0,1},{0,-1}};

    while (!queue.isEmpty() && freshCount > 0) {
        int size = queue.size();
        for (int i = 0; i < size; i++) {
            int[] cell = queue.poll();
            for (int[] dir : directions) {
                int r = cell[0] + dir[0], c = cell[1] + dir[1];
                if (r >= 0 && c >= 0 && r < rows && c < cols && grid[r][c] == 1) {
                    grid[r][c] = 2;
                    freshCount--;
                    queue.offer(new int[]{r, c});
                }
            }
        }
        minutes++;
    }
    return freshCount == 0 ? minutes : -1;
}
```

**Edge cases.**
- Mark nodes visited at the moment you enqueue them, not when you dequeue — delaying causes duplicate enqueues and wrong distance tracking.
- Multi-source BFS needs *all* sources seeded into the queue before the level loop starts — seeding one at a time and running separate BFS passes is both slower and can give wrong results if sources' shortest paths overlap.
- Word Ladder treats each word as a graph node with edges to words one character apart — the graph isn't given explicitly, you build adjacency implicitly by trying every character substitution per position.

**Problems:** 01 Matrix, Clone Graph, Rotting Oranges, Shortest Path in Binary Matrix, Escape the Spreading Fire, Word Ladder.

---

## Pattern: DFS (Connectivity)

**Trigger.** Connected components, cycle detection, bipartiteness checks, or structural properties (articulation points, bridges) — anywhere you need to fully explore reachability from a node before moving to the next unvisited one.

```java
public int numIslands(char[][] grid) {
    int count = 0;
    for (int r = 0; r < grid.length; r++) {
        for (int c = 0; c < grid[0].length; c++) {
            if (grid[r][c] == '1') {
                count++;
                sinkIsland(grid, r, c);
            }
        }
    }
    return count;
}

private void sinkIsland(char[][] grid, int r, int c) {
    if (r < 0 || c < 0 || r >= grid.length || c >= grid[0].length || grid[r][c] != '1') return;
    grid[r][c] = '0'; // mark visited by mutating in place
    sinkIsland(grid, r + 1, c);
    sinkIsland(grid, r - 1, c);
    sinkIsland(grid, r, c + 1);
    sinkIsland(grid, r, c - 1);
}

public boolean isBipartite(int[][] graph) {
    int n = graph.length;
    int[] colors = new int[n]; // 0 = uncolored, 1 or -1 = two colors

    for (int i = 0; i < n; i++) {
        if (colors[i] != 0) continue;
        colors[i] = 1;
        Queue<Integer> queue = new ArrayDeque<>();
        queue.offer(i);

        while (!queue.isEmpty()) {
            int node = queue.poll();
            for (int neighbor : graph[node]) {
                if (colors[neighbor] == 0) {
                    colors[neighbor] = -colors[node];
                    queue.offer(neighbor);
                } else if (colors[neighbor] == colors[node]) {
                    return false;
                }
            }
        }
    }
    return true;
}
```

**Directed cycle detection (needs a 3-color / recursion-stack scheme, not just a visited set):**

```java
public boolean hasCycleDirected(int n, List<List<Integer>> adj) {
    int[] state = new int[n]; // 0 = unvisited, 1 = in current path, 2 = fully processed

    for (int i = 0; i < n; i++) {
        if (state[i] == 0 && dfs(i, adj, state)) return true;
    }
    return false;
}

private boolean dfs(int node, List<List<Integer>> adj, int[] state) {
    state[node] = 1;
    for (int neighbor : adj.get(node)) {
        if (state[neighbor] == 1) return true; // back edge to a node in the current path = cycle
        if (state[neighbor] == 0 && dfs(neighbor, adj, state)) return true;
    }
    state[node] = 2;
    return false;
}
```

**Edge cases.**
- Undirected-graph cycle detection only needs a simple visited set plus tracking the parent (to avoid treating the edge back to your immediate parent as a false cycle) — directed graphs need the three-state scheme above, because a back edge to any *ancestor* (not just the immediate parent) signals a cycle, and a visited-but-not-in-current-path node is fine.
- Mutating the grid in place (marking visited by changing the value) saves a separate `visited` array, but only do this if the problem doesn't need the original grid preserved afterward.
- Articulation points and bridges need discovery time and low-link values (Tarjan's algorithm) — this is one of the more involved DFS variants; budget extra time and expect to look up the exact low-link update rule until it's memorized.

**Problems:** Flood Fill, Number of Islands, All Paths from Source to Target, Find Eventual Safe States, Count Components in Graph, Surrounded Regions, Is Graph Bipartite, Directed/Undirected Cycle Detection, Longest Cycle in a Graph, Articulation Points, Bridges in Graph.

---

## Pattern: Topological Sort

**Trigger.** Task scheduling with prerequisites, course ordering, or any directed acyclic graph where you need an ordering that respects all dependency edges.

```java
public int[] findOrder(int numCourses, int[][] prerequisites) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());

    int[] inDegree = new int[numCourses];
    for (int[] pre : prerequisites) {
        adj.get(pre[1]).add(pre[0]);
        inDegree[pre[0]]++;
    }

    Queue<Integer> queue = new ArrayDeque<>();
    for (int i = 0; i < numCourses; i++) {
        if (inDegree[i] == 0) queue.offer(i);
    }

    int[] order = new int[numCourses];
    int idx = 0;

    while (!queue.isEmpty()) {
        int node = queue.poll();
        order[idx++] = node;
        for (int neighbor : adj.get(node)) {
            if (--inDegree[neighbor] == 0) queue.offer(neighbor);
        }
    }
    return idx == numCourses ? order : new int[0]; // shorter output means a cycle exists
}
```

**Edge cases.**
- If the resulting order is shorter than the total node count, the graph contains a cycle and no valid topological order exists — this is your cycle-detection signal for free.
- Alien Dictionary: build edges by comparing adjacent words character-by-character at their first point of difference — and if one word is a prefix of the next but appears *after* it in the list, that's an invalid ordering to detect explicitly, not something topological sort catches automatically.

**Problems:** Task Scheduling with Dependencies, Course Schedule I & II, Find Eventual Safe States, Cycle Detection in Directed Graph, Alien Dictionary, Reconstruct Itinerary.

---

## Pattern: MST / Union-Find

**Trigger.** Minimum spanning tree, minimum cost to connect all nodes, or dynamic connectivity queries where edges are added incrementally and you need to check or merge components efficiently.

```java
public class DSU {
    private final int[] parent;
    private final int[] rank;

    public DSU(int n) {
        parent = new int[n];
        rank = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;
    }

    public int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]); // path compression
        return parent[x];
    }

    public boolean union(int x, int y) {
        int rootX = find(x), rootY = find(y);
        if (rootX == rootY) return false; // already connected — this edge would form a cycle

        if (rank[rootX] < rank[rootY]) {
            parent[rootX] = rootY;
        } else if (rank[rootX] > rank[rootY]) {
            parent[rootY] = rootX;
        } else {
            parent[rootY] = rootX;
            rank[rootX]++;
        }
        return true;
    }
}

public int minCostConnectPoints(int[][] points) { // Kruskal's MST
    int n = points.length;
    List<int[]> edges = new ArrayList<>();

    for (int i = 0; i < n; i++) {
        for (int j = i + 1; j < n; j++) {
            int dist = Math.abs(points[i][0] - points[j][0]) + Math.abs(points[i][1] - points[j][1]);
            edges.add(new int[]{dist, i, j});
        }
    }
    edges.sort((a, b) -> a[0] - b[0]);

    DSU dsu = new DSU(n);
    int totalCost = 0, edgesUsed = 0;

    for (int[] edge : edges) {
        if (dsu.union(edge[1], edge[2])) {
            totalCost += edge[0];
            edgesUsed++;
            if (edgesUsed == n - 1) break;
        }
    }
    return totalCost;
}
```

**Edge cases.**
- Skipping path compression turns `find` into a slow linear walk over time — always compress the path, as shown.
- Kruskal's needs edges sorted ascending by weight first — MST correctness depends entirely on greedily taking the cheapest edge that doesn't form a cycle, so an unsorted edge list breaks the algorithm.
- Redundant Connection: the answer is the *first* edge in input order that Union-Find rejects (i.e., the first edge connecting two already-connected nodes) — don't just find any cycle-forming edge, it has to be that specific one per the problem's tie-breaking rule.

**Problems:** Minimum Spanning Tree, Kruskal's Algorithm, Lexicographically Smallest Equivalent String, Number of Connected Components, Redundant Connection, Connecting Cities With Minimum Cost, Accounts Merge.

---

## Pattern: Dijkstra (Weighted)

**Trigger.** Shortest path in a graph with non-negative edge weights — priority queue relaxation, guaranteed correct as long as no edge weight is negative.

```java
public int networkDelayTime(int[][] times, int n, int k) {
    List<List<int[]>> adj = new ArrayList<>();
    for (int i = 0; i <= n; i++) adj.add(new ArrayList<>());
    for (int[] time : times) adj.get(time[0]).add(new int[]{time[1], time[2]});

    int[] dist = new int[n + 1];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[k] = 0;

    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[1] - b[1]);
    pq.offer(new int[]{k, 0});

    while (!pq.isEmpty()) {
        int[] current = pq.poll();
        int node = current[0], d = current[1];
        if (d > dist[node]) continue; // stale entry, already found a better path

        for (int[] edge : adj.get(node)) {
            int neighbor = edge[0], weight = edge[1];
            if (dist[node] + weight < dist[neighbor]) {
                dist[neighbor] = dist[node] + weight;
                pq.offer(new int[]{neighbor, dist[neighbor]});
            }
        }
    }

    int maxDist = Arrays.stream(dist, 1, n + 1).max().getAsInt();
    return maxDist == Integer.MAX_VALUE ? -1 : maxDist;
}
```

**Edge cases.**
- Negative edge weights break Dijkstra's correctness guarantee entirely — switch to Bellman-Ford if any edge can be negative.
- Skip stale priority-queue entries (where the popped distance no longer matches the recorded best distance) — without this check you still get a correct answer, but performance degrades badly on dense graphs.
- Cheapest Flights Within K Stops needs the state to include *stops used so far*, not just node and distance — plain Dijkstra will greedily lock in the shortest path first even if it uses too many stops, so track `(node, stopsUsed)` as the state instead of just `node`.

**Problems:** Dijkstra Implementation, Shortest Path in Weighted Graph, Minimum Cost Path in Grid, Network Delay Time, Cheapest Flights Within K Stops, Swim in Rising Water, Path With Minimum Effort.

---

## Pattern: Bellman-Ford

**Trigger.** Shortest path where negative edge weights are possible, or you explicitly need to detect a negative cycle.

```java
public int[] bellmanFord(int n, int[][] edges, int src) {
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;

    for (int i = 0; i < n - 1; i++) { // relax all edges n-1 times
        for (int[] edge : edges) {
            int u = edge[0], v = edge[1], w = edge[2];
            if (dist[u] != Integer.MAX_VALUE && dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
            }
        }
    }

    for (int[] edge : edges) { // one more pass to detect a negative cycle
        int u = edge[0], v = edge[1], w = edge[2];
        if (dist[u] != Integer.MAX_VALUE && dist[u] + w < dist[v]) {
            throw new IllegalStateException("Graph contains a negative weight cycle");
        }
    }
    return dist;
}
```

**Edge cases.**
- Exactly `n - 1` relaxation passes are needed because the longest possible shortest path in a graph with `n` nodes uses at most `n - 1` edges — fewer passes can miss valid relaxations, more passes are just wasted work (except the extra cycle-check pass).
- The `n`-th pass finding a further relaxation is precisely the negative-cycle signal — don't skip this check if the problem asks about cycle detection.
- Always check `dist[u] != Integer.MAX_VALUE` before adding an edge weight to it — adding to `MAX_VALUE` overflows and produces a nonsensical negative number.

**Problems:** Negative Weight Cycle Detection, Cheapest Flights Within K Stops (Bellman-Ford variant), Find the City With the Smallest Number of Neighbors at a Threshold Distance.

---

## Pattern: Floyd-Warshall

**Trigger.** All-pairs shortest paths — when you need the shortest distance between every pair of nodes, not just from a single source.

```java
public void floydWarshall(int[][] dist) { // dist[i][j] pre-filled with edge weights or infinity
    int n = dist.length;

    for (int k = 0; k < n; k++) {
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (dist[i][k] != Integer.MAX_VALUE && dist[k][j] != Integer.MAX_VALUE
                        && dist[i][k] + dist[k][j] < dist[i][j]) {
                    dist[i][j] = dist[i][k] + dist[k][j];
                }
            }
        }
    }
}
```

**Edge cases.**
- The loop order matters: `k` (the intermediate node) must be the outermost loop — this is what makes the DP correct, since it represents "shortest path using only nodes 0..k as intermediates."
- O(n³) time — only viable when n is small (roughly n ≤ 400-500 depending on time limits); for large sparse graphs, running Dijkstra from every node is often faster in practice.
- A negative cycle shows up as `dist[i][i] < 0` for some node `i` after the algorithm completes — check the diagonal if the problem asks about negative cycles.

**Problems:** Transitive Closure, All-Pairs Shortest Path, Detect Negative Cycle Using Floyd-Warshall.

---

# 12. Heap

Priority queue for efficient min/max retrieval. Java's `PriorityQueue` is a min-heap by default — the recurring skill in this topic is choosing the right heap direction and the right comparator, not implementing the heap yourself (except when the problem explicitly asks you to).

## Pattern: Top-K Elements

**Trigger.** Maintaining the k largest or k smallest elements, especially over a stream where elements arrive one at a time.

```java
public int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> minHeap = new PriorityQueue<>(); // min-heap keeps the k largest

    for (int num : nums) {
        minHeap.offer(num);
        if (minHeap.size() > k) {
            minHeap.poll(); // evict the smallest, keeping only the k largest
        }
    }
    return minHeap.peek();
}

public int[] findMedianStream() {
    // Two-heap design for Find Median from Data Stream
    return null; // see below
}

public class MedianFinder {
    private final PriorityQueue<Integer> small = new PriorityQueue<>(Collections.reverseOrder()); // max-heap
    private final PriorityQueue<Integer> large = new PriorityQueue<>(); // min-heap

    public void addNum(int num) {
        small.offer(num);
        large.offer(small.poll());
        if (large.size() > small.size()) {
            small.offer(large.poll());
        }
    }

    public double findMedian() {
        if (small.size() > large.size()) return small.peek();
        return (small.peek() + large.peek()) / 2.0;
    }
}
```

**Edge cases.**
- It's easy to invert the heap direction by mistake: a min-heap keeps the top-K *largest* values (because you evict the smallest each time you exceed size k); a max-heap keeps the top-K *smallest*.
- Two-heap median design must keep the heaps balanced within size 1 of each other after every insertion — the rebalancing step (`large.size() > small.size()`) has to run every single time, not just occasionally.
- `PriorityQueue` in Java is not thread-safe and does not guarantee FIFO order for equal-priority elements — irrelevant for most interview problems, but worth knowing.

**Problems:** K Frequent Words, Sort Characters by Frequency, Kth Largest Element in an Array, Top K Frequent Elements, Minimum Cost to Connect Ropes, Find Median from Data Stream.

---

## Pattern: Merge K Sorted

**Trigger.** Combining more than two sorted sequences (arrays or lists) efficiently — a min-heap holding one "current" element per source avoids full pairwise merging.

```java
public ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> pq = new PriorityQueue<>((a, b) -> a.val - b.val);
    for (ListNode node : lists) {
        if (node != null) pq.offer(node);
    }

    ListNode dummy = new ListNode(0);
    ListNode tail = dummy;

    while (!pq.isEmpty()) {
        ListNode smallest = pq.poll();
        tail.next = smallest;
        tail = tail.next;
        if (smallest.next != null) pq.offer(smallest.next);
    }
    return dummy.next;
}
```

**Edge cases.**
- This is O(N log k), where N is total elements and k is number of lists — meaningfully better than repeated pairwise merging (O(Nk)) when k is large, which is exactly why the heap approach is expected here.
- Null entries in the input `lists` array — filter these out before offering to the heap, since offering `null` throws a `NullPointerException`.

**Problems:** Find K Pairs with Smallest Sums, Merge K Sorted Lists, Smallest Range Covering Elements from K Lists.

---

## Pattern: Heap with Sliding Window

**Trigger.** Tracking a max/min/median within a moving window — a plain heap doesn't support efficient removal of arbitrary elements, so you need lazy deletion or a monotonic deque instead.

```java
public int[] maxSlidingWindow(int[] nums, int k) {
    Deque<Integer> deque = new ArrayDeque<>(); // stores indices, values decreasing front to back
    int[] result = new int[nums.length - k + 1];

    for (int i = 0; i < nums.length; i++) {
        while (!deque.isEmpty() && deque.peekFirst() <= i - k) {
            deque.pollFirst(); // remove indices that fell out of the window
        }
        while (!deque.isEmpty() && nums[deque.peekLast()] < nums[i]) {
            deque.pollLast(); // maintain decreasing order
        }
        deque.offerLast(i);

        if (i >= k - 1) {
            result[i - k + 1] = nums[deque.peekFirst()];
        }
    }
    return result;
}
```

**Edge cases.**
- Sliding Window Maximum is more naturally a monotonic deque problem than a heap problem, precisely because a heap can't cheaply remove an element that's fallen out of the window — a lazy-deletion heap (checking staleness on poll) also works but is more code for the same result.
- Task Scheduler needs a max-heap of remaining task counts plus a cooldown queue — don't confuse it with plain frequency counting; the cooldown constraint is the actual difficulty.

**Problems:** Task Scheduler, Sliding Window Maximum, Sliding Window Median.

---

## Pattern: Implementation of Heap

**Trigger.** You're asked to build the heap itself from an array — sift-up on insert, sift-down on extract.

```java
public class MinHeap {
    private final List<Integer> heap = new ArrayList<>();

    public void insert(int val) {
        heap.add(val);
        siftUp(heap.size() - 1);
    }

    public int extractMin() {
        int min = heap.get(0);
        int last = heap.remove(heap.size() - 1);
        if (!heap.isEmpty()) {
            heap.set(0, last);
            siftDown(0);
        }
        return min;
    }

    private void siftUp(int i) {
        while (i > 0) {
            int parent = (i - 1) / 2;
            if (heap.get(i) >= heap.get(parent)) break;
            Collections.swap(heap, i, parent);
            i = parent;
        }
    }

    private void siftDown(int i) {
        int n = heap.size();
        while (true) {
            int left = 2 * i + 1, right = 2 * i + 2, smallest = i;
            if (left < n && heap.get(left) < heap.get(smallest)) smallest = left;
            if (right < n && heap.get(right) < heap.get(smallest)) smallest = right;
            if (smallest == i) break;
            Collections.swap(heap, i, smallest);
            i = smallest;
        }
    }
}
```

**Edge cases.**
- Extracting from a single-element heap — guard the `siftDown` call so it doesn't run on an empty heap after removal.
- Parent/child index math (`(i-1)/2`, `2i+1`, `2i+2`) is the part people get wrong under pressure — write it down and verify with a small example before trusting it.

**Problems:** Implement Priority Queue, Implement Min Heap, Implement Max Heap.

---

## Pattern: Huffman Pattern

**Trigger.** Repeatedly combining the two smallest elements to minimize total combination cost — classic Huffman coding shape, but also appears in disguise as "minimum cost to merge/connect" problems.

```java
public int connectSticks(int[] sticks) {
    PriorityQueue<Integer> minHeap = new PriorityQueue<>();
    for (int stick : sticks) minHeap.offer(stick);

    int totalCost = 0;
    while (minHeap.size() > 1) {
        int first = minHeap.poll();
        int second = minHeap.poll();
        int combined = first + second;
        totalCost += combined;
        minHeap.offer(combined);
    }
    return totalCost;
}
```

**Edge cases.**
- Single-element input — no combination needed, cost is 0; the `while (size > 1)` guard handles this naturally.
- Reorganize String is the same "always take from the top of a max-heap by frequency" idea, but needs an extra check that the most frequent character doesn't exceed `(n+1)/2` occurrences, or no valid arrangement exists at all.

**Problems:** Minimum Cost to Connect Sticks, Minimum Cost of Ropes, Merge Files with Minimum Cost, Combine Cards/Numbers with Minimum Cost, Reorganize String.

---

# 13. Backtracking

Building a solution incrementally and undoing choices that don't pan out. Every template in this topic follows the same shape: choose, explore, un-choose. The differences between sub-patterns are almost entirely in what counts as a valid "choice" and when to prune.

## Pattern: Choice-Based Backtracking

**Trigger.** Generate all combinations, subsets, or permutations — the choice at each step is simply "which unused element goes next."

```java
public List<List<Integer>> permute(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, new ArrayList<>(), new boolean[nums.length], result);
    return result;
}

private void backtrack(int[] nums, List<Integer> current, boolean[] used, List<List<Integer>> result) {
    if (current.size() == nums.length) {
        result.add(new ArrayList<>(current));
        return;
    }
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;
        used[i] = true;
        current.add(nums[i]);
        backtrack(nums, current, used, result);
        current.remove(current.size() - 1);
        used[i] = false;
    }
}

public List<List<Integer>> combinationSum(int[] candidates, int target) {
    List<List<Integer>> result = new ArrayList<>();
    Arrays.sort(candidates);
    backtrackSum(candidates, target, 0, new ArrayList<>(), result);
    return result;
}

private void backtrackSum(int[] candidates, int remaining, int start, List<Integer> current, List<List<Integer>> result) {
    if (remaining == 0) {
        result.add(new ArrayList<>(current));
        return;
    }
    for (int i = start; i < candidates.length; i++) {
        if (candidates[i] > remaining) break; // sorted array lets us prune early
        current.add(candidates[i]);
        backtrackSum(candidates, remaining - candidates[i], i, current, result); // reuse allowed: pass i, not i+1
        current.remove(current.size() - 1);
    }
}
```

**Edge cases.**
- "With duplicates" variants (Subsets II, Permutations II, Combination Sum II) require sorting first and explicitly skipping repeated values at the same recursion depth: `if (i > start && nums[i] == nums[i-1]) continue;` — this is different from skipping *used* elements and is the most commonly botched detail in this whole topic.
- Combination Sum allows reusing the same element (pass `i`, not `i + 1`, in the recursive call); Combination Sum II does not (pass `i + 1`) — mixing these up is a frequent source of wrong output.
- Always deep-copy `current` into the result list — storing the live reference means every subsequent backtrack mutation corrupts already-saved answers.

**Problems:** Subsets, Subsets II, Combination Sum, Combination Sum II, Permutations, Permutations II, Generate Parentheses, Palindrome Partitioning, Restore IP Addresses.

---

## Pattern: Constraint-Based Backtracking

**Trigger.** Each choice must satisfy a non-trivial constraint checked against previous choices before it's even attempted — N-Queens-style problems where most of the work is in the validity check, not the enumeration itself.

```java
public List<List<String>> solveNQueens(int n) {
    List<List<String>> result = new ArrayList<>();
    int[] queens = new int[n]; // queens[row] = column
    placeQueen(0, n, queens, result);
    return result;
}

private void placeQueen(int row, int n, int[] queens, List<List<String>> result) {
    if (row == n) {
        result.add(buildBoard(queens, n));
        return;
    }
    for (int col = 0; col < n; col++) {
        if (isSafe(queens, row, col)) {
            queens[row] = col;
            placeQueen(row + 1, n, queens, result);
        }
    }
}

private boolean isSafe(int[] queens, int row, int col) {
    for (int r = 0; r < row; r++) {
        int c = queens[r];
        if (c == col || Math.abs(c - col) == Math.abs(r - row)) return false; // same column or diagonal
    }
    return true;
}

private List<String> buildBoard(int[] queens, int n) {
    List<String> board = new ArrayList<>();
    for (int col : queens) {
        StringBuilder row = new StringBuilder("_".repeat(n));
        row.setCharAt(col, 'Q');
        board.add(row.toString());
    }
    return board;
}
```

**Edge cases.**
- Checking column and both diagonals for every placement is O(n) per check — for large n, tracking used columns/diagonals with boolean arrays or bitmasks brings this down to O(1) per check, which matters for N-Queens II (counting only) at larger board sizes.
- Partition to K Equal Sum Subsets: sort descending first so large elements are placed early, which prunes the search tree dramatically faster than ascending order — this single ordering choice is often the difference between passing and timing out.

**Problems:** Graph Coloring (M-Coloring), Knight's Tour, Partition to K Equal Sum Subsets, Matchsticks to Square, N-Queens, N-Queens II.

---

## Pattern: Grid / Path Backtracking

**Trigger.** Exploring paths through a grid recursively, marking cells visited, and unmarking them on backtrack — word search, maze problems, path-finding with full exploration.

```java
public boolean exist(char[][] board, String word) {
    for (int r = 0; r < board.length; r++) {
        for (int c = 0; c < board[0].length; c++) {
            if (search(board, word, r, c, 0)) return true;
        }
    }
    return false;
}

private boolean search(char[][] board, String word, int r, int c, int idx) {
    if (idx == word.length()) return true;
    if (r < 0 || c < 0 || r >= board.length || c >= board[0].length || board[r][c] != word.charAt(idx)) {
        return false;
    }

    char temp = board[r][c];
    board[r][c] = '#'; // mark visited in place

    boolean found = search(board, word, r + 1, c, idx + 1)
            || search(board, word, r - 1, c, idx + 1)
            || search(board, word, r, c + 1, idx + 1)
            || search(board, word, r, c - 1, idx + 1);

    board[r][c] = temp; // backtrack: restore the cell
    return found;
}
```

**Edge cases.**
- Always restore the cell's original value after exploring from it — forgetting the "un-mark" step is the single most common bug in this pattern, and it silently produces wrong answers rather than crashing.
- Word Search II (multiple words) should use a Trie built from all target words instead of calling the single-word search repeatedly — this shares prefix exploration across words and avoids massive redundant work.
- Short-circuit evaluation (`||`) in the four-direction search is intentional — it stops exploring further directions the moment one succeeds, which matters for performance, not just style.

**Problems:** Rat in a Maze, Path with Maximum Gold, Sudoku Solver, Word Search II, Unique Paths III.

---

## Pattern: Decision Tree / Sequence Generation

**Trigger.** Building strings or sequences where each position offers a set of choices — phone keypad letter combinations, expression building with operators inserted between digits.

```java
public List<String> letterCombinations(String digits) {
    if (digits.isEmpty()) return new ArrayList<>();

    Map<Character, String> keypad = Map.of(
        '2', "abc", '3', "def", '4', "ghi", '5', "jkl",
        '6', "mno", '7', "pqrs", '8', "tuv", '9', "wxyz"
    );

    List<String> result = new ArrayList<>();
    buildCombination(digits, 0, new StringBuilder(), keypad, result);
    return result;
}

private void buildCombination(String digits, int index, StringBuilder current,
                               Map<Character, String> keypad, List<String> result) {
    if (index == digits.length()) {
        result.add(current.toString());
        return;
    }
    String letters = keypad.get(digits.charAt(index));
    for (char letter : letters.toCharArray()) {
        current.append(letter);
        buildCombination(digits, index + 1, current, keypad, result);
        current.deleteCharAt(current.length() - 1);
    }
}
```

**Edge cases.**
- Empty input string — return an empty list, not a list containing an empty string; this is an explicit edge case in the problem statement, easy to miss.
- Expression Add Operators needs to track the *last operand* separately to correctly handle multiplication's precedence over addition/subtraction when backtracking — a naive left-to-right evaluation without this gives wrong results for expressions mixing `+`/`-` with `*`.

**Problems:** Letter Combinations of a Phone Number, All Possible Full Binary Trees, Expression Add Operators, Word Break II.

---

# 14. Greedy

Making the locally optimal choice at each step and trusting it leads to a globally optimal result. This only works when the problem has the greedy-choice property — before applying a pattern here, it's worth a moment convincing yourself why the greedy choice can't be beaten by a different one, since greedy solutions that are almost right but subtly wrong are common.

## Pattern: Intervals & Reach

**Trigger.** Sorting intervals by start or end time, then sweeping through to merge, schedule, or track the farthest reachable point.

```java
public int[][] merge(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
    List<int[]> merged = new ArrayList<>();

    for (int[] interval : intervals) {
        if (merged.isEmpty() || merged.get(merged.size() - 1)[1] < interval[0]) {
            merged.add(interval);
        } else {
            merged.get(merged.size() - 1)[1] = Math.max(merged.get(merged.size() - 1)[1], interval[1]);
        }
    }
    return merged.toArray(new int[merged.size()][]);
}

public boolean canJump(int[] nums) {
    int maxReach = 0;
    for (int i = 0; i < nums.length; i++) {
        if (i > maxReach) return false; // this index is unreachable
        maxReach = Math.max(maxReach, i + nums[i]);
    }
    return true;
}

public int eraseOverlapIntervals(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[1] - b[1]); // sort by END time — the key greedy insight
    int count = 0, lastEnd = Integer.MIN_VALUE;

    for (int[] interval : intervals) {
        if (interval[0] >= lastEnd) {
            lastEnd = interval[1]; // keep this interval
        } else {
            count++; // must remove this one, it overlaps
        }
    }
    return count;
}
```

**Edge cases.**
- Merge Intervals sorts by *start* time; Non-overlapping Intervals and Minimum Arrows sort by *end* time — this distinction is the crux of the pattern, not a stylistic choice. Sorting by the wrong field gives a wrong answer that still compiles and runs.
- Use `<=` vs `<` carefully when checking overlap — whether touching intervals (`[1,2]` and `[2,3]`) count as overlapping depends on the specific problem's definition, and this off-by-one changes the answer.
- Jump Game II (minimum jumps, not just reachability) needs a slightly different sweep tracking the current jump's boundary and the farthest reach within it — don't reuse the plain `canJump` logic and expect it to also count jumps correctly.

**Problems:** Activity Selection, Merge Intervals, Insert Interval, Non-overlapping Intervals, Meeting Rooms II, Minimum Number of Arrows to Burst Balloons, Jump Game, Jump Game II, Car Pooling, Minimum Number of Taps to Open to Water Garden.

---

## Pattern: Sorting / Local Choice

**Trigger.** Sorting the input by some derived key, then making one pass of locally optimal decisions — largest number formation, fractional knapsack, task scheduling by frequency.

```java
public String largestNumber(int[] nums) {
    String[] strs = Arrays.stream(nums).mapToObj(String::valueOf).toArray(String[]::new);
    Arrays.sort(strs, (a, b) -> (b + a).compareTo(a + b)); // custom comparator: which concatenation is bigger

    if (strs[0].equals("0")) return "0"; // all zeros case

    StringBuilder sb = new StringBuilder();
    for (String s : strs) sb.append(s);
    return sb.toString();
}

public int[] partitionLabels(String s) {
    int[] lastIndex = new int[26];
    for (int i = 0; i < s.length(); i++) {
        lastIndex[s.charAt(i) - 'a'] = i;
    }

    List<Integer> result = new ArrayList<>();
    int start = 0, end = 0;

    for (int i = 0; i < s.length(); i++) {
        end = Math.max(end, lastIndex[s.charAt(i) - 'a']);
        if (i == end) {
            result.add(end - start + 1);
            start = i + 1;
        }
    }
    return result.stream().mapToInt(Integer::intValue).toArray();
}
```

**Edge cases.**
- Largest Number: the comparator must compare *concatenations* (`b+a` vs `a+b`), not the numeric or lexicographic value of the strings alone — `"9"` vs `"34"` needs `"934"` vs `"349"` compared, which is why plain string sort gives the wrong order.
- All-zero input to Largest Number produces a string of leading zeros unless explicitly checked — return `"0"` if the first sorted element is `"0"`.
- Partition Labels relies on tracking each character's *last* occurrence index — the partition boundary is only valid once the current position matches the farthest last-occurrence seen so far.

**Problems:** Maximum Units on a Truck, Largest Number, Fractional Knapsack, Partition Labels, Task Scheduler, Minimum Platforms, Next Permutation, Candy Distribution.

---

# 15. Dynamic Programming

Breaking a problem into overlapping subproblems and storing their solutions. The discipline that matters most here is defining the state precisely — what does `dp[i]` or `dp[i][j]` actually represent — before writing any transitions. Get the state definition wrong and every line after it is wrong too.

## Pattern: 1D / Linear DP

**Trigger.** The optimal answer at position `i` depends on a small, fixed number of previous positions — sequences, running counts, or simple decision chains.

```java
public int rob(int[] nums) { // House Robber
    int prevTwo = 0, prevOne = 0;
    for (int num : nums) {
        int current = Math.max(prevOne, prevTwo + num);
        prevTwo = prevOne;
        prevOne = current;
    }
    return prevOne;
}

public int numDecodings(String s) {
    if (s.charAt(0) == '0') return 0;
    int n = s.length();
    int prevTwo = 1, prevOne = 1;

    for (int i = 1; i < n; i++) {
        int current = 0;
        if (s.charAt(i) != '0') current += prevOne;

        int twoDigit = Integer.parseInt(s.substring(i - 1, i + 1));
        if (twoDigit >= 10 && twoDigit <= 26) current += prevTwo;

        prevTwo = prevOne;
        prevOne = current;
    }
    return prevOne;
}
```

**Edge cases.**
- Rolling variables (`prevOne`, `prevTwo`) instead of a full DP array bring space down from O(n) to O(1) — do this by default unless the problem needs to reconstruct the actual path/sequence, not just the final value.
- Decode Ways: a `'0'` at the start, or a `'0'` that can't be paired with the preceding digit into a valid 10-26 range, makes that decoding invalid — this problem has more edge cases hiding in it than it looks like at first glance; test on `"10"`, `"100"`, `"06"` explicitly.

**Problems:** Climbing Stairs, House Robber, Decode Ways.

---

## Pattern: 2D / Grid DP

**Trigger.** Movement through a grid, or any problem where state naturally depends on two indices (row/column, or two sequence positions) with local transitions.

```java
public int uniquePaths(int m, int n) {
    int[][] dp = new int[m][n];
    for (int i = 0; i < m; i++) dp[i][0] = 1;
    for (int j = 0; j < n; j++) dp[0][j] = 1;

    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
        }
    }
    return dp[m - 1][n - 1];
}

public int minPathSum(int[][] grid) {
    int rows = grid.length, cols = grid[0].length;
    int[][] dp = new int[rows][cols];
    dp[0][0] = grid[0][0];

    for (int i = 1; i < rows; i++) dp[i][0] = dp[i - 1][0] + grid[i][0];
    for (int j = 1; j < cols; j++) dp[0][j] = dp[0][j - 1] + grid[0][j];

    for (int i = 1; i < rows; i++) {
        for (int j = 1; j < cols; j++) {
            dp[i][j] = grid[i][j] + Math.min(dp[i - 1][j], dp[i][j - 1]);
        }
    }
    return dp[rows - 1][cols - 1];
}
```

**Edge cases.**
- Obstacles (Unique Paths II) need `dp[i][j] = 0` explicitly at obstacle cells — a plain sum-of-neighbors formula silently produces wrong (too high) counts if you forget this override.
- Dungeon Game requires working *backward* from the destination rather than forward from the start, because the minimum HP requirement at each cell depends on what's needed for the rest of the path ahead of it, not behind it — this reversed direction is the entire trick of that problem.
- Row/column base cases (first row, first column) must be initialized before the main double loop, since they have no "two neighbors" to take a min/max/sum over.

**Problems:** Running Sum 2D Array, Unique Paths, Unique Paths II, Minimum Path Sum, Maximum Path Sum in Grid, Minimum Falling Path Sum, Dungeon Game, Cherry Pickup.

---

## Pattern: DP on Strings

**Trigger.** Two strings (or a string against itself) being compared position by position — `dp[i][j]` represents a relationship between prefix/suffix `i` of one string and `j` of another. LCS, edit distance, and palindrome-subsequence problems all share this shape.

```java
public int longestCommonSubsequence(String text1, String text2) {
    int m = text1.length(), n = text2.length();
    int[][] dp = new int[m + 1][n + 1];

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1] + 1;
            } else {
                dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }
    return dp[m][n];
}

public int minDistance(String word1, String word2) { // Edit Distance
    int m = word1.length(), n = word2.length();
    int[][] dp = new int[m + 1][n + 1];

    for (int i = 0; i <= m; i++) dp[i][0] = i;
    for (int j = 0; j <= n; j++) dp[0][j] = j;

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1];
            } else {
                dp[i][j] = 1 + Math.min(dp[i - 1][j - 1], Math.min(dp[i - 1][j], dp[i][j - 1]));
            }
        }
    }
    return dp[m][n];
}
```

**Edge cases.**
- Base row and column (`dp[i][0]`, `dp[0][j]`) represent "transform to/from an empty string" — for Edit Distance these are `i` and `j` respectively (all insertions or all deletions), not zero; getting this wrong breaks the entire table.
- Longest Palindromic Subsequence is LCS of the string with its own reverse — recognizing this reduction saves writing a separate DP from scratch.
- Regular Expression Matching and Wildcard Matching need careful handling of `*` meaning "zero or more of the preceding element," which requires looking two characters back in the pattern, not one — this is one of the fiddlier DP-on-strings problems, expect to double check the transition against a few hand-traced examples.

**Problems:** Longest Common Subsequence, Longest Palindromic Subsequence, Minimum Insertions to Make String Palindrome, Minimum Insertions and Deletions, Edit Distance, Shortest Common Supersequence, Regular Expression Matching, Distinct Subsequences, Palindrome Partitioning II, Scramble String.

---

## Pattern: DP on Intervals

**Trigger.** The optimal solution for a range `[i, j]` depends on trying every possible split point `k` within that range and combining the two resulting sub-ranges — matrix chain multiplication, balloon bursting, stick cutting.

```java
public int mcm(int[] dims) { // Matrix Chain Multiplication: dims[i-1] x dims[i] is matrix i
    int n = dims.length - 1; // number of matrices
    int[][] dp = new int[n + 1][n + 1];

    for (int len = 2; len <= n; len++) {
        for (int i = 1; i <= n - len + 1; i++) {
            int j = i + len - 1;
            dp[i][j] = Integer.MAX_VALUE;
            for (int k = i; k < j; k++) {
                int cost = dp[i][k] + dp[k + 1][j] + dims[i - 1] * dims[k] * dims[j];
                dp[i][j] = Math.min(dp[i][j], cost);
            }
        }
    }
    return dp[1][n];
}

public int maxCoins(int[] nums) { // Burst Balloons
    int n = nums.length;
    int[] balloons = new int[n + 2];
    balloons[0] = balloons[n + 1] = 1;
    System.arraycopy(nums, 0, balloons, 1, n);

    int[][] dp = new int[n + 2][n + 2];

    for (int len = 1; len <= n; len++) {
        for (int left = 1; left <= n - len + 1; left++) {
            int right = left + len - 1;
            for (int k = left; k <= right; k++) {
                int coins = balloons[left - 1] * balloons[k] * balloons[right + 1]
                        + dp[left][k - 1] + dp[k + 1][right];
                dp[left][right] = Math.max(dp[left][right], coins);
            }
        }
    }
    return dp[1][n];
}
```

**Edge cases.**
- Loop order is by increasing *interval length*, not by row or column index — computing `dp[i][j]` requires all smaller sub-intervals already solved, and iterating by length is what guarantees that.
- Burst Balloons: think of `k` as the *last* balloon burst in the range `[left, right]`, not the first — this reframing (last, not first) is the entire insight that makes the DP transition correct, and it's genuinely non-obvious the first time you see it.
- These are O(n³) solutions — fine for n up to a few hundred, but will time out on larger inputs; know this complexity going in.

**Problems:** Matrix Chain Multiplication, Merge Intervals with Cost, Burst Balloons, Minimum Cost to Merge Stones, Minimum Cost to Cut a Stick, Evaluate Expression to True (Boolean Parenthesization).

---

## Pattern: DP on Trees / DAGs

**Trigger.** Recursion plus memoization where the state depends on tree structure — typically "include this node" vs "exclude this node," computed post-order so children are resolved before the parent.

```java
public int rob(TreeNode root) { // House Robber III
    int[] result = robHelper(root);
    return Math.max(result[0], result[1]);
}

private int[] robHelper(TreeNode node) {
    if (node == null) return new int[]{0, 0}; // {excluded, included}

    int[] left = robHelper(node.left);
    int[] right = robHelper(node.right);

    int excluded = Math.max(left[0], left[1]) + Math.max(right[0], right[1]);
    int included = node.val + left[0] + right[0]; // can't include children if this node is included

    return new int[]{excluded, included};
}
```

**Edge cases.**
- Returning a pair `{excluded, included}` from each recursive call, rather than a single value, is what avoids exponential recomputation here — without it, you'd need a separate memoization map keyed by node, which works too but is more code for the same result.
- Path Sum III as a tree-DP problem (rather than brute-force checking every node as a potential path start) uses a running prefix-sum map passed down through the recursion, similar in spirit to the HashMap Prefix-Sum pattern but applied along tree root-to-node paths instead of a linear array.

**Problems:** House Robber III, Path Sum III.

---

## Pattern: Knapsack / Subset Sum

**Trigger.** Choosing a subset of items under a weight/capacity constraint to maximize value, or determining whether some subset sums to a target — the classic 0/1, bounded, and unbounded variants.

```java
public boolean canPartition(int[] nums) { // Partition Equal Subset Sum
    int sum = Arrays.stream(nums).sum();
    if (sum % 2 != 0) return false;

    int target = sum / 2;
    boolean[] dp = new boolean[target + 1];
    dp[0] = true;

    for (int num : nums) {
        for (int cap = target; cap >= num; cap--) { // reverse order — 0/1 knapsack, each item used once
            dp[cap] = dp[cap] || dp[cap - num];
        }
    }
    return dp[target];
}

public int coinChange(int[] coins, int amount) { // unbounded knapsack — coins reusable
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1);
    dp[0] = 0;

    for (int coin : coins) {
        for (int cap = coin; cap <= amount; cap++) { // forward order — reuse allowed
            dp[cap] = Math.min(dp[cap], dp[cap - coin] + 1);
        }
    }
    return dp[amount] > amount ? -1 : dp[amount];
}
```

**Edge cases.**
- Loop direction is the entire difference between 0/1 knapsack (iterate capacity *backward*, so each item is only used once) and unbounded knapsack (iterate capacity *forward*, allowing reuse within the same item's pass) — mixing these up silently gives wrong answers rather than crashing, which makes it a dangerous bug to have.
- Coin Change: initialize `dp` with a sentinel larger than any achievable answer (like `amount + 1`), then check against that sentinel at the end — using `Integer.MAX_VALUE` directly risks overflow when you add 1 to it during transitions.
- Target Sum reduces to a subset-sum-with-partition problem (assigning + or − to each number is equivalent to splitting into two subsets whose difference equals the target) — recognizing this reduction turns an apparently different-looking problem into the same template as Partition Equal Subset Sum.

**Problems:** 0-1 Knapsack, Partition Equal Subset Sum, Partition with Given Difference, Coin Change, Coin Change II, Target Sum, Subset Sum, Combination Sum IV.

---

## Pattern: DP on Stocks

**Trigger.** Buy/sell sequencing with state that includes whether you're currently holding a position, and possibly a cooldown or per-transaction fee.

```java
public int maxProfit(int[] prices) { // with cooldown
    if (prices.length == 0) return 0;

    int hold = -prices[0];
    int sold = 0;
    int rest = 0;

    for (int i = 1; i < prices.length; i++) {
        int prevSold = sold;
        sold = hold + prices[i];
        hold = Math.max(hold, rest - prices[i]);
        rest = Math.max(rest, prevSold);
    }
    return Math.max(sold, rest);
}

public int maxProfitKTransactions(int k, int[] prices) {
    if (prices.length == 0) return 0;

    int[] buy = new int[k + 1];
    int[] sell = new int[k + 1];
    Arrays.fill(buy, Integer.MIN_VALUE);

    for (int price : prices) {
        for (int t = 1; t <= k; t++) {
            buy[t] = Math.max(buy[t], sell[t - 1] - price);
            sell[t] = Math.max(sell[t], buy[t] + price);
        }
    }
    return sell[k];
}
```

**Edge cases.**
- State transitions must be updated in the correct order within each iteration — using an already-updated value from the *same* iteration when the transition should reference the *previous* iteration's value is a subtle, easy-to-introduce bug (this is why `prevSold` is cached explicitly above).
- Empty price array, or a single price — zero profit possible, guard as a base case.
- The generalized k-transactions version subsumes "at most 2 transactions" (Stock III) as the special case `k = 2` — no need to memorize a separate template for that variant.

**Problems:** Best Time to Buy and Sell Stock, Stock II, Stock with Cooldown, Stock with Transaction Fee, Stock III, Stock IV.

---

# 16. Trie

Tree-based structure for efficiently storing and retrieving keys from a set of strings. The core idea across every sub-pattern: shared prefixes are stored once, so lookups cost O(word length) regardless of how many words are stored.

## Pattern: Basic Trie Operations

**Trigger.** Prefix-based lookup — checking if a word or prefix exists efficiently, especially with many repeated queries against the same dictionary.

```java
public class Trie {
    private final Trie[] children = new Trie[26];
    private boolean isEndOfWord = false;

    public void insert(String word) {
        Trie node = this;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) {
                node.children[idx] = new Trie();
            }
            node = node.children[idx];
        }
        node.isEndOfWord = true;
    }

    public boolean search(String word) {
        Trie node = traverse(word);
        return node != null && node.isEndOfWord;
    }

    public boolean startsWith(String prefix) {
        return traverse(prefix) != null;
    }

    private Trie traverse(String s) {
        Trie node = this;
        for (char c : s.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) return null;
            node = node.children[idx];
        }
        return node;
    }
}
```

**Edge cases.**
- Non-lowercase-a-through-z input (uppercase, digits, symbols) indexes outside the 26-slot array — normalize input first, or use a `HashMap<Character, Trie>` for children if the alphabet isn't fixed and small.
- "Add and Search Word" needs wildcard (`.`) support, which requires DFS branching into all 26 children at that position instead of a single deterministic traversal step.

**Problems:** Implement Trie (Prefix Tree), Add and Search Word, Longest Common Prefix, Longest Word in Dictionary, Search Suggestions System.

---

## Pattern: Word Break / Segmentation

**Trigger.** Splitting a string into valid dictionary words — Trie speeds up the "is this substring a valid word" check, usually combined with DP or backtracking to try all split points.

```java
public boolean wordBreak(String s, List<String> wordDict) {
    Set<String> dict = new HashSet<>(wordDict);
    boolean[] dp = new boolean[s.length() + 1];
    dp[0] = true;

    for (int i = 1; i <= s.length(); i++) {
        for (int j = 0; j < i; j++) {
            if (dp[j] && dict.contains(s.substring(j, i))) {
                dp[i] = true;
                break;
            }
        }
    }
    return dp[s.length()];
}
```

**Edge cases.**
- Naive substring generation is O(n²) or worse for the `substring` calls themselves — for large strings, a Trie lets you walk character by character and check word-endings without repeatedly slicing the string.
- Word Break II (return all segmentations, not just whether one exists) needs the full backtracking output, not just the boolean DP — combine memoization with the recursive construction of the actual sentences, or it becomes exponential without reuse.

**Problems:** Word Break, Replace Words, Concatenated Words.

---

## Pattern: Bitwise Trie / XOR

**Trigger.** Maximizing or minimizing XOR between pairs of numbers — build a trie over the binary representation of each number (typically 32 bits, most significant bit first), then greedily walk it to find the best XOR partner.

```java
public int findMaximumXOR(int[] nums) {
    class TrieNode {
        TrieNode[] children = new TrieNode[2];
    }

    TrieNode root = new TrieNode();
    for (int num : nums) {
        TrieNode node = root;
        for (int i = 31; i >= 0; i--) {
            int bit = (num >> i) & 1;
            if (node.children[bit] == null) node.children[bit] = new TrieNode();
            node = node.children[bit];
        }
    }

    int maxXor = 0;
    for (int num : nums) {
        TrieNode node = root;
        int currentXor = 0;
        for (int i = 31; i >= 0; i--) {
            int bit = (num >> i) & 1;
            int wantedBit = 1 - bit; // greedily prefer the opposite bit to maximize XOR
            if (node.children[wantedBit] != null) {
                currentXor |= (1 << i);
                node = node.children[wantedBit];
            } else {
                node = node.children[bit];
            }
        }
        maxXor = Math.max(maxXor, currentXor);
    }
    return maxXor;
}
```

**Edge cases.**
- Always process bits from most significant to least significant — greedy XOR maximization only works because a higher bit set in the result outweighs any combination of lower bits, so the greedy choice at each level is provably safe.
- Negative numbers in Java use two's complement — if the problem allows negatives, decide explicitly whether to treat the sign bit like any other bit or handle it separately, since this changes the trie depth and greedy logic.

**Problems:** Maximum XOR of Two Numbers in Array, Bit Manipulation / Subset XOR Problems, Maximum XOR With an Element From Array.

---

# 17. Bit Manipulation

Operating on data at the bit level. The recurring theme: XOR's self-cancelling property (`x ^ x = 0`, `x ^ 0 = x`) does an enormous amount of work across this entire topic — it's worth being genuinely fluent with it rather than memorizing individual problems.

## Pattern: Basic Bit Operations

**Trigger.** Detecting a single/missing number, counting set bits, or checking/toggling a specific bit position.

```java
public int singleNumber(int[] nums) { // every element appears twice except one
    int result = 0;
    for (int num : nums) result ^= num; // pairs cancel out via XOR
    return result;
}

public int hammingWeight(int n) {
    int count = 0;
    while (n != 0) {
        n &= (n - 1); // clears the lowest set bit
        count++;
    }
    return count;
}

public boolean isPowerOfTwo(int n) {
    return n > 0 && (n & (n - 1)) == 0; // power of two has exactly one set bit
}
```

**Edge cases.**
- `n & (n - 1)` clearing the lowest set bit is a genuinely useful trick beyond just counting bits — it also underlies the power-of-two check above, worth memorizing as a primitive, not just a one-off snippet.
- Single Number II and III (elements appearing three times, or two unique elements instead of one) need more than plain XOR — Single Number II typically uses bitwise counting per bit position mod 3, and Single Number III first XORs everything together, then uses the lowest set bit of that result to partition the array into two groups, each containing one unique number.
- Negative numbers and Java's `>>` (arithmetic, sign-extending) vs `>>>` (logical, zero-filling) shift — using the wrong one silently changes results for negative inputs.

**Problems:** Missing Number, Number of 1 Bits, Alternating Bits, Check Kth Bit, Power of Two, Single Number, Single Number II, Single Number III.

---

## Pattern: Subsets / Bitmask

**Trigger.** Iterating over all 2ⁿ subsets of a small set by treating each integer from 0 to 2ⁿ−1 as a bitmask, where bit `i` set means element `i` is included.

```java
public List<List<Integer>> subsetsViaBitmask(int[] nums) {
    int n = nums.length;
    List<List<Integer>> result = new ArrayList<>();

    for (int mask = 0; mask < (1 << n); mask++) {
        List<Integer> subset = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if ((mask & (1 << i)) != 0) {
                subset.add(nums[i]);
            }
        }
        result.add(subset);
    }
    return result;
}
```

**Edge cases.**
- Only viable when n is small (roughly n ≤ 20), since the loop runs 2ⁿ times — for larger n, this pattern isn't feasible regardless of how tight the inner loop is.
- Partition to K Equal Sum Subsets can be solved with bitmask DP (`dp[mask]` = whether this subset of used elements can be partitioned so far) as an alternative to plain backtracking — worth knowing both approaches, since the bitmask DP version is often faster in practice for moderate n.

**Problems:** Subsets, Subsets II, Partition to K Equal Sum Subsets.

---

## Pattern: Advanced XOR

**Trigger.** Maximizing or minimizing XOR over a range, subarray, or subset — usually combining XOR's algebraic properties with prefix sums or a bitwise trie.

```java
public int subarrayXorQueries(int[] arr, int[][] queries) {
    int n = arr.length;
    int[] prefixXor = new int[n + 1];
    for (int i = 0; i < n; i++) {
        prefixXor[i + 1] = prefixXor[i] ^ arr[i];
    }
    // XOR of range [L, R] inclusive = prefixXor[R + 1] ^ prefixXor[L]
    int result = 0;
    for (int[] q : queries) {
        result ^= (prefixXor[q[1] + 1] ^ prefixXor[q[0]]);
    }
    return result;
}
```

**Edge cases.**
- Prefix XOR works the same way prefix sum does, but with XOR instead of addition — the range query formula (`prefixXor[R+1] ^ prefixXor[L]`) follows directly from `x ^ x = 0` cancelling the overlapping prefix.
- Subset XOR problems (find all achievable XOR totals over subsets) grow the achievable-set size fast — track achievable values in a `HashSet<Integer>` or use a bitwise trie for efficient max/min queries against them, rather than regenerating all 2ⁿ subsets from scratch each time.

**Problems:** Sum of Subset XOR Totals, Maximum XOR of Two Numbers in Array, Subarray XOR Queries / K-th XOR, Maximum XOR With an Element From Array.

---
