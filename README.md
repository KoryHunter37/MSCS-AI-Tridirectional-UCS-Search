# MSCS Project

This repository overviews an Artificial Intelligence design project which I completed while completing my Master's in Computer Science at Georgia Tech. The source code for this project is not publically available, in compliance with the Georgia Institute of Technology Academic Honor Code<sup><a href="https://policylibrary.gatech.edu/student-affairs/academic-honor-code">[1]</a></sup>.

Instead, I have substituted my raw code with a guided walkthrough of the project itself, the steps I went through to complete it, and the skills I acquired by solving it.

# AI Search

Search is an integral part of AI. It helps in problem solving across a wide variety of domains where a solution isn’t immediately clear. I  implemented several graph search algorithms with the goal of optimally solving tri-directional search.


Here are the series of assigned tasks I completed, in ascending order of difficulty:

| Points    | Condition                                |
| --------- | ---------------------------------------- |
| 5 points | Design and implement a "Priority Queue" data structure. |
| 5 points | Implement and test breadth-first search over the test network. |
| 10 points | Implement uniform-cost search, using PriorityQueue as your frontier. |
| 10 points | Implement A* search using Euclidean distance as your heuristic. |
| 15 points | Implement bidirectional uniform-cost search. |
| 20 points | Implement bidirectional A* search.  |
| 20 points | Implement tridirectional search in the naive way: starting from each goal node, perform a uniform-cost search and keep expanding until two of the three searches meet. This should be one continuous path that connects all three nodes.  |
| 15 points | Implement tridirectional search in such a way as to consistently improve on the performance of the previous implementation. This means consistently exploring fewer nodes during the search in order to reduce runtime.  |

## Priority Queue

I implemented a simple data structure which became the core node representation for the remainder of my searches. It kept large amounts of nodes in order, each of which contained information about a location and held associated edge weight to neighbors.

This data structure was designed to order my search frontier, and that is exactly what it did. Based on the heapq<sup><a href="https://docs.python.org/2/library/heapq.html">[2]</a></sup> Python module, I was able to keep my inputs sorted, and maintain an O(1) amortized insertion time. Additionally, my removal time was at most O(log(n)).

## BFS, UCS, and A*

I was provided pickled data representations of Atlanta and Romania, and tasked with implementing these three classical searching algorithms. I was able to use my Priority Queue structure in simple implementations of breadth-first search<sup><a href="https://en.wikipedia.org/wiki/Breadth-first_search">[3]</a></sup>, uniform-cost search<sup><a href="https://en.wikipedia.org/wiki/Dijkstra's_algorithm">[4]</a></sup>, and A*<sup><a href="https://en.wikipedia.org/wiki/A*_search_algorithm">[5]</a></sup> search. For my A* search, I was restricted to using euclidean distance as my evaluation heuristic, so that is what I used to solve it.

In each implementation, I created a Priority Queue called "frontier", which kept track of nodes as I saw them, and followed the normal route through these search algorithms. Since my implementations do not deviate much from the textbook traditional implementation, I'll refrain from going into more specific details about these three.

## Bi-Directional UCS

After warming up with those fundamental implementations, I was tasked with improving performance by searching for a route through my demo maps from two directions at once.

<p align="center"><img width="400" height="300" src=images/romania.jpg></img></p>
<div align="center"><b>Fig 1. Map of Romania Used</b>
  (The Atlanta area map features <b>tens of thousands</b> of nodes, and is not suitable for image representation!)</div>

To implement this, I created **two** Priority Queues, called "frontier_start" and "frontier_goal". I also kept track of nodes which had been considered explored from each end of my search.

I alternated between the two Priortiy Queues, and at first I simply took turns between the two. However I soon realized this was not the most efficient approach-- and switched to working with whichever Priority Queue currently had the smallest node (in terms of smallest cost to reach that node, of all the nodes stored). Sometimes this even led to a very lopsided distribution of turn order, with one Priority Queue seeing much more activity than the other. This was actually very helpful, as it represented the performance improvement over my original erroneous method!

After selecting the optimal Priority Queue, if I wasn't already at the goal, I would have to examine my current node's state and neighbors.

For example, if the node I was examining now was already in the set of explored nodes from the *other end* of the problem, I could have found a solution!

If not, I still needed to look at all the neighbors of the current node, and add them to my frontier.

I also systematically updated the cost of any neighbors found, to see if a better path to them had been found than ones I was aware of thusfar. This ensured that I would always consider the best possibly paths to any node, and thus the correct cost, whenever any node was popped for evaluation when it was the "smallest" of all nodes in either Priority Queue!

Whether by reaching the goal state directly, or finding a node which had already been explored by the other end of the search, I would accrue optimal paths during my search. Eventually, the true best path was found and returned by this approach!

In the vast majority of cases, the best path was first found in the situation where a discovered node had already been explored by the other end of the search. In these cases, the bi-directional approach was **definitively better than the mono-directional approach**, as that route would not have been found without exhaustively searching to a goal node in the single direction implementation.

## Bi-Directional A*

The changes I made to my A* algorithm to convert it to bi-directional A* mirrored those which I made to my UCS implementation. As a result, it was much easier to solve this part of the problem. The only non-trivial difference between it and my bi-directional UCS algorithm is the use of an evaluation function to serve as a heuristic.

## Tri-Directional Search

This was the core of the project. When I talk about this project, I often have to clarify what "tri-directional search" really is, as that is not the common name for it. In reality, this is a combination of multiple bi-directional searches, but which respect eachother such that they are attempting to form a triangular route between three unique points. This is to simplify a problem in the form of, "I need to visit these three locations, but I don't care where I start and where I end up." A full circuit is not needed-- each must be visited once.

As such, it is important to establish **six different frontiers.** These are with respect to the "Start", "Middle", and "End" locations which we are trying to reach. (Of course, this designation is arbitrary-- in the optimal route it may turn out that starting at the node called "Middle" would be best. The naming simply helped me concepetualize the nodes I was working with.)

Frontiers exist:
* Between the Start and Middle points, starting from the Start
* Between the Start and Middle points, starting from the Middle
* Between the Middle and End points, starting from the Middle
* Between the Middle and End points, starting from the End
* Between the End and Start points, starting from the End
* Between the End and Start points, starting from the Start

I also created tracking for the explored nodes which mirrored each of these six frontiers. It was a lot to keep track of!

Much like in my bi-directional UCS and A*, I had to consider the order of evaluation for each of my Priority Queues. I chose the same approach I discovered to be best, only now with respect to all six queues instead of just two. (The smallest cost node was popped for evaluation.)

Another core part of this problem was deciding when to stop. My stopping condition varied from approach to approach, but finally I settled on conducting a series of tests to ensure that the ideal route had been found, while still ending as soon as I was sure it was optimal. Three tests were performed, one on each pair of frontiers (SM, ME, and ES). If either of the frontiers in the pair were empty, the test was passed. Additionally, if the next smallest value in either frontier was larger than the cost of an already found path between those two nodes, the test was passed. (This ensured that we would stop evaluating when we were looking at nodes further away than a path already found!)

If tests between all three pairs of frontiers passed, I could definitively say that the optimal solution had been found, as either the only remaining choices were worse than the path already found, or there were no choices left to pick from!

# Reflection

This project helped me expand on the foundational searches I had learned in my undergraduate education, by pushing me to maximize their efficiency in new ways, and more deeply understand *why* certain approaches were more efficient than others.

I found myself looking into stopping conditions in a way that I never had before. In my previous education, I had gotten used to having a stopping condition be quite clear-- but now I had two major hurdles to overcome. One, I had to identify when I could truly say that I had solved the problem. And two, I had to come up with a method for deciding this which was not computationally expensive.

One of the major pitfalls I fell into, which I'm also aware the majority of my peers fell into, was coming up with stopping conditions which required some recomputation. While this may have worked on the simple Romania map, it was absolutely not going to fly on the massively detailed Atlanta area map.

Understanding why an algorithm is optimal, how it can be proven to be optimal, and recognizing patterns where programmers often fall into redundency and inefficiency, is my core takeaway from this project.
