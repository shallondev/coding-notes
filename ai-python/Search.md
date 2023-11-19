# Search

## Definitions

Terminology 
- agent: entity that perceives its environment and acts upon that environment.
- state: configuration of a agent and its environment.
    - inital state: state where agent begins.
- actions: choices that can be made in a state.
    - ACTIONS(s): returns a set of actions that can be executed in state s.
- transition model: a description of what state results from performing any applicable action in any state.
    - RESULTS(s, a) returns the state resulting from performing action a in state s.
- state space: the set of all states reachable from the initial state by any sequence of actions.
    - represented by a graphh with nodes (states) and arrows showing reachable nodes (states).
- goal test: way to determine whether a given state is a goal state (solution).
- path cost: numerical cost associated with a given path.

Formulation of the search problem:
- initial state
- actions
- transition model
- goal test
- path cost function

Overall objective.
- optimal solution: find the soltion that has the lowest path cost among all solutions (there can be multiple optimal solutions). 

## Finding a solution

Need a data structure to hold data regarding where we are in relation to a soltion. We can use a a node.

Nodes contain:
- a state
- a parent (a node that generated this node)
- an action that was applied to the parent
- a path cost from initial state.

Approach:
- Start with a frontier data structure that contains the initial state.
- Repeat:
    - If frontier is empty, then there is no solution.
    - remove a node from the frontier.
    - If node contains a goal state, return the solution.
    - Expand node (look at neighborhood of nodes), add resulting nodes to the frontier.

Example: Find a path A to E.
```
A -> B -> D -> F
       -> C -> E
```
Frontier (initially): A <br>
Expand the node to add B as the frontier is not empty and A is not the goal state and remove A.<br>
Then we remove B to get to C, D in the frontier and repeat. <br>
Suppose we choose C then we remove C and look at E and bingo! we found the solution.

Issues with approach.
```
A <-> B -> D -> F
        -> C -> E
```
With the approach we defined above we can get stuck moving back and forth between A and B in our frontier never finding a solution.

Revised Approach:
- Contains frontier data structure only containing the initial state.
- Contains a empty data structure called the explored set.
- Repeat:
    - If empty, no solution.
    - Remove node from frontier.
    - If node contains goal state, return the solution.
    - Add node to explored set.
    - expand node, adding resulting nodes to the frontier except if they are in the frontier or in the explored set.

The order remove nodes is important!

## Depth First Search

ALways explore the deepest node expansion first in the frontier.

- Stack: last-in first-out data type.

Lets try using stack with our example:
```
A -> B -> D -> F
       -> C -> E
```
Frontier:
1. A 
2. B 
3. C, D (apply stack and explore D)
4. C, F (explore F)
5. C 
6. E

Explored Set:
1. A
2. B
3. D
4. F
5. C
6. E

## Breadth-first search

Search that always expands teh shallowest node in the frontier.

- queue: first-in first-out data type.

```
A -> B -> D -> F
       -> C -> E
```
Frontier
1. A
2. B
3. C, D
4. D, E
5. E, F
6. E is solution!

Explored Set
1. A
2. B
3. C
4. D
5. E

## Solving a Bredth vs Depth

- depth-first search will always find a soltuion so long the maze is finite.
    - The solution may not be optimal.
- bredth-first search tends to explore many states to find a solution.
    - The solution found by bredth will be optimal.

## Coding Bredth and Depth
```python
class Node():
    def __init__(self, state, parent, action):
        self.state = state
        self.parent = parent
        self.action = action
        # Path cost is not included because it can be calculuated later.
    
class StackFrontier():
    def __init__(self):
        self.frontier = []

    def add(self, node):
        self.frontier.append(node)

    # Checks if frontier contains a state.
    def contains_state(self, state):
        return any(node.state == state for node in self.frontier)

    # Checks if frontier is empty.
    def empty(self):
        return len(self.frontier) == 0
    
    def remove(self):
        if self.empty():
            raise Exception("empty frontier")
        else:
            # gets last item in list.
            node = self.frontier[-1]
            # remove add from frontier.
            self.frontier = self.frontier[:-1]
            return node

# Inherits from StackFrontier
class QueueFrontier(StackFrontier):

    def remove(self):
        if self.empty():
            raise Exception("empty frontier")
        else:
            node = self.frontier[0]
            self.frontier = self.frontier[1:]
            return node

def solve(self):

    # Keep track of # of states.
    self.num_explored = 0

    # Initialize frontier
    start = Node(state=self.start, parent=None, action=None)
    frontier = StackFrontier()
    frontier.add(start)

    # Initialize explored set
    self.explored = set()

    # Reapeats
    while True:

        # If empty no solution
        if frontier.empty():
            raise Exception("no solution")
        
        # Choose a node from frontier
        node = frontier.remove()
        self.num_explored += 1

        # Check if node is goal, if it is then we have a solution.
        if node.state == self.goal:
            actions = []
            cells = []

            # Follow parent nodes to solution
            while node.parent is not None:
                actions.append(node.action)
                cells.append(node.state)
                node = node.parent
            actions.reverse()
            cells.reverse()
            self.solution = (actions, cells)
            return
```
Note the code above i just snipits.

## Greedy Best-First Search
- Uninformed search: Strategy that uses no problem specific knowledge
- Informed search: Uses knowledge specific knowledge to find more efficient solutions.
- GBFS - search algorith that expands the node that is closest to the goals, as estimated by heuristic function h(n).
    - Manhattan Distance - a h(n) where we choose cells closer to the goal.
    - GBFS seems better than the BFS and DFS but you need a well defined heurestic!
    - Also, GBFS does not always find the best solution, specifically if the solution is found by taking steps that seem odd or weird.

Key Purpose: Using the knowledge of the layout (heurestic) of the problem we want to find the solution by choosing paths that we think would be good.


## A* Search
A modification of GBSF to fix the problem of finding not optimal solutions.
Expands that node with the lowest value of g(n) + h(n)
- g(n) = cost to reach node.
- h(n) = estimated cost to goal.

Optimal if
- h(n) is admissible (never overestimates the true cost).
- h(n) is consistent (for every node n and successor n' with step cose c, h(n) <= h(n') + c)

Main challenge: Finding h(n) to make the optimal solution!
Another issue: A* tends to use lots of memory. 

## Adversarial Search
Algorithm where there is a opponent.

## Minimax
Best for turn based games.
- Label each player as MAX(X) and MIN(O) where.
- Assign scores to different states like -1 for loss, 0 for tie, and 1 for a win.

Game
- S0: Initial State
- PLAYER(s): returns player state s
- ACTIONS(s): returns legal moves in state s.
- RESULTS(s, a): Returns state after action a taken in state a.
- Terminal(s): Checks if state s is terminal state (the game is over).

Example: Tik-Tac-Toe
- S0: Empty board.
- PLAYER(s): If X has moved then it is O's, if O moved then it is X's turn, if the board is empty it is X's turn.
- ACTIONS(s): Returns the set of possible cells X or O can play on.
- RESULT(s, a): Return the state with the placement of X or O added to the board.
- TERMINAL(s): If the board has 3 in a row of X's or O's return True otherwise return False.

When we do minimax as the name suggests with each move we want to first maximize our move but then also minimize the enemy.

Given state s:
- MAX picks action in a ACTIONS(s) that produces the highest value of MIN-VALUE(RESULT(s,a)).
- MIN picks action a in ACTIONS(s) that produces the smallest value of MAX-VALUE(RESULTS(s, a))

Minimax
```
function MAX_VALUE(state):
    if TERMINAL(state):
        return UTILITY(state)
    v = -infinity
    for action in ACTIONS(state):
        v = MAX(v, MIN_VALUE(RESULT(state, action)))
    return v

function MIN_VALUE(state):
    if TERMINAL(state):
        return UTILITY(state)
    v = +infinity
    for action in ACTIONS(state):
        v = MIN(v, MAX_VALUE(RESULT(state, action)))
    return v
```