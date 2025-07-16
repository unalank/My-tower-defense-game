# My-tower-defense-game
import pygame
import math
import random
import os

pygame.init()
WIDTH, HEIGHT = 800, 600
WIN = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Tower Defense - Phases and Upgrades")

# Colors
WHITE = (255, 255, 255)
RED = (200, 0, 0)
GREEN = (0, 200, 0)
BLUE = (50, 50, 255)
YELLOW = (240, 240, 60)
BLACK = (0, 0, 0)
ORANGE = (255, 165, 0)
LIGHT_BLUE = (173, 216, 230)
BUTTON_COLOR = (0, 150, 0)
BUTTON_HOVER_COLOR = (0, 200, 0)

# Game States
LOADING = -1
MENU = 0
PLAYING = 1
HOW_TO_PLAY = 2
UPDATES = 3

current_game_state = LOADING

# Define the new complex path based on image_c2e5cd.png
PATH = [
    (0, 300),
    (200, 300),
    (250, 320),
    (280, 350),
    (300, 400),
    (300, 500),
    (350, 500),
    (450, 500),
    (550, 450),
    (600, 400),
    (700, 350),
    (WIDTH, 350)
]

FONT = pygame.font.SysFont(None, 30)
TITLE_FONT = pygame.font.SysFont(None, 70)
SUB_FONT = pygame.font.SysFont(None, 24)

# Load images - Initial loading for the loading screen itself
LOADING_SCREEN_IMAGE = None
try:
    LOADING_SCREEN_IMAGE = pygame.image.load(os.path.join("assets", "image_c3468e.png")).convert()
    LOADING_SCREEN_IMAGE = pygame.transform.scale(LOADING_SCREEN_IMAGE, (WIDTH, HEIGHT))
except pygame.error as e:
    print(f"Failed to load assets/image_c3468e.png, using fallback color. Error: {e}")

DEMOGORGON_IMAGE = None
MAIN_MENU_BG_IMAGE = None

# Tower level definitions (unchanged)
TOWER_LEVELS = [
    {"level": 1, "damage": 10, "range": 100, "cooldown": 60, "color": BLUE, "shape": "square"},
    {"level": 2, "damage": 20, "range": 110, "cooldown": 55, "color": RED, "shape": "triangle"},
    {"level": 3, "damage": 40, "range": 120, "cooldown": 50, "color": GREEN, "shape": "stars"},
    {"level": 4, "damage": 60, "range": 130, "cooldown": 45, "color": YELLOW, "shape": "circle"},
]
BONUS_TOWER_STATS = {"damage": 100, "range": 150, "cooldown": 70, "color": ORANGE, "shape": "bonus"}

MAX_TOWERS = 50
MAX_TOWERS_PER_SPOT = 2

class Enemy:
    def __init__(self, phase_index):
        self.phase_index = phase_index
        phase_data = PHASES[self.phase_index]
        self.x, self.y = PATH[0]
        self.vel = phase_data["speed"]
        self.health = phase_data["health"]
        self.max_health = phase_data["health"]
        self.color = phase_data["color"]
        self.sprite = phase_data["sprite"]
        self.radius = phase_data["radius"]
        self.damage_to_player = phase_data["damage_to_player"]
        self.damage_to_towers = phase_data["damage_to_towers"]
        self.path_index = 0

    def move(self):
        if self.path_index < len(PATH) - 1:
            target_x, target_y = PATH[self.path_index + 1]
            dx, dy = target_x - self.x, target_y - self.y
            dist = math.hypot(dx, dy)

            if dist > self.vel:
                self.x += self.vel * dx / dist
                self.y += self.vel * dy / dist
            else:
                self.x, self.y = target_x, target_y
                self.path_index += 1
        else:
            pass

    def draw(self, win):
        if self.sprite:
            rect = self.sprite.get_rect(center=(int(self.x), int(self.y)))
            win.blit(self.sprite, rect)
        else:
            pygame.draw.circle(win, self.color, (int(self.x), int(self.y)), self.radius)
        
        # Health bar
        bar_w = 40
        bar_h = 6
        health_ratio = self.health / self.max_health
        pygame.draw.rect(win, RED, (self.x - bar_w//2, self.y - self.radius - 15, bar_w, bar_h))
        pygame.draw.rect(win, GREEN, (self.x - bar_w//2, self.y - self.radius - 15, int(bar_w * health_ratio), bar_h))

# Redefined PHASES to include all enemy properties
PHASES = [
    {"name": "Normal", "health": 100, "speed": 2.0, "color": RED, "radius": 15, "sprite": None, "damage_to_player": 1, "damage_to_towers": 0},
    {"name": "Medium", "health": 150, "speed": 2.5, "color": (255, 100, 0), "radius": 17, "sprite": None, "damage_to_player": 1, "damage_to_towers": 0},
    {"name": "Zombie", "health": 200, "speed": 1.3, "color": (50, 150, 50), "radius": 18, "sprite": None, "damage_to_player": 1, "damage_to_towers": 0},
    {"name": "Overpowered", "health": 350, "speed": 2.8, "color": (100, 0, 100), "radius": 20, "sprite": None, "damage_to_player": 2, "damage_to_towers": 0},
    {"name": "Monster", "health": 600, "speed": 1.5, "color": (80, 40, 0), "radius": 22, "sprite": None, "damage_to_player": 3, "damage_to_towers": 1},
    {"name": "Demogorgon", "health": 800, "speed": 2.6, "color": (120, 120, 120), "radius": 24, "sprite": None, "damage_to_player": 4, "damage_to_towers": 1},
    {"name": "Uncontrollable", "health": 400, "speed": 3.5, "color": (255, 0, 255), "radius": 19, "sprite": None, "damage_to_player": 2, "damage_to_towers": 1},
    {"name": "Cooked", "health": 500, "speed": 1.8, "color": (255, 150, 150), "radius": 20, "sprite": None, "damage_to_player": 3, "damage_to_towers": 1},
    {"name": "Dimensions", "health": 700, "speed": 2.2, "color": (0, 255, 255), "radius": 23, "sprite": None, "damage_to_player": 4, "damage_to_towers": 1},
    {"name": "Bombarded", "health": 1000, "speed": 1.7, "color": (255, 255, 0), "radius": 25, "sprite": None, "damage_to_player": 5, "damage_to_towers": 1},
]


class Bullet:
    def __init__(self, x, y, target, damage):
        self.x = x
        self.y = y
        self.target = target
        self.vel = 10
        self.damage = damage
        self.radius = 6
        self.color = YELLOW

    def move(self):
        if self.target is None or not hasattr(self.target, 'health') or self.target.health <= 0:
            return False
        
        dx = self.target.x - self.x
        dy = self.target.y - self.y
        dist = math.hypot(dx, dy)
        if dist < self.vel:
            return False
        self.x += self.vel * dx / dist
        self.y += self.vel * dy / dist
        return True

    def draw(self, win):
        pygame.draw.circle(win, self.color, (int(self.x), int(self.y)), self.radius)

class Tower:
    def __init__(self, x, y, tower_type="standard"):
        self.x = x
        self.y = y
        self.tower_type = tower_type
        self.level = 1
        self.is_bonus = False

        if tower_type == "standard":
            self.set_level_stats(self.level)
        elif tower_type == "bonus":
            self.damage = BONUS_TOWER_STATS["damage"]
            self.range = BONUS_TOWER_STATS["range"]
            self.cooldown = BONUS_T
