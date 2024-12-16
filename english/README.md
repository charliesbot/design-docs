# Advent Of Code: Day 01, 2024 {#advent-of-code:-day-01,-2024}

**Link:** [design-doc](https://docs.google.com/document/d/1bkR_rfu39NeNWETZT7zseFpxWMqhP4Mz9vhWKgtCNkA)  
**Author(s):** [Charlie L](mailto:me@gmail.com)  
**Status:** In progress  
**Last Updated:** MMM dd, yyyy

[Advent Of Code: Day 01, 2024](#advent-of-code:-day-01,-2024)

[Links](#links)

[Context](#context)

[Detailed Design](#detailed-design)

[Parsing the input](#parsing-the-input)

[Formatting output as a list of numbers](#formatting-output-as-a-list-of-numbers)

[Calculate the differences of all pairs](#calculate-the-differences-of-all-pairs)

[Sort each list (preferred)](<#sort-each-list-(preferred)>)

[Use Binary Heap](#use-binary-heap)

**Document summary**  
This document outlines a solution for the Advent of Code 2024 Day 1 puzzle, which involves calculating the total difference between pairs of numbers in two lists. The solution provides two approaches: sorting each list and iterating through them to calculate the absolute differences, and using a binary heap to store and compare the smallest values from each list. The document includes code examples for both methods.

## Links {#links}

- [Exercise](https://adventofcode.com/2024/day/1)
- [Binary Heap](https://deno.land/std@0.177.0/collections/binary_heap.ts?s=BinaryHeap)

## Context {#context}

The puzzle presents a scenario where two groups of historians have compiled lists of location IDs, and the task is to determine how different these lists are.

The difference is determined by pairing the corresponding elements of the two lists, calculating the absolute difference for each pair, and then summing these differences.

Given the following input:

3 4  
4 3  
2 5  
1 3  
3 9  
3 3

We first calculate the difference between the smallest numbers in both lists. Next, we repeat the process with the second smallest numbers in each list, and so on.

After calculating the differences for all pairs, we sum them up.  
This total represents the **solution to the puzzle.**

## Detailed Design {#detailed-design}

### Parsing the input {#parsing-the-input}

Advent of Code delivers the input as a blob that we can add to a repository as our best convenience.  
For practicing purposes, we’ll add it as a _.txt_ file.

Considering that we will keep reading more files in following exercises, we can create a util that we can reuse across them.

Deno prefers using Web Standards over custom APIs, that’s why we need to use [Streams API](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API).

For this document purposes, we will assume the text input was already parsed, where the output is an array of strings.

### Formatting output as a list of numbers {#formatting-output-as-a-list-of-numbers}

Considering that we need to identify the left and the right list, we can

- Iterate through every line of the array
- Split the string by the whitespace
- Convert each split into a number

**`export`** `async function getListsAsNumbers(lines: string[]) {`  
 `const leftList: number[] = [];`  
 `const rightList: number[] = [];`  
 `lines.forEach((line) => {`  
 `const [left, right] = line.trim().split(“ “).map(Number);`  
 `leftList.push(left);`  
 `rightList.push(right);`  
 `});`

`return { leftList, rightList };`  
`}`

### Calculate the differences of all pairs {#calculate-the-differences-of-all-pairs}

The core problem of the puzzle relies on comparing pairs of the smallest numbers between lists.  
There are multiple ways to accomplish this goal.

#### Sort each list (preferred) {#sort-each-list-(preferred)}

This is the easiest approach, as we can rely on common JavaScript APIs to sort the list by the smallest ones, and then just iterate through both lists, get the difference between pairs, and sum the results.

**`function`** `totalDistanceBetweenLists(left: number[], right: number[]) {`  
 `left.sort((a, b) => a - b);`  
 `right.sort((a, b) => a - b);`  
 `let totalDistance = 0;`

`for (let i = 0; i < left.length; i++) {`  
 `totalDistance += Math.abs(left[i] - right[i]);`  
 `}`

`return totalDistance;`  
`}`

Time Complexity: O (n log n)

| Pros                                                                               | Cons                                                                                                                                  |
| ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| Great readability: straightforward and easy to understand No External Dependencies | Side Effects: sort method modifies the original arrays, which might cause issues if we intend to maintain the original order of them. |

#### Use Binary Heap {#use-binary-heap}

Deno offers a [Binary Heap](https://deno.land/std@0.177.0/collections/binary_heap.ts?s=BinaryHeap) utility as part of their standard library. The heap is in descending order by default, which is useful for the goal of this puzzle.

We can iterate both lists, store them in the binary heap, and then iterate through the heaps to get the difference between both of them and get the sum.

**`import`** `{ BinaryHeap, ascend } from "https://deno.land/std@0.177.0/collections/binary_heap.ts";`

**`function`** `getHeap(array: number[]) {`  
 `const minHeap = new BinaryHeap<number>(ascend);`  
 `minHeap.push(...array);`  
 `return minHeap;`  
`}`

**`function`** `totalDistanceBetweenLists(left: number[], right: number[]) {`  
 `const leftHeap = getHeap(left);`  
 `const rightHeap = getHeap(right);`  
 `let totalDistance = 0;`

`while (leftHeap.length > 0) {`  
 `totalDistance += Math.abs(leftHeap.pop() - rightHeap.pop());`  
 `}`

`return totalDistance;`  
`}`

Time Complexity: O (n log n)

| Pros                                                                                                             | Cons                                                                                                                                                                                                                       |
| ---------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Using a BinaryHeap explicitly signals that we need to efficiently retrieve the minimum element No Input Mutation | The getHeap helper function adds a layer of abstraction that, while potentially reusable, makes the main logic slightly less direct. Less readable We depend on data structures provided as an external dependency by Deno |
