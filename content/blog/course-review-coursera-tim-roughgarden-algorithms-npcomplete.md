---
title: "Course Review: Tim Roughgarden's Algorithms: Shortest Paths Revisited, NP-Complete Problems and What To Do About Them"
description: "Review of the forth and the last of the Algorithms specialization from Standford Online"
tags:
    - review
    - algorithms
    - computer-science
    - coursera
pubDatetime: 2025-07-24T18:27:23.887Z
modDatetime: 2025-09-06T18:06:15.000Z
---

[Shortest Paths Revisited, NP-Complete Problems and What To Do About Them](https://www.coursera.org/learn/algorithms-npcomplete) (henceforth referred to as Algo4) is the fourth and the last course in the Algorithms Specialization by Stanford Online. And I feel very excited to announce that I finished it last week 🕺.

(Respective blogs on the previous courses are here: [Algo2](/posts/course-review-coursera-tim-roughgarden-algorithms-2-graph-data-structure), [Algo1](/posts/course-review-coursera-tim-roughgarden-algorithms-1-divide-and-conquer), [Algo3](/posts/course-review-coursera-tim-roughgarden-algorithms-3-algorithms-greedy-dynamic-programming))

## Table of contents

## Module 1

**Difficulty: ★★☆☆☆ (Not Difficult)**

The first module starts off with the Bellman-Ford algorithm - an algorithm that improves on Dijkstra's on two fronts - 1) works on graphs with negative edge lengths, and 2) can be implemented in a distributed way, i.e., the input doesn't need to contain the entire graph.
Pretty easy, we do this with a dynamic approach. The module concludes with all-pairs shortest paths algorithms. It discusses three ways: running Dijkstra or Bellman-Ford for each vertex, a better O(n^3) approach using Floyd-Warshall (also works on graphs with negative edges), and 3) a neat Johnson's algorithm that reduces APSP to one invocation of Bellman-Ford, reweights the graph to get rid of negative edges (if and only if there are no negative cycles), and n invocations of Dijkstra's - pretty cool and runs blazingly fast.

## Module 2

**Difficulty: ★★★★☆ (Challenging)**

This module is about possibly the most important topic in the entire history of computing - Complexity Theory and NP-Completeness. Basically, an age-old question: are all problems equally hard? Can we solve all computable problems, or are there some problems that cannot be solved in polynomial time? Is P = NP? etc. This was mind-blowing and fascinating. I had an idea of what to expect with my little prior knowledge on the topic and Alan Turing's halting problem proof. This discusses the theoretical aspects of Class P, Class NP, and reductions to give a better understanding of the concepts and lingo. And then we discuss two NP-complete problems as an intro - Traveling Salesman and Vertex Cover. I really felt the difficulty of the TSP running time in this week's homework xD.

Since any NP-complete problem cannot be solved in polynomial time (unless, of course, P = NP), we can try to work around this by:

1. focusing on special cases of the problem, which often requires good domain knowledge
2. heuristics - sacrifice correctness for speed
3. employing clever brute-force search for exact solutions when correctness is a must, accepting a reasonable exponential running time

## Module 3

**Difficulty: ★★★★★ (Very Challenging)**

This module builds upon the last module's NP-completeness and revisits the Knapsack problem from [Algo3](/posts/course-review-coursera-tim-roughgarden-algorithms-3-algorithms-greedy-dynamic-programming), which turns out the general case of knapsack is NP-complete. We try to develop heuristics and also discuss the pros and cons of a greedy approach. Particularly, the observation for the heuristics is that if the values associated with items are not restricted, it has no polynomial-time solution. By rounding off some of the least significant digits to bound the maximum possible values, we can get an approximate solution that is within (1-e), where e is the user-supplied error tolerance parameter. This module was the hardest for me, particularly with all the math and proofs necessary to understand the heuristics, and then later implement the algorithms.

## Module 4

**Difficulty: ★★★★☆ (Challenging)**

The final module continues with discussing more NP-complete problems, namely the maximum cut problem in graphs (we did minimum cut in [Algo1](/posts/course-review-coursera-tim-roughgarden-algorithms-1-divide-and-conquer), which was polynomial-time solvable) and the local search algorithm as one of its applications, and the 2-SAT problem, a special case of n-SAT which is polynomial-time solvable. The professor mainly discusses Papadimitriou's algorithm, a randomized local search algorithm, but since the final homework was open-ended, I implemented my solution by reducing the problem to Strongly-Connected-Components of a graph. My SCC code from [Algo2](/posts/course-review-coursera-tim-roughgarden-algorithms-2-graph-data-structure) came in handy, with a slight modification for the final check, and I solved the final homework in a couple of hours ✅

## Conclusion

With that, I have completed all of the courses in the Algorithms Specialization! I have not decided where to focus my studies next; however, I'll be taking a week off for the time being. Doing some LeetCode problems is not out of the question, or taking CMU's database course, or catching up on my Japanese studies. Man, I feel so great!

**Update**: I decided to [start learning Systems Programming](/posts/x64-assembly-csapp-intro-to-computer-systems) using the book Computer Applications: A Programmer's Perspective, and CMU's 15-213:Intro to Systems course.

![anime proud gif](https://media1.tenor.com/m/WpjmnZCt4oUAAAAC/anime-boy.gif)
