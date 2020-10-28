# Shortest Path

## Problem

A maze can be describes as an array of strings. For example, a simple 4x4 maze can be described by the following array:

```
[
    "    ",
    " XX ",
    " XE ",
    "SX  "
]
```

Each location in the maze can be assigned one of the following characters:

- ' ': space (you can walk through this)
- 'X': wall (blocking your path)
- 'S': starting point
- 'E': ending point

Given a 2 dimensional maze of size n * m, described in an array of strings as per the above rules, create a function that
calculates the shorted path between the start end end points.

In the above example, the shortest path from the start point to the end point is 9 (denoted by the '.' characters indicating the path).

```
[
    "....",
    ".XX.",
    ".XE.",
    "SX  "
]
```

## Test Cases

```
solution(["S   E"])
# 4
```

```
solution([
    "    ",
    " XX ",
    " XE ",
    "SX  "
])
# 9
```

```
solution([
    '         X       X    ',
    '         X       X E  ',
    '                 X    ',
    ' XXXXX XXXXX     X    ',
    '    X  X         X    ',
    '    X  X         XXXX ',
    'XX  X  X   XXXXXXX    ',
    '    X  X         X    ',
    '    X  X              ',
    '  S X  X    XXXXX     ',
    '    X    XXXXX        '
])
# 49
```

## Solution

``` python
def getGridValue(grid, tuple):
    '''
        Returns the value within a 2d grid, located by 2-element tuple.
    '''
    return grid[tuple[0]][tuple[1]]
 
def setGridValue(grid, tuple, value):
    '''
        Sets a value within a 2d grid, located by 2-element tuple.
    '''
    grid[tuple[0]][tuple[1]] = value
   
def neighbors(maze, cell):
    '''
        Returns the neighbors to a cell. Diagonal neighbors are excluded.
        Neighbor must be inside the maze, and not a wall.
        Cell must be defined as a tuple, (row, col). (0,0) is top left of maze.
    '''
    neighbors = []
    directions = [
        (-1,0),(1,0),(0,-1),(0,1)
    ]
   
    for direction in directions:
        new = tuple(sum(x) for x in zip(cell, direction))
        if new[0] >= 0 and new[0] < len(maze) and new[1] >= 0 and new[1] < len(maze[new[0]]) and getGridValue(maze, new) != 'X':
            neighbors.append(new)
    
    return neighbors

def prettyPrint(maze, distances, start, end):
    '''
        Pretty-prints the solution for checking.
    '''
    pathCell = end
    distance = getGridValue(distances, pathCell)
    while distance > 1:
        distance = distance - 1
        for neighbor in neighbors(maze, pathCell):
            if getGridValue(distances, neighbor) == distance:
                pathCell = neighbor
                maze[neighbor[0]] = maze[neighbor[0]][:neighbor[1]] + '.' + maze[neighbor[0]][neighbor[1] + 1:]
                break
               
    return maze

def solution(maze):
    '''
        Takes a maze (array of strings), and draws the path from start to finish.
        The elements in the array must be as follows:
        a) ' ': empty space
        b) 'X': wall
        c) 'S': starting point
        d) 'E': end point
    '''
   
    # Setup
   
    distances = []
    queue = []
    start = ()
    end = ()
   
    # Set initial distances, and find start / end
   
    for y, row in enumerate(maze):
        distance = []
        for x, col in enumerate(row):
            if col == 'X':
                distance.append(-1)
            else:
                distance.append(10000000)
                if col == 'S':
                    start = (y, x)
                elif col == 'E':
                    end = (y, x)
   
        distances.append(distance)
   
    # Solve
   
    setGridValue(distances, start, 0)
    for neighbor in neighbors(maze, start):
        queue.append(neighbor)
        setGridValue(distances, neighbor, 1)
   
    while len(queue) > 0:
        cell = queue.pop(0)  # dequeue
        d = getGridValue(distances, cell) + 1
        neighbor = neighbors(maze, cell)
        for neighbor in neighbor:
            if getGridValue(distances, neighbor) > d:
                setGridValue(distances, neighbor, d)
                queue.append(neighbor)

    # At this point, if you want to pretty-print
    # the solution, uncomment the following line.
    # return prettyPrint(maze, distances, start, end)
    
    # return the shortest path
    return getGridValue(distances, end)

assert solution(["S   E"]) == 4
assert solution([
    "    ",
    " XX ",
    " XE ",
    "SX  "
]) == 9
assert solution([
    '         X       X    ',
    '         X       X E  ',
    '                 X    ',
    ' XXXXX XXXXX     X    ',
    '    X  X         X    ',
    '    X  X         XXXX ',
    'XX  X  X   XXXXXXX    ',
    '    X  X         X    ',
    '    X  X              ',
    '  S X  X    XXXXX     ',
    '    X    XXXXX        '
]) == 49
```