# Animated-Snake-Game
This  is a simple animated snake gmae
# pygame 120-FPS,Speed-10
import pygame
import sys
import random
import math

# ---------------- CONFIG ----------------
CELL_SIZE = 20        # size of each snake segment / food
GRID_WIDTH = 30       # number of cells horizontally
GRID_HEIGHT = 20      # number of cells vertically
FPS = 900             # frames per second for smooth animation
SNAKE_SPEED = 10       # snake moves X cells per second

# ---------------- INITIALIZE ----------------
pygame.init()
SCREEN = pygame.display.set_mode((GRID_WIDTH * CELL_SIZE, GRID_HEIGHT * CELL_SIZE))
pygame.display.set_caption("Animated Snake Game")
CLOCK = pygame.time.Clock()
FONT = pygame.font.SysFont("consols", 24)

# ---------------- HELPER FUNCTIONS ----------------

def new_food(snake):
    """Return a random position for food that is not on the snake."""
    empty_cells = [(x, y) for x in range(GRID_WIDTH) for y in range(GRID_HEIGHT) if (x, y) not in snake]
    return random.choice(empty_cells)

def rainbow_color(index):
    """Generate a rainbow color for the snake body based on segment index."""
    r = int(127 * (math.sin(index * 0.3) + 1))
    g = int(127 * (math.sin(index * 0.3 + 2) + 1))
    b = int(127 * (math.sin(index * 0.3 + 4) + 1))
    return (r, g, b)

def draw_snake(snake, timer):
    """Draw the snake with animated head and rainbow body."""
    for i, (x, y) in enumerate(snake):
        color = rainbow_color(i)
        if i == 0:
            # Animate the head with a pulsing effect
            pulse = 4 * math.sin(timer * 10)
            pygame.draw.rect(SCREEN, color, (x * CELL_SIZE - pulse/2, y * CELL_SIZE - pulse/2, CELL_SIZE + pulse, CELL_SIZE + pulse))
        else:
            pygame.draw.rect(SCREEN, color, (x * CELL_SIZE, y * CELL_SIZE, CELL_SIZE-1, CELL_SIZE-1))

def draw_food(food_x, food_y, timer):
    """Draw animated pulsating food."""
    pulse = 4 * math.sin(timer * 8)
    pygame.draw.ellipse(SCREEN, (255, 50, 50), 
                        (food_x * CELL_SIZE + 2 - pulse/2, food_y * CELL_SIZE + 2 - pulse/2, CELL_SIZE - 4 + pulse, CELL_SIZE - 4 + pulse))

def draw_screen(food, snake, game_over, timer):
    """Draw the entire game screen."""
    SCREEN.fill((0, 0, 0))            # clear screen
    draw_snake(snake, timer)          # draw snake
    draw_food(*food, timer)           # draw food

    if game_over:
        msg = FONT.render("GAME OVER", True, (255, 255, 255))
        msg2 = FONT.render("Press R to restart | ESC to exit", True, (255, 255, 255))
        SCREEN.blit(msg, msg.get_rect(center=(GRID_WIDTH*CELL_SIZE//2, GRID_HEIGHT*CELL_SIZE//2 - 20)))
        SCREEN.blit(msg2, msg2.get_rect(center=(GRID_WIDTH*CELL_SIZE//2, GRID_HEIGHT*CELL_SIZE//2 + 20)))

    pygame.display.update()

# ---------------- MAIN GAME FUNCTION ----------------
def game():
    # Initialize snake, direction, food
    snake = [(15, 10), (14, 10), (13, 10)]
    direction = (1, 0)
    food = new_food(snake)
    game_over = False

    move_timer = 0
    timer = 0  # used for animation effects

    while True:
        dt = CLOCK.tick(FPS) / 1000  # delta time in seconds
        move_timer += dt
        timer += dt

        # Handle events
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_ESCAPE:   # exit game
                    pygame.quit()
                    sys.exit()
                if event.key == pygame.K_r and game_over:   # restart
                    return game()
                if not game_over:
                    # prevent snake from moving backwards
                    if event.key in [pygame.K_w, pygame.K_UP] and direction != (0, 1): direction = (0, -1)
                    if event.key in [pygame.K_s, pygame.K_DOWN] and direction != (0, -1): direction = (0, 1)
                    if event.key in [pygame.K_a, pygame.K_LEFT] and direction != (1, 0): direction = (-1, 0)
                    if event.key in [pygame.K_d, pygame.K_RIGHT] and direction != (-1, 0): direction = (1, 0)

        # Move snake based on timer
        if move_timer >= 1 / SNAKE_SPEED and not game_over:
            move_timer = 0
            hx, hy = snake[0]
            dx, dy = direction
            new_head = (hx + dx, hy + dy)

            # Collision detection
            if new_head in snake or not (0 <= new_head[0] < GRID_WIDTH and 0 <= new_head[1] < GRID_HEIGHT):
                game_over = True
            else:
                snake.insert(0, new_head)
                if new_head == food:
                    food = new_food(snake)
                else:
                    snake.pop()

        # Draw everything
        draw_screen(food, snake, game_over, timer)

# ---------------- RUN GAME ----------------
game()

