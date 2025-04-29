# Catchthe-ghost

import pygame
import random
import sys

# تهيئة Pygame
pygame.init()

# إعدادات الشاشة
WIDTH, HEIGHT = 400, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption('Flappy Bird')

# الألوان
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)

# إعدادات الطائر
bird_width, bird_height = 40, 40
bird_x, bird_y = 50, HEIGHT // 2
bird_velocity = 0
gravity = 0.5
jump_strength = -10

# إعدادات الأنابيب
pipe_width = 60
pipe_gap = 150
pipe_velocity = 3
pipes = []

# النقاط
score = 0
font = pygame.font.SysFont('Arial', 30)

# إعداد الصوت
jump_sound = pygame.mixer.Sound('jump.wav')
hit_sound = pygame.mixer.Sound('hit.wav')

def draw_bird(x, y):
    pygame.draw.rect(screen, BLUE, (x, y, bird_width, bird_height))

def draw_pipes(pipes):
    for pipe in pipes:
        pygame.draw.rect(screen, GREEN, pipe)

def move_pipes(pipes):
    global score
    for pipe in pipes:
        pipe.x -= pipe_velocity
    if pipes and pipes[0].x + pipe_width < 0:
        pipes.pop(0)
        score += 1

def create_pipe():
    height = random.randint(150, 450)
    top_rect = pygame.Rect(WIDTH, 0, pipe_width, height)
    bottom_rect = pygame.Rect(WIDTH, height + pipe_gap, pipe_width, HEIGHT - height - pipe_gap)
    return top_rect, bottom_rect

def check_collision(bird_rect, pipes):
    if bird_rect.top <= 0 or bird_rect.bottom >= HEIGHT:
        return True
    for pipe in pipes:
        if bird_rect.colliderect(pipe):
            return True
    return False

def display_score(score):
    score_text = font.render(f'Score: {score}', True, WHITE)
    screen.blit(score_text, (10, 10))

# الدورة الرئيسية للعبة
clock = pygame.time.Clock()

while True:
    screen.fill(BLACK)

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE:  # عندما يضغط اللاعب على المسافة
                bird_velocity = jump_strength
                jump_sound.play()

    # تحديث حركة الطائر
    bird_velocity += gravity
    bird_y += bird_velocity

    # تحديث الأنابيب
    if len(pipes) == 0 or pipes[-1][0].x < WIDTH - 300:
        pipes.extend(create_pipe())

    move_pipes(pipes)

    # رسم الطائر
    bird_rect = pygame.Rect(bird_x, bird_y, bird_width, bird_height)
    draw_bird(bird_x, bird_y)

    # رسم الأنابيب
    draw_pipes([pipe[0] for pipe in pipes] + [pipe[1] for pipe in pipes])

    # عرض النقاط
    display_score(score)

    # التحقق من الاصطدام
    if check_collision(bird_rect, [pipe[0] for pipe in pipes] + [pipe[1] for pipe in pipes]):
        hit_sound.play()
        pygame.time.delay(500)
        score = 0
        bird_y = HEIGHT // 2
        bird_velocity = 0
        pipes.clear()

    # تحديث الشاشة
    pygame.display.update()

    # تحديد معدل التحديث
    clock.tick(60)
