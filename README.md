# boom
# My first experience with gaming ( knowledge applied )
import pygame
import random
import math
from pygame import mixer

# intialiaze
pygame.init()
# clock = pygame.time.Clock()

# create the screen
screen = pygame.display.set_mode((800, 600))

# importing background of game into window
background = pygame.image.load('img.png')

# background music
mixer.music.load('background.wav')
mixer.music.play(-2)

# titles and icons
pygame.display.set_caption("Space Invaders")
icon = pygame.image.load('spaceship.png')
pygame.display.set_icon(icon)

# player
playerimg = pygame.image.load('player.png')
playerx = 370
playery = 480
playerx_change = 0

# enemy
enemyimg = []
enemyx = []
enemyy = []
enemyx_change = []
enemyy_change = []
no_of_enemies = 5

for i in range(no_of_enemies):
    enemyimg.append(pygame.image.load('enemy.png'))
    enemyx.append(random.randint(0, 751))
    enemyy.append(random.randint(50, 150))
    enemyx_change.append(0.3)
    enemyy_change.append(40)

# bullet
# ready = bullet not in motion
# fire = bullet in motion
bulletimg = pygame.image.load('bullet.png')
bulletx = 0
bullety = 480
bulletx_change = 0
bullety_change = 0.8
bullet_state = "ready"

# showing score
score_value = 0
font = pygame.font.Font('freesansbold.ttf', 32)

text_x = 20
text_y = 10

# game over text
over_font = pygame.font.Font('freesansbold.ttf', 32)


def show_score(x, y):
    score = font.render("Score : " + str(score_value), True, (0, 255, 0))
    screen.blit(score, (x, y))


def game_over_text():
    over_text = over_font.render("GAME OVER !", True, (0, 255, 0))
    screen.blit(over_text, (300, 250))


def player(x, y):
    screen.blit(playerimg, (x, y))


def enemy(x, y, i):
    screen.blit(enemyimg[i], (x, y))


def fire_bullet(x, y):
    global bullet_state
    bullet_state = "fire"
    screen.blit(bulletimg, (x + 10, y + 10))


def iscollision(enemyx, enemyy, bulletx, bullety):
    distance = math.sqrt((math.pow(enemyx - bulletx, 2)) + (math.pow(enemyy - bullety, 2)))
    if distance < 27:
        return True
    else:
        return False


# gameloop
running = True
while running:
    # background color
    screen.fill((0, 0, 0))
    # background image location
    screen.blit(background, (0, 0))

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_LEFT:
                playerx_change = -0.5
            if event.key == pygame.K_RIGHT:
                playerx_change = 0.5
            if event.key == pygame.K_SPACE:
                if bullet_state == "ready":
                    bullet_sound = mixer.Sound('laser.wav')
                    bullet_sound.play()
                    # getting the current coordiante of spaceship
                    bulletx = playerx
                    fire_bullet(playerx, bullety)
        if event.type == pygame.KEYUP:
            if event.key == pygame.K_LEFT or event.key == pygame.K_RIGHT:
                playerx_change = 0

    # screen boundary player
    playerx += playerx_change

    if playerx <= 0:
        playerx = 0
    elif playerx >= 752:
        playerx = 752

    # screen boundary enemy and enemy movement
    for i in range(no_of_enemies):

        # game over
        if enemyy[i] > 440:
            for j in range(no_of_enemies):
                enemyy[j] = 2000
            game_over_text()
            break

        enemyx[i] += enemyx_change[i]
        if enemyx[i] <= 0:
            enemyx_change[i] = 0.4
            enemyy[i] += enemyy_change[i]
        elif enemyx[i] >= 752:
            enemyx_change[i] = -0.4
            enemyy[i] += enemyy_change[i]
        # collisions
        collision = iscollision(enemyx[i], enemyy[i], bulletx, bullety)
        if collision:
            explosion_sound = mixer.Sound('explosion.wav')
            explosion_sound.play()
            bullety = 480
            bullet_state = "ready"
            score_value += 1
            enemyx[i] = random.randint(0, 751)
            enemyy[i] = random.randint(50, 150)
        enemy(enemyx[i], enemyy[i], i)
    # bullet movement
    if bullety <= 0:
        bullety = 480
        bullet_state = "ready"

    if bullet_state == "fire":
        fire_bullet(bulletx, bullety)
        bullety -= bullety_change

    # updating the positions
    player(playerx, playery)
    show_score(text_x, text_y)

    pygame.display.update()
