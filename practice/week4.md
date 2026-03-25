import pygame
import math

pygame.init()
screen = pygame.display.set_mode((800, 600))
clock = pygame.time.Clock()
font = pygame.font.SysFont("arial", 24, bold=True)

# 1. 로켓 이미지 로드 및 설정
size = 100
try:
    original_rocket = pygame.image.load("rocket.png").convert_alpha()
    original_rocket = pygame.transform.scale(original_rocket, (size, size))
except:
    original_rocket = pygame.Surface((size, size), pygame.SRCALPHA)
    pygame.draw.polygon(original_rocket, (255, 0, 0), [(50, 0), (20, 80), (80, 80)])
    pygame.draw.rect(original_rocket, (200, 200, 200), (40, 80, 20, 20))

# --- SAT(OBB) 충돌 함수 ---
def get_axes(corners):
    axes = []
    for i in range(len(corners)):
        p1, p2 = corners[i], corners[(i + 1) % len(corners)]
        edge = p2 - p1
        axes.append(pygame.Vector2(-edge.y, edge.x).normalize())
    return axes

def is_obb_colliding(corners1, corners2):
    axes = get_axes(corners1) + get_axes(corners2)
    for axis in axes:
        proj1 = [axis.dot(c) for c in corners1]
        proj2 = [axis.dot(c) for c in corners2]
        if min(proj1) > max(proj2) or min(proj2) > max(proj1):
            return False
    return True

# 오브젝트 설정
static_pos = pygame.Vector2(400, 300)
player_rect = pygame.Rect(100, 100, size, size)
angle = 0
is_rotating = True

running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT: running = False
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_s: is_rotating = not is_rotating

    # 이동 및 회전 로직
    keys = pygame.key.get_pressed()
    if keys[pygame.K_LEFT]:  player_rect.x -= 5
    if keys[pygame.K_RIGHT]: player_rect.x += 5
    if keys[pygame.K_UP]:    player_rect.y -= 5
    if keys[pygame.K_DOWN]:  player_rect.y += 5
    if is_rotating: angle += 10 if keys[pygame.K_z] else 2

    # OBB 데이터 계산
    half = size / 2
    base = [pygame.Vector2(-half, -half), pygame.Vector2(half, -half), 
            pygame.Vector2(half, half), pygame.Vector2(-half, half)]
    static_corners = [c.rotate(-angle) + static_pos for c in base]
    player_corners = [c + pygame.Vector2(player_rect.center) for c in base]

    # 로켓 이미지 회전 및 AABB 구하기
    rotated_rocket = pygame.transform.rotate(original_rocket, angle)
    static_aabb = rotated_rocket.get_rect(center=static_pos)

    # 2. OBB 충돌 판정 (가장 정밀함)
    hit_obb = is_obb_colliding(static_corners, player_corners)

    # 그리기
    screen.fill((255, 255, 255))
    
    # 3. 충돌 시 로켓 위치에 노란색 강조 표시 추가
    if hit_obb:
        # 회전된 모양에 맞춰 노란색 실루엣 그리기 (또는 배경색 변경)
        pygame.draw.polygon(screen, (255, 255, 0), static_corners) 
    
    # 플레이어 (회색 사각형)
    pygame.draw.rect(screen, (200, 200, 200), player_rect)

    # 가이드라인 시각화
    pygame.draw.rect(screen, (255, 0, 0), static_aabb, 2)           # AABB
    pygame.draw.polygon(screen, (0, 180, 0), static_corners, 2)    # OBB

    # UI 텍스트
    msg = f"OBB Collision: {'HIT (YELLOW)' if hit_obb else '---'}"
    txt_img = font.render(msg, True, (0, 150, 0))
    screen.blit(txt_img, (20, 20))

    pygame.display.flip()
    clock.tick(60)

pygame.quit()

