# Parcial 1
import pygame
import random
from tkinter import Tk, messagebox

# Inicializar pygame
pygame.init()

# Configuración de la pantalla
TAMANO_CELDA = 30
FILAS, COLUMNAS = 10, 10
ANCHO, ALTO = COLUMNAS * TAMANO_CELDA, FILAS * TAMANO_CELDA + 50  # Espacio extra para la puntuación
pantalla = pygame.display.set_mode((ANCHO, ALTO))
pygame.display.set_caption("Pac-Man")

# Definición de colores
NEGRO = (0, 0, 0)
BLANCO = (255, 255, 255)
ROJO = (255, 0, 0)
AMARILLO = (255, 255, 0)
AZUL = (0, 0, 255)

# Fuente para mostrar la puntuación
fuente = pygame.font.Font(None, 36)

def generar_laberinto():
    """
    Genera un laberinto aleatorio con paredes, Pac-Man, fantasmas y cocos.
    """
    laberinto = [[0 for _ in range(COLUMNAS)] for _ in range(FILAS)]  # Inicializa el laberinto con celdas vacías
    
    # Generar paredes aleatoriamente
    for i in range(FILAS):
        for j in range(COLUMNAS):
            if random.random() < 0.2:  # 20% de probabilidad de ser pared
                laberinto[i][j] = 1  
    
    # Posicionar a Pac-Man y los fantasmas
    laberinto[1][1] = 3  # Pac-Man
    laberinto[FILAS-2][COLUMNAS-2] = 4  # Fantasma 1
    laberinto[FILAS-2][1] = 4  # Fantasma 2
    
    # Colocar cocos en las celdas vacías
    for i in range(FILAS):
        for j in range(COLUMNAS):
            if laberinto[i][j] == 0 and random.random() < 0.5:
                laberinto[i][j] = 2  # Coco
    
    return laberinto

laberinto_matriz = generar_laberinto()
puntaje = 0

def encontrar_posiciones():
    """
    Encuentra la posición inicial de Pac-Man y los fantasmas en el laberinto.
    """
    global posicion_jugador, posiciones_fantasmas, contenido_fantasmas
    posiciones_fantasmas = []
    contenido_fantasmas = {}
    for i in range(FILAS):
        for j in range(COLUMNAS):
            if laberinto_matriz[i][j] == 3:
                posicion_jugador = (i, j)
            elif laberinto_matriz[i][j] == 4:
                posiciones_fantasmas.append((i, j))
                contenido_fantasmas[(i, j)] = 0  # Guardamos el contenido original de la celda

encontrar_posiciones()

def dibujar_laberinto():
    """
    Dibuja el laberinto en la pantalla.
    """
    pantalla.fill(NEGRO)
    for i in range(FILAS):
        for j in range(COLUMNAS):
            x, y = j * TAMANO_CELDA, i * TAMANO_CELDA
            if laberinto_matriz[i][j] == 1:
                pygame.draw.rect(pantalla, BLANCO, (x, y, TAMANO_CELDA, TAMANO_CELDA))
            elif laberinto_matriz[i][j] == 3:
                pygame.draw.circle(pantalla, AMARILLO, (x + TAMANO_CELDA // 2, y + TAMANO_CELDA // 2), TAMANO_CELDA // 2)
            elif laberinto_matriz[i][j] == 4:
                pygame.draw.circle(pantalla, ROJO, (x + TAMANO_CELDA // 2, y + TAMANO_CELDA // 2), TAMANO_CELDA // 2)
            elif laberinto_matriz[i][j] == 2:
                pygame.draw.circle(pantalla, AZUL, (x + TAMANO_CELDA // 2, y + TAMANO_CELDA // 2), TAMANO_CELDA // 6)
    
    # Mostrar el puntaje
    texto_puntaje = fuente.render(f'Puntaje: {puntaje}', True, BLANCO)
    pantalla.blit(texto_puntaje, (10, FILAS * TAMANO_CELDA + 10))
    pygame.display.flip()

def mover_jugador(dx, dy):
    """
    Mueve a Pac-Man en la dirección indicada (dx, dy).
    """
    global posicion_jugador, puntaje
    x, y = posicion_jugador
    nueva_x, nueva_y = x + dx, y + dy
    
    if 0 <= nueva_x < FILAS and 0 <= nueva_y < COLUMNAS:
        if laberinto_matriz[nueva_x][nueva_y] != 1:  # No puede moverse a una pared
            if laberinto_matriz[nueva_x][nueva_y] == 2:  # Si hay un coco, suma puntos
                puntaje += 10
            elif laberinto_matriz[nueva_x][nueva_y] == 4:  # Si toca un fantasma, pierde
                game_over()
                return
            
            # Actualizar posición de Pac-Man
            laberinto_matriz[x][y] = 0
            laberinto_matriz[nueva_x][nueva_y] = 3
            posicion_jugador = (nueva_x, nueva_y)
            verificar_victoria()

def mover_fantasmas():
    """
    Mueve los fantasmas aleatoriamente.
    """
    global posiciones_fantasmas, contenido_fantasmas
    nuevas_posiciones = {}

    for x, y in posiciones_fantasmas:
        opciones = [(x-1, y), (x+1, y), (x, y-1), (x, y+1)]
        random.shuffle(opciones)

        for nueva_x, nueva_y in opciones:
            if 0 <= nueva_x < FILAS and 0 <= nueva_y < COLUMNAS and laberinto_matriz[nueva_x][nueva_y] not in [1, 4]:
                if laberinto_matriz[nueva_x][nueva_y] == 3:
                    game_over()

                laberinto_matriz[x][y] = contenido_fantasmas.get((x, y), 0)
                contenido_fantasmas[(nueva_x, nueva_y)] = laberinto_matriz[nueva_x][nueva_y]
                laberinto_matriz[nueva_x][nueva_y] = 4
                nuevas_posiciones[(nueva_x, nueva_y)] = contenido_fantasmas[(nueva_x, nueva_y)]
                break
        else:
            nuevas_posiciones[(x, y)] = contenido_fantasmas.get((x, y), 0)

    posiciones_fantasmas = list(nuevas_posiciones.keys())
    contenido_fantasmas = nuevas_posiciones

def verificar_victoria():
    """
    Comprueba si todos los cocos han sido comidos.
    """
    if all(2 not in fila for fila in laberinto_matriz):
        victoria()

def game_over():
    """
    Muestra un mensaje de "Game Over" y permite reiniciar el juego.
    """
    Tk().withdraw()
    if messagebox.askyesno("Game Over", "¿Quieres jugar de nuevo?"):
        reiniciar_juego()
    else:
        pygame.quit()
        exit()

def victoria():
    """
    Muestra un mensaje de victoria y permite reiniciar el juego.
    """
    Tk().withdraw()
    if messagebox.askyesno("¡Ganaste!", "¿Jugar de nuevo?"):
        reiniciar_juego()
    else:
        pygame.quit()
        exit()

def reiniciar_juego():
    """
    Reinicia el laberinto y el puntaje.
    """
    global laberinto_matriz, puntaje
    puntaje = 0
    laberinto_matriz = generar_laberinto()
    encontrar_posiciones()

clock = pygame.time.Clock()
tick_fantasmas = 0
ejecutando = True

# Bucle principal del juego
while ejecutando:
    for evento in pygame.event.get():
        if evento.type == pygame.QUIT:
            ejecutando = False
        elif evento.type == pygame.KEYDOWN:
            if evento.key == pygame.K_w:
                mover_jugador(-1, 0)
            elif evento.key == pygame.K_s:
                mover_jugador(1, 0)
            elif evento.key == pygame.K_a:
                mover_jugador(0, -1)
            elif evento.key == pygame.K_d:
                mover_jugador(0, 1)
    
    if tick_fantasmas % 5 == 0:
        mover_fantasmas()
    
    dibujar_laberinto()
    tick_fantasmas += 1
    clock.tick(10)

pygame.quit()


