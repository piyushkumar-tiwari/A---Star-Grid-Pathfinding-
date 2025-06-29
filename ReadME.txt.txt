
This Python code implements the A* pathfinding algorithm using the pygame library for visualization. Let's break down the different parts of the code.

Imports
import pygame
import math
from queue import PriorityQueue
Use code with caution
This section imports the necessary libraries:

pygame: Used for creating the graphical user interface and handling events.
math: Although imported, it is not explicitly used in the provided code snippet. It might be intended for other parts of the larger program.
queue.PriorityQueue: This is a data structure that will be used to efficiently manage the nodes (or "spots") to be explored during the pathfinding algorithm.
Game Window and Colors
WIDTH = 800
WIN = pygame.display.set_mode((WIDTH, WIDTH))
pygame.display.set_caption("A* Path Finding Algorithm")

RED = (255, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 255, 0)
YELLOW = (255, 255, 0)
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
PURPLE = (128, 0, 128)
ORANGE = (255, 165 ,0)
GREY = (128, 128, 128)
TURQUOISE = (64, 224, 208)
Use code with caution
WIDTH: This variable sets the width of the game window to 800 pixels.
WIN: This variable initializes the pygame display window with the specified width and height (since it's a square window).
pygame.display.set_caption(): This sets the title of the game window.
The subsequent lines define color tuples. These tuples represent RGB color values and are used to color the different states of the "spots" on the grid (e.g., start, end, barrier, path).
The Spot Class
class Spot:
    def __init__(self, row, col, width, total_rows):
        self.row = row
        self.col = col
        self.x = row * width
        self.y = col * width
        self.color = WHITE
        self.neighbors = []
        self.width = width
        self.total_rows = total_rows

    def get_pos(self):
        return self.row, self.col

    def is_closed(self):
        return self.color == RED

    def is_open(self):
        return self.color == GREEN

    def is_barrier(self):
        return self.color == BLACK

    def is_start(self):
        return self.color == ORANGE

    def is_end(self):
        return self.color == TURQUOISE

    def reset(self):
        self.color = WHITE

    def make_start(self):
        self.color = ORANGE

    def make_closed(self):
        self.color = RED

    def make_open(self):
        self.color = GREEN

    def make_barrier(self):
        self.color = BLACK

    def make_end(self):
        self.color = TURQUOISE

    def make_path(self):
        self.color = PURPLE

    def draw(self, win):
        pygame.draw.rect(win, self.color, (self.x, self.y, self.width, self.width))

    def update_neighbors(self, grid):
        self.neighbors = []
        if self.row < self.total_rows - 1 and not grid[self.row + 1][self.col].is_barrier(): # DOWN
            self.neighbors.append(grid[self.row + 1][self.col])

        if self.row > 0 and not grid[self.row - 1][self.col].is_barrier(): # UP
            self.neighbors.append(grid[self.row - 1][self.col])

        if self.col < self.total_rows - 1 and not grid[self.row][self.col + 1].is_barrier(): # RIGHT
            self.neighbors.append(grid[self.row][self.col + 1])

        if self.col > 0 and not grid[self.row][self.col - 1].is_barrier(): # LEFT
            self.neighbors.append(grid[self.row][self.col - 1])

    def __lt__(self, other):
        return False
Use code with caution
The Spot class represents each individual square or node in the grid.

The __init__ method initializes a spot with its row and col in the grid, its x and y pixel coordinates, an initial color (white), an empty list to store its neighbors, the width of the spot, and the total_rows in the grid.
get_pos(): Returns the row and col of the spot.
is_closed(), is_open(), is_barrier(), is_start(), is_end(): These methods check the current color of the spot to determine its state.
reset(), make_start(), make_closed(), make_open(), make_barrier(), make_end(), make_path(): These methods change the color of the spot to indicate a change in its state.
draw(self, win): This method draws the spot as a colored rectangle on the provided pygame window (win).
update_neighbors(self, grid): This method finds the valid neighbors for the current spot. It checks the spots directly up, down, left, and right. A neighbor is considered valid if it's within the grid boundaries and is not a barrier.
__lt__(self, other): This method is required for the PriorityQueue to compare Spot objects. In this case, it simply returns False, meaning the default comparison based on the order they are added to the priority queue will be used if the primary sorting key is the same.
Heuristic Function
def h(p1, p2):
    x1, y1 = p1
    x2, y2 = p2
    return abs(x1 - x2) + abs(y1 - y2)
Use code with caution
This function calculates the Manhattan distance between two points (p1 and p2). In the context of pathfinding, this is a common heuristic function used to estimate the distance from a current spot to the end spot. The A* algorithm uses this heuristic to guide its search.
Path Reconstruction
def reconstruct_path(came_from, current, draw):
    while current in came_from:
        current = came_from[current]
        current.make_path()
        draw()
Use code with caution
This function takes a dictionary came_from (which stores the predecessor of each spot on the optimal path), the current spot (which will be the end spot initially), and a draw function as input.
It traces back from the end spot to the start spot using the came_from dictionary, changing the color of each spot on the path to purple (PURPLE) to visualize the found path.
The draw() function is called after coloring each spot to update the display and show the path being reconstructed.
A* Algorithm
def algorithm(draw, grid, start, end):
    count = 0
    open_set = PriorityQueue()
    open_set.put((0, count, start))
    came_from = {}
    g_score = {spot: float("inf") for row in grid for spot in row}
    g_score[start] = 0
    f_score = {spot: float("inf") for row in grid for spot in row}
    f_score[start] = h(start.get_pos(), end.get_pos())

    open_set_hash = {start}

    while not open_set.empty():
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()

        current = open_set.get()[2]
        open_set_hash.remove(current)

        if current == end:
            reconstruct_path(came_from, end, draw)
            end.make_end()
            return True

        for neighbor in current.neighbors:
            temp_g_score = g_score[current] + 1

            if temp_g_score < g_score[neighbor]:
                came_from[neighbor] = current
                g_score[neighbor] = temp_g_score
                f_score[neighbor] = temp_g_score + h(neighbor.get_pos(), end.get_pos())
                if neighbor not in open_set_hash:
                    count += 1
                    open_set.put((f_score[neighbor], count, neighbor))
                    open_set_hash.add(neighbor)
                    neighbor.make_open()

        draw()

        if current != start:
            current.make_closed()

    return False
Use code with caution
This is the core of the pathfinding implementation, using the A* algorithm.

draw: A function to update the display.
grid: The grid of Spot objects.
start: The starting Spot.
end: The ending Spot.
count: Used as a tie-breaker in the priority queue.
open_set: A PriorityQueue to store the spots to be explored, ordered by their f-score.
came_from: A dictionary to keep track of the optimal path.
g_score: A dictionary to store the cost from the start spot to each spot. Initially set to infinity for all spots except the start.
f_score: A dictionary to store the estimated total cost from the start to the end through each spot (g-score + heuristic). Initially set to infinity for all spots except the start.
open_set_hash: A set to quickly check if a spot is already in the open_set.
The while loop continues as long as there are spots to explore in the open_set.
Inside the loop:
It handles pygame events, allowing the user to quit.
It retrieves the spot with the lowest f-score from the open_set.
If the current spot is the end spot, the path is found. It calls reconstruct_path to visualize the path and returns True.
It iterates through the neighbors of the current spot.
It calculates the temp_g_score for the neighbor (the cost to reach the neighbor from the current spot).
If the temp_g_score is less than the current g_score of the neighbor, it means a shorter path to the neighbor has been found.
It updates came_from, g_score, and f_score for the neighbor.
If the neighbor is not in the open_set_hash, it adds it to the open_set and open_set_hash, and changes its color to green (GREEN) to indicate it's being considered.
The draw() function is called to update the visualization.
If the current spot is not the start spot, its color is changed to red (RED) to indicate it has been visited.
If the open_set becomes empty and the end spot is not reached, it means no path exists, and the function returns False.
Grid Creation and Drawing
def make_grid(rows, width):
    grid = []
    gap = width // rows
    for i in range(rows):
        grid.append([])
        for j in range(rows):
            spot = Spot(i, j, gap, rows)
            grid[i].append(spot)

    return grid


def draw_grid(win, rows, width):
    gap = width // rows
    for i in range(rows):
        pygame.draw.line(win, GREY, (0, i * gap), (width, i * gap))
        for j in range(rows):
            pygame.draw.line(win, GREY, (j * gap, 0), (j * gap, width))


def draw(win, grid, rows, width):
    win.fill(WHITE)

    for row in grid:
        for spot in row:
            spot.draw(win)

    draw_grid(win, rows, width)
    pygame.display.update()
Use code with caution
make_grid(rows, width): This function creates a 2D list (the grid) of Spot objects. It divides the window width by the number of rows to determine the size of each spot (gap).
draw_grid(win, rows, width): This function draws the grey lines on the window to visually represent the grid structure.
draw(win, grid, rows, width): This function is responsible for drawing the entire grid on the window. It fills the background with white, draws each individual Spot object, and then draws the grid lines. Finally, it updates the pygame display to show the changes.
Getting Clicked Position
def get_clicked_pos(pos, rows, width):
    gap = width // rows
    y, x = pos

    row = y // gap
    col = x // gap

    return row, col
Use code with caution
This function takes the pixel coordinates of a mouse click (pos), the number of rows in the grid, and the window width.
It calculates the gap between grid lines and then determines which row and col in the grid correspond to the clicked pixel coordinates.
Main Game Loop
def main(win, width):
    ROWS = 50
    grid = make_grid(ROWS, width)

    start = None
    end = None

    run = True
    while run:
        draw(win, grid, ROWS, width)
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                run = False

            if pygame.mouse.get_pressed()[0]: # LEFT
                pos = pygame.mouse.get_pos()
                row, col = get_clicked_pos(pos, ROWS, width)
                spot = grid[row][col]
                if not start and spot != end:
                    start = spot
                    start.make_start()

                elif not end and spot != start:
                    end = spot
                    end.make_end()

                elif spot != end and spot != start:
                    spot.make_barrier()

            elif pygame.mouse.get_pressed()[2]: # RIGHT
                pos = pygame.mouse.get_pos()
                row, col = get_clicked_pos(pos, ROWS, width)
                spot = grid[row][col]
                spot.reset()
                if spot == start:
                    start = None
                elif spot == end:
                    end = None

            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE and start and end:
                    for row in grid:
                        for spot in row:
                            spot.update_neighbors(grid)

                    algorithm(lambda: draw(win, grid, ROWS, width), grid, start, end)

                if event.key == pygame.K_c:
                    start = None
                    end = None
                    grid = make_grid(ROWS, width)

    pygame.quit()

main(WIN, WIDTH)
Use code with caution
main(win, width): This is the main function that runs the simulation.
ROWS = 50: Sets the number of rows (and columns) in the grid.
grid = make_grid(ROWS, width): Creates the initial grid.
start = None, end = None: Variables to store the start and end spots, initially set to None.
The while run: loop is the main game loop that continues as long as run is True.
Inside the loop:
draw(win, grid, ROWS, width): Draws the current state of the grid.
It processes pygame events:
If the QUIT event is detected (e.g., closing the window), run is set to False to exit the loop.
If the left mouse button is pressed (pygame.mouse.get_pressed()[0]):
It gets the clicked position.
If no start spot is set and the clicked spot is not the end spot, the clicked spot is set as the start and its color is changed to orange.
If no end spot is set and the clicked spot is not the start spot, the clicked spot is set as the end and its color is changed to turquoise.
Otherwise (if both start and end are set, and the clicked spot is neither), the clicked spot is set as a barrier and its color is changed to black.
If the right mouse button is pressed (pygame.mouse.get_pressed()[2]):
It gets the clicked position.
The clicked spot is reset to its default white color.
If the reset spot was the start or end spot, the corresponding variable is set back to None.
If a key is pressed (event.type == pygame.KEYDOWN):
If the space key (pygame.K_SPACE) is pressed and both start and end spots are set:
It updates the neighbors for all spots in the grid.
It calls the algorithm function to find the path, passing a lambda function that redraws the grid each time the algorithm updates a spot's state.
If the 'c' key (pygame.K_c) is pressed:
It resets the start and end spots to None.
It creates a new, empty grid.
After the loop finishes, pygame.quit() is called to uninitialize pygame.
Finally, main(WIN, WIDTH) starts the main game loop.
This code provides a visual representation of the A* pathfinding algorithm, allowing the user to interactively set the start and end points and add barriers before running the algorithm to find the shortest path.