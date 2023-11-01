import tkinter as tk
from PIL import Image, ImageTk
import math
import random
import cv2
import mediapipe as mp
import imageio

# Función para calcular la posición de la Luna durante el eclipse
def calcular_posicion_luna(tiempo):
    radio_orbita_luna = 200
    velocidad_angular_luna = 0.01

    # Calcular la posición de la Luna en coordenadas cartesianas
    x_luna = radio_orbita_luna * math.cos(velocidad_angular_luna * tiempo)
    y_luna = radio_orbita_luna * math.sin(velocidad_angular_luna * tiempo)

    return x_luna, y_luna

# Función para crear estrellas en el fondo
def crear_estrellas(canvas):
    num_estrellas = 1000
    for _ in range(num_estrellas):
        # Generar coordenadas aleatorias para las estrellas y dibujarlas
        x = random.randint(0, canvas.winfo_reqwidth())
        y = random.randint(0, canvas.winfo_reqheight())
        canvas.create_rectangle(x, y, x+1, y+1, fill="white", outline="white")

# Función para actualizar el valor del slider en función de la posición de la mano
def actualizar_slider():
    _, frame = cap.read()
    frame = cv2.flip(frame, 1)
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

    # Detección de la mano utilizando MediaPipe
    results = hands.process(frame)

    if results.multi_hand_landmarks:
        hand_landmarks = results.multi_hand_landmarks[0]  # Tomar solo la primera mano detectada
        hand_position_x = hand_landmarks.landmark[mp.solutions.hands.HandLandmark.WRIST].x * frame.shape[1]
        hand_position_x = int(max(0, min(frame.shape[1], hand_position_x)))  # Asegurarse de que esté dentro de los límites de la imagen

        # Actualizar el valor del slider en función de la posición de la mano
        slider_value = int((hand_position_x / frame.shape[1]) * (slider_max - slider_min)) + slider_min
        tiempo_slider.set(slider_value)

    # Mostrar la imagen con la detección de la mano y el valor del slider
    cv2.imshow("Hand Tracking", frame)

    # Llamar a la función nuevamente después de un breve retraso
    ventana.after(10, actualizar_slider)

# Función para actualizar la posición de la Luna y la Tierra
def actualizar_posiciones():
    tiempo = tiempo_slider.get()
    x_luna, y_luna = calcular_posicion_luna(tiempo)

    # Posición fija de la Tierra en la parte superior central del lienzo
    x_tierra, y_tierra = ventana.winfo_reqwidth() / 2 + 250, 250

    # Mover y ajustar el tamaño de la Tierra
    canvas.move(tierra, x_tierra - canvas.coords(tierra)[0], y_tierra - canvas.coords(tierra)[1])
    canvas.scale(tierra, x_tierra, y_tierra, 1.4, 1.4)  # Aumentar a 140%

    # Mover y ajustar el tamaño de la Luna
    canvas.move(luna, x_tierra + x_luna - canvas.coords(luna)[0], y_tierra + y_luna - canvas.coords(luna)[1])
    canvas.scale(luna, x_tierra + x_luna, y_tierra + y_luna, 0.5, 0.5)  # Reducir a la mitad

    # Llamar a la función nuevamente después de 10 milisegundos para animar la simulación
    ventana.after(10, actualizar_posiciones)

# Función para mostrar el GIF correspondiente al valor del slider en el lienzo
def mostrar_gif_en_lienzo(valor_slider):
    valor_slider = int(valor_slider)
    if 276 <= valor_slider <= 299:
        indice_gif = 0
        fotograma = lista_fotogramas_gifs[indice_gif][valor_slider - 276]
    elif 300 <= valor_slider <= 326:
        indice_gif = 1
        fotograma = lista_fotogramas_gifs[indice_gif][valor_slider - 300]
    elif 327 <= valor_slider <= 328:
        indice_gif = 2
        fotograma = lista_fotogramas_gifs[indice_gif][valor_slider - 327]
    elif 329 <= valor_slider <= 355:
        indice_gif = 3
        fotograma = lista_fotogramas_gifs[indice_gif][valor_slider - 329]
    elif 356 <= valor_slider <= 344:
        indice_gif = 4
        fotograma = lista_fotogramas_gifs[indice_gif][valor_slider - 356]
    else:
        return

    if valor_slider == 345:
        canvas.delete("gif")  # Eliminar el GIF anterior si se detiene
    else:
        # Ajustar el tamaño del GIF a 250x250 píxeles
        fotograma = Image.fromarray(fotograma)
        fotograma = fotograma.resize((250, 250), Image.LANCZOS)

        imagen_gif = ImageTk.PhotoImage(fotograma)
        
        canvas.create_image(1250, 250, image=imagen_gif, tag="gif")  # Mover a (100, 300)
        canvas.image = imagen_gif  # Conservar una referencia para evitar que se elimine

# Configuración de la ventana principal
ventana = tk.Tk()
ventana.title("Dark Moon Rising")

# Canvas para dibujar
canvas = tk.Canvas(ventana, width=1500, height=600)
canvas.config(bg="black")
canvas.pack()

# Slider para controlar el tiempo
slider_min = 256
slider_max = 406  # Cambiado para incluir el valor 383
tiempo_slider = tk.Scale(ventana, from_=slider_min, to=slider_max, orient="horizontal", label="Tiempo", command=mostrar_gif_en_lienzo)
tiempo_slider.pack()

# Inicializar el detector de manos de MediaPipe
mp_hands = mp.solutions.hands
hands = mp_hands.Hands()

# Iniciar la captura de video desde la cámara
cap = cv2.VideoCapture(0)

# Iniciar la actualización del slider
actualizar_slider()

# Crear estrellas en el fondo
crear_estrellas(canvas)

# Crear el Sol con la imagen más grande
imagen_sol = Image.open("3 Imagenes Luna Tierra y Sol/sol.png")
imagen_sol = imagen_sol.resize((150, 150), Image.LANCZOS)  # Aumentar a 150x150
imagen_sol_tk = ImageTk.PhotoImage(imagen_sol)
sol = canvas.create_image(ventana.winfo_reqwidth() / 2, 150, image=imagen_sol_tk)

# Crear la Luna con la imagen
imagen_luna = Image.open("3 Imagenes Luna Tierra y Sol/luna.png")
imagen_luna = imagen_luna.resize((45, 45), Image.LANCZOS)  # Reducir a 45x45
imagen_luna_tk = ImageTk.PhotoImage(imagen_luna)
luna = canvas.create_image(ventana.winfo_reqwidth() / 2, 150, image=imagen_luna_tk)

# Crear la Tierra con la imagen más grande
imagen_tierra = Image.open("3 Imagenes Luna Tierra y Sol/tierra.png")
imagen_tierra = imagen_tierra.resize((105, 105), Image.LANCZOS)  # Aumentar a 105x105
imagen_tierra_tk = ImageTk.PhotoImage(imagen_tierra)
tierra = canvas.create_image(ventana.winfo_reqwidth() / 2, 250, image=imagen_tierra_tk)

# Cargar archivos GIF y crear listas de fotogramas
gifs = [
    "2 Gifs/a/1-Eclicpce-Prenumbral_1.gif",
    "2 Gifs/a/2-Eclipce-Umbral_1.gif",
    "2 Gifs/a/4-Eclipce-umbral-saliendo-bien_1.gif",
    "2 Gifs/a/5-Eclicpce-prenumbral-saliendo_Bien_1.gif",
    "2 Gifs/a/3-Eclipce-Total_1.gif"
]

lista_frames_gifs = [imageio.get_reader(gif_file) for gif_file in gifs]
lista_fotogramas_gifs = [list(reader) for reader in lista_frames_gifs]

# Iniciar la simulación de eclipse solar
actualizar_posiciones()

# Iniciar el bucle principal de la ventana
ventana.mainloop()

# Liberar la cámara y cerrar las ventanas de OpenCV cuando se cierre la aplicación
cap.release()
cv2.destroyAllWindows()
