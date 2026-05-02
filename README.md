import pygame
import math
import random

pygame.init()

# okno
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("AI Arena - Auto Aim + Dodge + Super")

clock = pygame.time.Clock()

# pozice
player_pos = [WIDTH // 2, HEIGHT // 2]
enemy_pos = [100, 100]

# střely
player_bullets = []
enemy_bullets = []

# win/emote
win = False
win_time = 0

font = pygame.font.SysFont("arial", 50)

# super attack
super_ready = True
super_cooldown = 3000
last_super = 0

def get_angle(p1, p2):
    return math.atan2(p2[1] - p1[1], p2[0] - p1[0])

def shoot_player():
    angle = get_angle(player_pos, enemy_pos)
    player_bullets.append([player_pos[0], player_pos[1], angle])

def shoot_enemy():
    angle = get_angle(enemy_pos, player_pos)
    enemy_bullets.append([enemy_pos[0], enemy_pos[1], angle])

def super_attack():
    angle = get_angle(player_pos, enemy_pos)
    spread = [-0.4, -0.2, 0, 0.2, 0.4]

    for s in spread:
        player_bullets.append([
            player_pos[0],
            player_pos[1],
            angle + s
        ])

def dodge():
    for b in enemy_bullets:
        dx = player_pos[0] - b[0]
        dy = player_pos[1] - b[1]
        dist = math.hypot(dx, dy)

        if dist < 120:
            perp_x = -math.sin(b[2])
            perp_y = math.cos(b[2])

            player_pos[0] += perp_x * 5
            player_pos[1] += perp_y * 5

running = True
shoot_timer = 0
enemy_timer = 0

while running:
    screen.fill((25, 25, 25))

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    keys = pygame.key.get_pressed()

    # SUPER ATTACK
    if keys[pygame.K_SPACE] and super_ready:
        super_attack()
        super_ready = False
        last_super = pygame.time.get_ticks()

    if not super_ready:
        if pygame.time.get_ticks() - last_super > super_cooldown:
            super_ready = True

    # AUTO AIM
    shoot_timer += 1
    if shoot_timer > 25:
        shoot_player()
        shoot_timer = 0

    # enemy shoot
    enemy_timer += 1
    if enemy_timer > 40:
        shoot_enemy()
        enemy_timer = 0

    # dodge AI
    dodge()

    # update player bullets
    for b in player_bullets[:]:
        b[0] += math.cos(b[2]) * 7
        b[1] += math.sin(b[2]) * 7

        # hit enemy
        if abs(b[0] - enemy_pos[0]) < 12 and abs(b[1] - enemy_pos[1]) < 12:
            enemy_pos = [random.randint(50, WIDTH - 50), random.randint(50, HEIGHT - 50)]
            player_bullets.remove(b)

            win = True
            win_time = pygame.time.get_ticks()

    # update enemy bullets
    for b in enemy_bullets:
        b[0] += math.cos(b[2]) * 5
        b[1] += math.sin(b[2]) * 5

    # DRAW
    pygame.draw.circle(screen, (0, 255, 0), (int(player_pos[0]), int(player_pos[1])), 10)
    pygame.draw.circle(screen, (255, 0, 0), (int(enemy_pos[0]), int(enemy_pos[1])), 10)

    for b in player_bullets:
        pygame.draw.circle(screen, (255, 255, 0), (int(b[0]), int(b[1])), 4)

    for b in enemy_bullets:
        pygame.draw.circle(screen, (0, 150, 255), (int(b[0]), int(b[1])), 4)

    # EMOTE WIN
    if win:
        text = font.render("😎 WIN", True, (255, 255, 255))
        screen.blit(text, (player_pos[0] - 40, player_pos[1] - 80))

        if pygame.time.get_ticks() - win_time > 2000:
            win = False

    pygame.display.flip()
    clock.tick(60)

pygame.quit()
