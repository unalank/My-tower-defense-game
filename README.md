import pygame
import math
import sys

pygame.init()
WIDTH, HEIGHT = 800, 600
WIN = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Tower Defense Level 2")

FONT = pygame.font.SysFont(None, 30)

# Colors
WHITE = (255, 255, 255)
RED = (255, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)
YELLOW = (255, 255, 0)
BLACK = (0, 0, 0)

# Path for enemies: left to right at y=300
PATH_START = (0, 300)
PATH_END = (WIDTH, 300)

class Enemy:
    def __init__(self):
        self.x, self.y = PATH_START
        self.speed = 2
        self.radius = 15
        self.health = 100
        self.max_health = 100
        self.color = RED
        self.alive = True

    def move(self):
        if self.x < PATH_END[0]:
            self.x += self.speed
        else:
            self.alive = False

    def draw(self, win):
        pygame.draw.circle(win, self.color, (int(self.x), int(self.y)), self.radius)
        # Draw health bar
        health_bar_width = 40
        health_bar_height = 5
        pygame.draw.rect(win, RED, (self.x - health_bar_width//2, self.y - 25, health_bar_width, health_bar_height))
        health_ratio = self.health / self.max_health
        pygame.draw.rect(win, GREEN, (self.x - health_bar_width//2, self.y - 25, health_bar_width * health_ratio, health_bar_height))

class Tower:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.color = BLUE
        self.range = 150
        self.fire_rate = 60  # frames between shots
        self.last_shot = 0
        self.damage = 20

    def draw(self, win):
        pygame.draw.rect(win, self.color, (self.x - 15, self.y - 15, 30, 30))
        # Draw range circle
        pygame.draw.circle(win, YELLOW, (self.x, self.y), self.range, 1)

    def can_shoot(self, tick):
        return tick - self.last_shot >= self.fire_rate

    def shoot(self, enemies, bullets, tick):
        if not self.can_shoot(tick):
            return
        # Find first enemy in range
        for enemy in enemies:
            dist = math.hypot(enemy.x - self.x, enemy.y - self.y)
            if dist <= self.range and enemy.alive:
                bullets.append(Bullet(self.x, self.y, enemy, self.damage))
                self.last_shot = tick
                break

class Bullet:
    def __init__(self, x, y, target, damage):
        self.x = x
        self.y = y
        self.target = target
        self.speed = 8
        self.radius = 5
        self.color = YELLOW
        self.damage = damage
        self.alive = True

    def move(self):
        if not self.target.alive:
            self.alive = False
            return
        dx = self.target.x - self.x
        dy = self.target.y - self.y
        dist = math.hypot(dx, dy)
        if dist < self.speed:
            # Hit enemy
            self.target.health -= self.damage
            if self.target.health <= 0:
                self.target.alive = False
            self.alive = False
        else:
            self.x += self.speed * dx / dist
            self.y += self.speed * dy / dist

    def draw(self, win):
        pygame.draw.circle(win, self.color, (int(self.x), int(self.y)), self.radius)

def main_game():
    clock = pygame.time.Clock()
    enemies = []
    towers = []
    bullets = []

    tick = 0
    spawn_interval = 120  # spawn enemy every 2 seconds

    lives = 10

    running = True
    while running:
        WIN.fill(WHITE)
        tick += 1

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            elif event.type == pygame.MOUSEBUTTONDOWN:
                pos = pygame.mouse.get_pos()
                # Place tower on click if not too close to existing tower
                if len(towers) < 50:
                    too_close = False
                    for t in towers:
                        if math.hypot(t.x - pos[0], t.y - pos[1]) < 40:
                            too_close = True
                            break
                    if not too_close:
                        towers.append(Tower(pos[0], pos[1]))

        # Spawn enemies periodically
        if tick % spawn_interval == 0:
            enemies.append(Enemy())

        # Update enemies
        for enemy in enemies[:]:
            enemy.move()
            if not enemy.alive:
                enemies.remove(enemy)
                lives -= 1
                if lives <= 0:
                    print("Game Over!")
                    running = False

        # Update towers shooting
        for tower in towers:
            tower.shoot(enemies, bullets, tick)

        # Update bullets
        for bullet in bullets[:]:
            bullet.move()
            if not bullet.alive:
                bullets.remove(bullet)

        # Draw everything
        for enemy in enemies:
            enemy.draw(WIN)

        for tower in towers:
            tower.draw(WIN)

        for bullet in bullets:
            bullet.draw(WIN)

        # Draw HUD
        lives_text = FONT.render(f"Lives: {lives}", True, BLACK)
        towers_text = FONT.render(f"Towers placed: {len(towers)}", True, BLACK)
        WIN.blit(lives_text, (10, 10))
        WIN.blit(towers_text, (10, 40))

        pygame.display.update()
        clock.tick(60)

if __name__ == "__main__":
    main_game()
