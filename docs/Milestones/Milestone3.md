# Milestone 3: Maze Search Algorithm

### Purpose: 
The goal of this milestone was to implement a search algorithm that would allow our robot to traverse an entire maze as efficiently as possible. We start with a simple BFS-like algorithm for constructing movement instructions between squares in the maze, and eventually upgraded our search to implement [Dijkstra's algorith](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm). 

### Hardware Updates:
Going into this milestone, our robot was using right hand wall-following to traverse mazes. This only requred two wall sensors, a forward-facing and right-facing sensor, to accurately traverse the maze. Before writing our new algorithm, we knew that our robot would need accurate information about the walls on all sides to make informed decisions about where to go next. Therefore, we added a left-facing wall sensor. 

With sensors facing forward, left, and right, our robot could determine every wall surrounding it after entering an unexplored square. It can assume that there is no wall behind it because it just came from that square (except at the start, when we assume the robot starts with a wall behind it).

## Version 1: Movement Tree Algorithm (BFS)

When first envisioning an algorithm for traversing the maze, we wanted a generalized algorithm that could move the robot between square in the frontier of the maze (i.e. squares that have not been explored yet). Our first idea for doing this was to construct a tree of possible moves (nodes) from a square using [Breadth First Search (BFS)](https://en.wikipedia.org/wiki/Breadth-first_search) through the explored territory of the maze until a new frontier square was reached, then tracing through the tree to create a list of moves for the robot to follow to move to this new location. The algorithm looks like this:

```
let T be a tree of explored squares in the maze
let v be a possible move for the robot (node)
bfs-moves(T,v):
       let Q be a queue
       Q.push(v)
       while Q is not empty
         v = Q.pop()
         label v as visited
           if v is not in T, construct the movement path to v and return
           otherwise:
              for all possible moves w from v in the maze 
              (priority order: forward, left, right, backwards)
                if w moves to a node not visited by the algorithm
                  Q.push(w)
```

We created a `Node` data structure to maintain information about the graph. It stores information about the robot's location and orientation, the robot's previous move, and the move required to travel there.

```cpp
struct Node {
  byte position;      //corresponds to x,y coordinates of the robot
  char move;          //move used to get here 
                      //(forward: 'f', left turn: 'l', 
                      //right turn: 'r', no move: 's')
  byte orientation;   //orientation of the robot at this point 
                      //(North, South, East, or West)
  struct Node* parent;//the parent of this node 
                      //(the previous move)
};
```

The valid moves that the robot can make in this algorithm are "move forward", "turn left", "turn right" and "stop". The algorithm will find the next unexplored square that requires the least number of moves to reach, prioritized in the order listed. That is, if two unexplored squares are an equal number of moves away, the one that uses higher-priority moves will be chosen. 

<img src="https://media.giphy.com/media/3iySk9yKxzuh4hMkYz/giphy.gif" width="700" />

Once we found an unexplored square, our algorithm traverses the tree back to the starting node, adding the moves at each node to a stack along the way. The final stack contains an ordered list of moves that the robot can follow to get from its current location to the new unexplored node. 

```cpp
  let S be a stack of chars representing robot movements 
  (forward: 'f', left turn: 'l', right turn: 'r', no move: 's')
  let v be a move to an unexplored square
  while (v != NULL) {
    char next_move = v->move;
    S.push(next_move);
    v = v->parent;
  }
```

These moves can then be executed in order by popping elements off of the movement stack.

```cpp
while (!S.isEmpty())
  {
    char move = S.pop();
    //have robot perform [move] and update its coordinates and orientation
    performAction(move);
  }
```

To keep track of unvisited nodes in the bfs algorithm above, we used the arduino [QueueList](https://playground.arduino.cc/Code/QueueList) library. We used the [StackArray](https://playground.arduino.cc/Code/StackArray) library for generating the stack of moves to be performed by the robot. 

### Example running movement tree algorithm on 5x4 maze:
<iframe width="700" height="400" src="https://www.youtube.com/embed/Hyez9abC3vw" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

&nbsp;

### Advantages and Disadvantages:

While this algorithm was relatively simple to implement and resulted in accurate and somewhat-efficient traversal of the entire maze, we realized that it had some issues:

* Since the algorith requires keeping track of each individual move separately, the number of elements in the tree could exceed the number of squares in the graph. Given the constrained memory of the Arduino and the size of the `Node` data structure being stored (7 bytes as shown above), we realized that we may run into memory constraints when using this algorithm on a 9x9 maze. We ended up deciding to cut the size by storing the move and orientation in the same byte, but this still left the increased memory usage of storing each move.

* This algorithm weighs each of the given movements (forward, left, and right) equally, when in reality we may want to give then different weights. When timing our robot, we realized that on average a turn of 90 degrees took about 1/3 as long as moving forward between squares. To weigh these moves differently, we would need an algorithm that accounts for "distance" between nodes when deciding on the order to traverse the frontier (see our implementaton of Dijkstra below). 


## Version 2: Dijkstra's Algorithm

Since our algorithm above was already pretty close to [Dijkstra's algorithm](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm), we modified our algorithm to keep track of the "distance" of each node from the start node. The modified algorith is as follows:

```
let T be a tree of explored squares in the maze
let v be a possible move for the robot (node)
procedure dijkstra(T, v):
        let Q be a queue nodes in the frontier
        L.append(v)
        while L is not empty
           sort Q by shortest distance
           v = Q.pop() //node with shortest distance in Q
             if v is not in T, construct the path to v and return
             otherwise (have not found path to frontier yet):
                for all moves to neighboring nodes w from v in the maze 
                (priority order: forward, left, right, backwards)
                   if w has not been visited yet
                    Q.push(w)
                    else
                     if the path from v to w is shorter than dist(w)
                       update w with path from v
```

<img src="https://media.giphy.com/media/w79tdWasWjS4MEFXsP/giphy.gif" width="700" />

The data structure for `Node` now has an additional field `dist` for keeping track of distance from the start node.

```cpp
struct Node
{
  byte position; //corresponds to x,y coordinates of the robot
  byte move_and_or;    //contains the orientation of the robot 
                       //and move required to get here
  struct Node *parent; //the parent of this node 
                       //(the previous position and move)
  unsigned int dist; //distance from start to [position]
};

```

Since nodes are now prioritized by distance rather than move, the algorithm does not have to keep track of every individual turn made by the robot. Instead, each node can represent a movement between squares. That is, `'l'` now represents "turn left, then go forward" and `'r'` represents "turn right, then go forward". We also introduced new move `'t'` for going backwards, that represents "turn twice in place, then go forward." With this new representation of moves, we have condensed the maximum number of moves to store to the number of squares in the maze, and we have reduced the number of individual moves that need to be added to the movement stack. 

Instead of the `ListQueue` we used in the previous algorithm, we used the [LinkedList](https://www.arduinolibraries.info/libraries/linked-list) library to store elements. `LinkedList` features a useful `sort` function that allowed us to sort the elements of the queue using a comparison function, shown below: 

```cpp
int compare(Node *&a, Node *&b)
{
  if (a->dist < b->dist)
    return -1;
  else if (a->dist == b->dist)
    return 0;
  else
    return 1;
}
```

Weighing the moves so that a turn has weight 1 and moving forward has weight 2, we get the following maze traversals:

See [our source code](../sourceCode.md) for the full implementation on our robot. 

### Running Dijkstra's algorithm on a 5x4 maze:
<iframe width="700" height="400" src="https://www.youtube.com/embed/NXd52I8BfZY" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

&nbsp;

### Traversing the second half of a full 9x9 maze with Dijkstra's algorithm: 
<iframe width="700" height="400" src="https://www.youtube.com/embed/oD0W9CT7CgI" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

**Note:** We provided some movement assistance to the robot at certain points due to it getting stuck on the tape, which did not affect the search algorithm. The robot misses a turn at the end (turning around instead of right) but maps the maze accurately because the walls on the path it took are almost identical to the intended path. 

&nbsp;

### Map of the 9x9 maze after complete traversal:

<img width="600" alt="maze1" src="https://user-images.githubusercontent.com/12742304/49325004-51758580-f508-11e8-8272-cc45d9ce8817.png">
