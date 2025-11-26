# Desafio-Tetris-Stack-Tema-3🎮 Desafio Tetris Stack – Tema 1

Este projeto apresenta uma implementação completa de um Tetris minimalista, desenvolvido em Python utilizando a biblioteca Pygame.
O objetivo do trabalho é demonstrar lógica de jogos, manipulação de matrizes, colisões, controle de eventos e atualização de tela em tempo real — elementos essenciais no desenvolvimento de jogos 2D.

🧠 Descrição Técnica Detalhada
1. Estrutura do jogo

O jogo utiliza uma grade de 10 colunas × 20 linhas, onde cada posição representa um bloco.
Cada peça é uma matriz contendo:

1 → bloco preenchido

0 → espaço vazio

As peças seguem o padrão clássico do Tetris:

I, O, T, L, J, S, Z

Cada uma com cor distinta

Rotação por transposição da matriz

2. Mecânica de queda e colisão

O jogo utiliza um temporizador baseado em pygame.time.Clock, que controla:

Velocidade da queda da peça

Atualização da grade

Suavidade do movimento

A função valid_position() garante que:

A peça não ultrapasse bordas

Não colida com outras peças fixadas

Não ultrapasse o chão

3. Fixação de peças e limpeza de linhas

Quando uma peça não pode mais descer:

Ela é gravada na matriz do grid

As linhas completas são removidas

Linhas removidas são repostas no topo

Uma nova peça é gerada

O sistema de clear funciona com:

clear_rows(grid)

4. Rotação

A rotação é feita com:

list(zip(*shape[::-1]))


Que realiza:

Reverse vertical

Transposição

Atualização da matriz

5. Controles

← Move a peça para esquerda

→ Move a peça para direita

↓ Acelera a queda

↑ Rotaciona a peça

🚀 Como rodar o projeto
1. Instalar dependência
pip install pygame

2. Rodar o jogo
python main.py

📁 Estrutura do projeto
tetris_stack/
│── main.py
│── README.md
import pygame
import random

pygame.init()

# Game settings
WIDTH, HEIGHT = 300, 600
BLOCK_SIZE = 30
COLS, ROWS = WIDTH // BLOCK_SIZE, HEIGHT // BLOCK_SIZE

screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Desafio Tetris Stack – Tema 1")

# Shapes
SHAPES = [
    [[1, 1, 1, 1]],  # I
    [[1, 1], [1, 1]],  # O
    [[0, 1, 0], [1, 1, 1]],  # T
    [[1, 0, 0], [1, 1, 1]],  # J
    [[0, 0, 1], [1, 1, 1]],  # L
    [[1, 1, 0], [0, 1, 1]],  # S
    [[0, 1, 1], [1, 1, 0]]   # Z
]

COLORS = [
    (0, 255, 255),
    (255, 255, 0),
    (128, 0, 128),
    (0, 0, 255),
    (255, 140, 0),
    (0, 255, 0),
    (255, 0, 0)
]

class Piece:
    def __init__(self, x, y, shape):
        self.x = x
        self.y = y
        self.shape = shape
        self.color = COLORS[SHAPES.index(shape)]
        self.rotation = 0

def rotate_shape(shape):
    return list(zip(*shape[::-1]))

def valid_position(grid, shape, offset):
    off_x, off_y = offset
    for y, row in enumerate(shape):
        for x, cell in enumerate(row):
            if cell:
                if x + off_x < 0 or x + off_x >= COLS or y + off_y >= ROWS:
                    return False
                if grid[y + off_y][x + off_x] != (0,0,0):
                    return False
    return True

def clear_rows(grid):
    global ROWS, COLS
    new_grid = [row for row in grid if any(c == (0,0,0) for c in row) is False]
    cleared = ROWS - len(new_grid)
    for _ in range(cleared):
        new_grid.insert(0, [(0,0,0)] * COLS)
    return new_grid, cleared

def draw_grid(grid):
    for y in range(ROWS):
        for x in range(COLS):
            pygame.draw.rect(screen, grid[y][x], (x*BLOCK_SIZE, y*BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE))
    for x in range(COLS):
        pygame.draw.line(screen, (50,50,50), (x*BLOCK_SIZE, 0), (x*BLOCK_SIZE, HEIGHT))
    for y in range(ROWS):
        pygame.draw.line(screen, (50,50,50), (0, y*BLOCK_SIZE), (WIDTH, y*BLOCK_SIZE))

def main():
    grid = [[(0,0,0)] * COLS for _ in range(ROWS)]
    clock = pygame.time.Clock()

    current_piece = Piece(COLS//2 - 2, 0, random.choice(SHAPES))
    fall_time = 0
    fall_speed = 500
    running = True

    while running:
        dt = clock.tick()
        fall_time += dt

        if fall_time >= fall_speed:
            fall_time = 0
            if valid_position(grid, current_piece.shape, (current_piece.x, current_piece.y+1)):
                current_piece.y += 1
            else:
                for y, row in enumerate(current_piece.shape):
                    for x, cell in enumerate(row):
                        if cell:
                            grid[current_piece.y+y][current_piece.x+x] = current_piece.color
                grid, _ = clear_rows(grid)
                current_piece = Piece(COLS//2 - 2, 0, random.choice(SHAPES))
                if not valid_position(grid, current_piece.shape, (current_piece.x, current_piece.y)):
                    running = False

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_LEFT and valid_position(grid, current_piece.shape, (current_piece.x-1, current_piece.y)):
                    current_piece.x -= 1
                if event.key == pygame.K_RIGHT and valid_position(grid, current_piece.shape, (current_piece.x+1, current_piece.y)):
                    current_piece.x += 1
                if event.key == pygame.K_DOWN and valid_position(grid, current_piece.shape, (current_piece.x, current_piece.y+1)):
                    current_piece.y += 1
                if event.key == pygame.K_UP:
                    rotated = rotate_shape(current_piece.shape)
                    if valid_position(grid, rotated, (current_piece.x, current_piece.y)):
                        current_piece.shape = rotated

        screen.fill((0,0,0))
        draw_grid(grid)

        for y, row in enumerate(current_piece.shape):
            for x, cell in enumerate(row):
                if cell:
                    pygame.draw.rect(screen, current_piece.color,
                                     ((current_piece.x+x)*BLOCK_SIZE, (current_piece.y+y)*BLOCK_SIZE,
                                      BLOCK_SIZE, BLOCK_SIZE))

        pygame.display.update()

main()
pygame.quit()


✨ Autor

Projeto desenvolvido para finalidade acadêmica.
