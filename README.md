import pygame
import random

pygame.init()

# Constants
WIDTH, HEIGHT = 800, 600
FPS = 30

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
GREEN = (0, 255, 0)

# Speed
SPEED = 5  # Adjust this value to control the speed

# Player class
class Human(pygame.sprite.Sprite):
    def __init__(self, x, y):
        super().__init__()
        self.image = pygame.Surface((50, 50))
        self.image.fill(GREEN)
        self.rect = self.image.get_rect()
        self.rect.center = (x, y)
        self.sleepiness = 0
        self.hunger = 0
        self.decision_timer = random.randint(50, 150)

    def update(self):
        self.sleepiness += SPEED
        self.hunger += SPEED
        self.decision_timer -= 1

        if self.decision_timer <= 0:
            # Make a decision based on needs
            if self.sleepiness > self.hunger:
                self.move_towards(random.randint(0, WIDTH), random.randint(0, HEIGHT))
            else:
                self.move_towards(WIDTH // 2, HEIGHT // 2)

            # Reset decision timer
            self.decision_timer = random.randint(50, 150)

    def move_towards(self, target_x, target_y):
        dx = target_x - self.rect.centerx
        dy = target_y - self.rect.centery
        distance = max(1, (dx ** 2 + dy ** 2) ** 0.5)
        direction = (dx / distance, dy / distance)
        self.rect.x += direction[0] * SPEED
        self.rect.y += direction[1] * SPEED

# Monster class
class Monster(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = pygame.Surface((30, 30))
        self.image.fill(RED)
        self.rect = self.image.get_rect()
        self.rect.center = (random.randint(0, WIDTH), random.randint(0, HEIGHT))

# Initialize Pygame
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Simulation Game with AI")
clock = pygame.time.Clock()

# Sprite Groups
all_sprites = pygame.sprite.Group()
humans = pygame.sprite.Group()
monsters = pygame.sprite.Group()

# Create Humans
for _ in range(10):
    human = Human(random.randint(0, WIDTH), random.randint(0, HEIGHT))
    all_sprites.add(human)
    humans.add(human)

# Create Monsters
for _ in range(5):
    monster = Monster()
    all_sprites.add(monster)
    monsters.add(monster)

# Game Loop
running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    # Update
    all_sprites.update()

    # Check for collisions between humans and monsters
    collisions = pygame.sprite.groupcollide(humans, monsters, False, True)
    for human, monster_list in collisions.items():
        human.hunger -= len(monster_list) * 10

    # Check for player stats
    for human in humans:
        if human.sleepiness >= 100 or human.hunger >= 100:
            print("Human needs were not met!")
            running = False

    # Draw
    screen.fill(BLACK)
    all_sprites.draw(screen)

    pygame.display.flip()
    clock.tick(FPS)

pygame.quit()
