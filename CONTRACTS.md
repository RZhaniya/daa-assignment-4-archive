# Assignment 4 - exact contracts

Version 1.0.0

One JSON object per input line, one JSON result object per line. One task per request. No prompts or debug output on stdout. Independent requests reset all task state. Provided adapters handle input/output. Inputs satisfy the published domain; malformed JSON is not graded.

UTF-8; strings contain printable ASCII characters (32..126), case-sensitive; no Unicode normalization

All vertices, edges, characters and original blocks use zero-based indices. An edge ID is its position in the input edge array. All costs, distances, weights and counts use signed 64-bit integers; null denotes unreachable distance.

## Task 1 - Recover Shared Message / LCS

Recover an ordered common message from two damaged copies; missing characters may be skipped.

Input: {task:lcs,a:string,b:string}

Output: {length,sequence:string}

Limits: 0 <= |a|,|b| <= 1000; |a|*|b| <= 1000000.

Implement prefix dynamic programming and reconstruct one longest common subsequence. sequence must be a subsequence of both copies and have the optimal length. It need not be contiguous. Any optimal sequence is accepted. Empty copies return length=0 and sequence="". Explain states, transitions, base cases and reconstruction in ANALYSIS.md.

Complexity: O(|a|*|b|) time and memory are permitted.

Example input:

{"task": "lcs", "a": "tea_time", "b": "team"}

One accepted output:

{"length": 4, "sequence": "team"}

The final m is later in tea_time, so the recovered message is not a contiguous substring.

## Task 2 - Merge Archive Blocks / Interval DP

Combine neighbouring archive blocks without repeatedly copying the biggest block first.

Input: {task:merge,sizes:[...]}

Output: {cost,plan:[[l,k,r],...]}

Limits: 0 <= n <= 200; 0 <= size <= 1000000000.

A current block represents a closed original interval. Merge adjacent blocks [l,k] and [k+1,r]; pay their combined size and replace them by [l,r]. Return the minimum total cost and a chronological valid plan using original indices. Each input block begins as [i,i]. For n<=1 cost=0 and plan=[]. Any optimal plan is accepted. Greedily choosing the smallest two arbitrary blocks violates adjacency.

Complexity: O(n^3) time, O(n^2) memory; no advanced DP optimization is required.

Example input:

{"task": "merge", "sizes": [2, 3, 4]}

One accepted output:

{"cost": 14, "plan": [[0, 0, 1], [0, 1, 2]]}

First cost 5, then cost 9. The alternative costs 7+9=16.

## Task 3 - Recovery Team / Tree DP

Choose archive robots whose adjacent charging stations will not overload each other.

Input: {task:tree,n,weights:[...],edges:[[u,v],...]}

Output: {weight,selected:[vertex IDs,...]}

Limits: 0 <= n <= 100000; -1000000000 <= weight <= 1000000000. For n>0 input is a connected undirected tree with n-1 distinct edges; n=0 has no edges.

Implement maximum weight independent set by take/skip tree DP and reconstruct a selected set. The empty set is allowed, so all-negative inputs return weight 0. No adjacent vertices may both be selected. Any optimum is accepted. Root choice must not change the optimum; avoid recursion failure on deep chains.

Complexity: O(n) time and memory.

Example input:

{"task": "tree", "n": 3, "weights": [4, 9, 6], "edges": [[0, 1], [1, 2]]}

One accepted output:

{"weight": 10, "selected": [0, 2]}

The two outer robots contribute 10, beating the central robot's 9.

## Task 4 - Find a Phrase / Naive Search

Build a simple exact-search baseline for the archivist's favourite phrase.

Input: {task:naive,text:string,pattern:string}

Output: {positions:[...]}

Limits: 0 <= |text| <= 3000; 0 <= |pattern| <= 3000.

Implement naive character comparisons. Return ALL zero-based occurrence starts in strictly increasing order, including overlaps. Empty pattern matches every boundary 0..|text|, including the final boundary. Pattern longer than text returns []. No built-in search functions in the algorithm.

Complexity: O(n*m+n+1) time, O(number of answers) output memory.

Example input:

{"task": "naive", "text": "aaaaa", "pattern": "aaa"}

One accepted output:

{"positions": [0, 1, 2]}

Overlaps count: each alarm can start before the previous one has finished.

## Task 4 - Find a Phrase / KMP

Reuse known prefix matches instead of asking the same character the same question again.

Input: {task:kmp,text:string,pattern:string}

Output: {positions:[...]}

Limits: 0 <= |text| <= 200000; 0 <= |pattern| <= 100000.

Implement prefix function and KMP. Use exactly the same matching contract as naive, including the empty pattern. Keep overlaps after a full match using prefix fallback. Compare outputs of all three search algorithms on inputs in their common domain.

Complexity: O(n+m+number of answers) time, O(m+number of answers) memory.

Example input:

{"task": "kmp", "text": "abababa", "pattern": "aba"}

One accepted output:

{"positions": [0, 2, 4]}

Resetting the matched-prefix length to zero after a match can miss valid overlaps.

## Task 4 - Find a Phrase / Rabin-Karp

Use rolling fingerprints to reject unequal windows quickly, then verify each candidate.

Input: {task:rk,text:string,pattern:string}

Output: {positions:[...]}

Limits: 0 <= |text|,|pattern| <= 10000; |text|*|pattern| <= 20000000.

Implement polynomial rolling hash. Use base 31, modulus 101 and ASCII character codes; highest power belongs to the leftmost character. This deliberately small modulus makes collisions testable. Normalize negative remainders. Equal hashes are candidates only: compare actual characters before reporting. The exact matching contract, including empty patterns and overlaps, is shared with KMP. 64-bit arithmetic is sufficient. No built-in search functions.

Complexity: O(n+m+c*m) time where c is the number of equal-hash windows; worst case O(n*m). O(m+answers) rolling-window memory, or O(n+answers) with prefix hashes, is permitted. Do not claim deterministic linear time.

Example input:

{"task": "rk", "text": "abdj", "pattern": "ab"}

One accepted output:

{"positions": [0]}

ab and dj both hash to 75 modulo 101 with base 31. Only ab is an actual match.

## Task 5 - Repeated Messages / Suffix Array and LCP

Find the longest exact repeated fragment; the robot would like fewer duplicate 'please reboot' messages.

Input: {task:suffix,text:string}

Output: {sa:[...],lcp:[...],repeated:string}

Limits: 0 <= |text| <= 50000. Build exactly the nonempty suffixes; do not include a sentinel or the empty suffix in output.

Implement prefix doubling and Kasai. sa is a permutation of 0..n-1 in lexicographic suffix order. lcp has length n; lcp[0]=0 and lcp[i] compares sa[i-1] with sa[i]. Return any longest substring occurring at two distinct starts; overlaps are allowed. If none exists, repeated="". Empty input has both arrays empty. Do not materialize all suffix strings.

Complexity: O(n log^2(n+1)) time using comparison sorting of rank pairs is allowed; O(n log(n+1)) with radix/counting sorting is also allowed. Kasai O(n); O(n) auxiliary memory.

Example input:

{"task": "suffix", "text": "banana"}

One accepted output:

{"sa": [5, 3, 1, 0, 4, 2], "lcp": [0, 1, 3, 0, 0, 2], "repeated": "ana"}

ana starts at positions 1 and 3; these occurrences overlap.

## Task 6 - Alarm Dictionary / Aho-Corasick

Count every dictionary phrase across corrupted messages. 'tea' may be useful; 'reboot reboot' is usually less reassuring.

Input: {task:aho,patterns:[nonempty strings,...],texts:[strings,...]}

Output: {counts:[[count for each dictionary position],...]}

Limits: 0 <= number of patterns <= 10000; total pattern length <= 100000; 0 <= number of texts <= 100; total text length <= 200000. Patterns are nonempty; texts may be empty.

Build one Aho-Corasick trie/automaton per request, with suffix/failure links. Query it for each independent text; reset state and counts. Count overlaps and suffix-pattern matches. Duplicate patterns are separate dictionary positions and receive equal counts. An empty dictionary produces an empty count row per text. An empty texts array produces []. Do not replace the automaton by multiple KMP searches. Count results using visit propagation along failure links or correct output-link traversal.

Complexity: Build O(95*S) with fixed ASCII alphabet, or a justified sparse equivalent. Per-text counting O(|text|+S+p) with failure propagation is allowed, or O(|text|+matches+p) with output links. S is trie states and p is dictionary size. Avoid copying quadratically many inherited output lists.

Example input:

{"task": "aho", "patterns": ["he", "she", "he"], "texts": ["sheshe", "sheshe", ""]}

One accepted output:

{"counts": [[2, 2, 2], [2, 2, 2], [0, 0, 0]]}

he is a suffix of she; both count. Repeating the query does not accumulate earlier counts.