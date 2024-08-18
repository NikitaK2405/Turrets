import pygame
import random
import math

# Инициализация Pygame
pygame.init()

# Константы
SCREEN_WIDTH = 500
SCREEN_HEIGHT = 1000
TURRET_SIZE = 30
CUBE_SIZE = 60
BULLET_SIZE = 3
FPS = 60
MAX_REFLECTIONS = 10
GRID_COLS = 6
GRID_ROWS = 12  # Увеличено количество строк
INITIAL_CUBES = 18
INITIAL_FIRE_RATE = 30
INITIAL_CUBE_SPEED = 0.5
PULSE_DURATION = 10
TURRET_Y = SCREEN_HEIGHT - 200  # Поднята на 100 пикселей
LINE_Y = TURRET_Y + 50
SLOT_SIZE = 50
BUTTON_SIZE = 50

# Цвета
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
GREEN = (0, 255, 0)
DARK_BLUE = (0, 0, 139)
LIGHT_GRAY = (200, 200, 200)
BACKGROUND = (100, 180, 220)

# Создание экрана
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Turret Defense")


# Классы
class Turret:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.angle = 0
        self.cooldown = 0
        self.fire_rate = INITIAL_FIRE_RATE
        self.active = True

    def draw(self):
        rotated_turret = pygame.Surface((TURRET_SIZE, TURRET_SIZE), pygame.SRCALPHA)
        pygame.draw.rect(rotated_turret, GREEN if self.active else LIGHT_GRAY, (0, 0, TURRET_SIZE, TURRET_SIZE),
                         border_radius=5)
        pygame.draw.rect(rotated_turret, BLACK, (TURRET_SIZE // 2 - 3, 0, 6, TURRET_SIZE // 2), 2)
        rotated_turret = pygame.transform.rotate(rotated_turret, math.degrees(-self.angle) - 90)
        screen.blit(rotated_turret,
                    (self.x - rotated_turret.get_width() // 2, self.y - rotated_turret.get_height() // 2))

    def aim(self, cubes):
        if cubes and self.active:
            nearest_cube = min(cubes, key=lambda c: math.hypot(c.x + CUBE_SIZE / 2 - self.x, c.y + CUBE_SIZE - self.y))
            target_x = nearest_cube.x + CUBE_SIZE / 2
            target_y = nearest_cube.y + CUBE_SIZE
            self.angle = math.atan2(target_y - self.y, target_x - self.x)

    def shoot(self, cubes):
        if cubes and self.cooldown == 0 and self.active:
            self.cooldown = self.fire_rate
            return Bullet(self.x, self.y, self.angle)
        return None

    def update(self):
        if self.cooldown > 0:
            self.cooldown -= 1


class Cube:
    def __init__(self, x, y, speed):
        self.x = x
        self.y = y
        self.value = random.randint(5, 10)
        self.pulse = 0
        self.speed = speed

    def draw(self):
        pulse_size = max(0, math.sin(self.pulse * math.pi / PULSE_DURATION) * 5) if self.pulse > 0 else 0
        pygame.draw.rect(screen, RED, (self.x - pulse_size / 2, self.y - pulse_size / 2,
                                       CUBE_SIZE + pulse_size, CUBE_SIZE + pulse_size), border_radius=5)
        font = pygame.font.Font(None, 24)
        text = font.render(str(self.value), True, BLACK)
        text_rect = text.get_rect(center=(self.x + CUBE_SIZE // 2, self.y + CUBE_SIZE // 2))
        screen.blit(text, text_rect)

    def move(self):
        self.y += self.speed
        if self.pulse > 0:
            self.pulse -= 1

    def hit(self):
        self.pulse = PULSE_DURATION


class Bullet:
    def __init__(self, x, y, angle):
        self.x = x
        self.y = y
        self.angle = angle
        self.speed = 10
        self.reflections = 0

    def move(self):
        self.x += math.cos(self.angle) * self.speed
        self.y += math.sin(self.angle) * self.speed

    def draw(self):
        pygame.draw.circle(screen, DARK_BLUE, (int(self.x), int(self.y)), BULLET_SIZE)

    def reflect(self, normal_x, normal_y):
        dot_product = (math.cos(self.angle) * normal_x + math.sin(self.angle) * normal_y) * 2
        self.angle = math.atan2(
            math.sin(self.angle) - dot_product * normal_y,
            math.cos(self.angle) - dot_product * normal_x
        )
        self.reflections += 1


class Slot:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.turret = None

    def draw(self):
        pygame.draw.rect(screen, LIGHT_GRAY, (self.x, self.y, SLOT_SIZE, SLOT_SIZE), border_radius=10)
        if self.turret:
            self.turret.draw()


class Button:
    def __init__(self, x, y, action, icon):
        self.x = x
        self.y = y
        self.action = action
        self.icon = icon

    def draw(self):
        pygame.draw.rect(screen, WHITE, (self.x, self.y, BUTTON_SIZE, BUTTON_SIZE), border_radius=10)
        pygame.draw.rect(screen, BLACK, (self.x, self.y, BUTTON_SIZE, BUTTON_SIZE), 2, border_radius=10)
        screen.blit(self.icon, (self.x + 5, self.y + 5))


# Функция для создания новой волны кубиков
def create_cube_wave(num_cubes, cube_speed):
    cubes = []
    cell_width = SCREEN_WIDTH // GRID_COLS
    cell_height = (LINE_Y - CUBE_SIZE) // GRID_ROWS
    all_positions = [(col, row) for col in range(GRID_COLS) for row in range(GRID_ROWS)]
    random.shuffle(all_positions)
    for i in range(min(num_cubes, len(all_positions))):
        col, row = all_positions[i]
        x = col * cell_width + (cell_width - CUBE_SIZE) // 2
        y = row * cell_height - GRID_ROWS * cell_height
        cubes.append(Cube(x, y, cube_speed))
    return cubes


# Создание слотов и кнопок
slots = [Slot(10 + i * (SLOT_SIZE + 10), SCREEN_HEIGHT - 120) for i in range(4)] + \
        [Slot(10 + i * (SLOT_SIZE + 10), SCREEN_HEIGHT - 60) for i in range(4)]

# Создание иконок для кнопок
new_turret_icon = pygame.Surface((40, 40))
pygame.draw.rect(new_turret_icon, GREEN, (10, 10, 20, 20))
manual_icon = pygame.Surface((40, 40))
pygame.draw.circle(manual_icon, BLACK, (20, 20), 15, 2)
auto_merge_icon = pygame.Surface((40, 40))
pygame.draw.polygon(auto_merge_icon, BLACK, [(10, 30), (20, 10), (30, 30)])
shop_icon = pygame.Surface((40, 40))
pygame.draw.rect(shop_icon, BLACK, (10, 20, 20, 15))
pygame.draw.polygon(shop_icon, BLACK, [(15, 20), (25, 20), (20, 10)])

buttons = [
    Button(SCREEN_WIDTH - 120, SCREEN_HEIGHT - 120, "new_turret", new_turret_icon),
    Button(SCREEN_WIDTH - 60, SCREEN_HEIGHT - 120, "manual", manual_icon),
    Button(SCREEN_WIDTH - 120, SCREEN_HEIGHT - 60, "auto_merge", auto_merge_icon),
    Button(SCREEN_WIDTH - 60, SCREEN_HEIGHT - 60, "shop", shop_icon)
]

# Игровые переменные
turret = Turret(SCREEN_WIDTH // 2, TURRET_Y)
level = 1
cubes = create_cube_wave(INITIAL_CUBES, INITIAL_CUBE_SPEED)
bullets = []
game_money = 0
manual_mode = False

# Игровой цикл
running = True
clock = pygame.time.Clock()

while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        elif event.type == pygame.MOUSEBUTTONDOWN:
            for button in buttons:
                if button.x < event.pos[0] < button.x + BUTTON_SIZE and button.y < event.pos[
                    1] < button.y + BUTTON_SIZE:
                    if button.action == "new_turret":
                        for slot in slots:
                            if not slot.turret:
                                slot.turret = Turret(slot.x + SLOT_SIZE // 2, slot.y + SLOT_SIZE // 2)
                                slot.turret.active = False
                                break
                    elif button.action == "manual":
                        manual_mode = not manual_mode
                    elif button.action == "auto_merge":
                        if game_money >= 1500:
                            game_money -= 1500
                            # Implement auto-merge logic here
                    elif button.action == "shop":
                        # Implement shop logic here
                        pass

    # Обновление игровой логики
    if not manual_mode:
        turret.aim(cubes)
    else:
        mouse_x, mouse_y = pygame.mouse.get_pos()
        turret.angle = math.atan2(mouse_y - turret.y, mouse_x - turret.x)

    turret.update()
    new_bullet = turret.shoot(cubes)
    if new_bullet:
        bullets.append(new_bullet)

    for bullet in bullets[:]:
        bullet.move()

        # Отражение от боковых стен
        if bullet.x <= 0 or bullet.x >= SCREEN_WIDTH:
            bullet.reflect(1, 0)

        # Отражение от кубиков и проверка попаданий
        for cube in cubes[:]:
            if (cube.x < bullet.x < cube.x + CUBE_SIZE and
                    cube.y < bullet.y < cube.y + CUBE_SIZE):
                dx = min(bullet.x - cube.x, cube.x + CUBE_SIZE - bullet.x)
                dy = min(bullet.y - cube.y, cube.y + CUBE_SIZE - bullet.y)
                if dx < dy:
                    bullet.reflect(1, 0)
                else:
                    bullet.reflect(0, 1)

                cube.value -= 1
                cube.hit()  # Запуск пульсации
                if cube.value <= 0:
                    cubes.remove(cube)
                    game_money += 10
                break

        if (bullet.y > SCREEN_HEIGHT or bullet.reflections >= MAX_REFLECTIONS):
            bullets.remove(bullet)

    for cube in cubes[:]:
        cube.move()
        if cube.y + CUBE_SIZE >= LINE_Y:
            # Игра окончена
            level = max(1, level - 1) # Уменьшение уровня, если игра окончена
            turret.fire_rate = max(5, INITIAL_FIRE_RATE - level)
            cubes = create_cube_wave(INITIAL_CUBES, INITIAL_CUBE_SPEED)
            bullets.clear()
            break

    # Создание новой волны, если все кубики уничтожены
    if not cubes:
        level += 1
        turret.fire_rate = max(5, INITIAL_FIRE_RATE - level)  # Увеличение скорости стрельбы
        cube_speed = INITIAL_CUBE_SPEED + level * 0.1  # Увеличение скорости кубиков
        cubes = create_cube_wave(INITIAL_CUBES + level - 1, cube_speed)

    # Отрисовка
    screen.fill(BACKGROUND)

    # Рисуем линию
    pygame.draw.line(screen, BLACK, (0, LINE_Y), (SCREEN_WIDTH, LINE_Y), 2)

    for cube in cubes:
        cube.draw()

    for bullet in bullets:
        bullet.draw()

    turret.draw()

    for slot in slots:
        slot.draw()

    for button in buttons:
        button.draw()

    # Отображение уровня и денег
    font = pygame.font.Font(None, 36)
    level_text = font.render(f"Level: {level}", True, BLACK)
    money_text = font.render(f"Money: {game_money}", True, BLACK)
    screen.blit(level_text, (10, 10))
    screen.blit(money_text, (10, 50))

    pygame.display.flip()
    clock.tick(FPS)

pygame.quit()
