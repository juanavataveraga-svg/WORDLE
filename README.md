
import ipywidgets as widgets
from IPython.display import display, clear_output
import random

# variables
nivel_actual = 1
max_intentos = 6
intentos_realizados = []
palabra_secreta = ""

# diccionario de palabras
palabras_por_nivel = {
    1: ["sol", "pan", "luz", "mar", "pez", "red"],
    2: ["gato", "luna", "flor", "agua", "nube", "tren"],
    3: ["perro", "arbol", "raton", "lapiz", "fuego"],
    4: ["bosque", "camino", "planta", "ciudad", "pintor"],
    5: ["planeta", "estrella", "montana", "tortuga"]
}

# logica principal
def evaluar_palabra(intento, secreta):
    # creamos listas de la longitud de la palabra
    resultado = [None] * len(secreta)
    letras_secretas = list(secreta) # Convertimos la secreta a lista para poder "tachar" letras

    # buscar los VERDES (letra correcta, lugar correcto)
    for i in range(len(intento)):
        if intento[i] == secreta[i]:
            resultado[i] = 'verde'
            letras_secretas[i] = None # Tachamos esta letra para no volver a contarla

    # buscar los AMARILLOS y GRISES
    for i in range(len(intento)):
        # Si no es verde, evaluamos qué es
        if resultado[i] is None:
            if intento[i] in letras_secretas:
                resultado[i] = 'amarillo'
                indice = letras_secretas.index(intento[i])
                letras_secretas[indice] = None
            else:
                resultado[i] = 'gris'

    return resultado

def generar_html_grilla():
    longitud_palabra = len(palabra_secreta)
    html = '<div style="display: flex; flex-direction: column; gap: 5px; margin-top: 10px;">'

    for i in range(max_intentos):
        html += '<div style="display: flex; gap: 5px; justify-content: center;">'

        if i < len(intentos_realizados):
            # dibujar un intento ya realizado
            palabra = intentos_realizados[i]
            colores = evaluar_palabra(palabra, palabra_secreta)

            for j in range(longitud_palabra):
                if colores[j] == 'verde':
                    bg_color = "#538d4e" # VERDE
                elif colores[j] == 'amarillo':
                    bg_color = "#b59f3b" # AMARILLO
                else:
                    bg_color = "#3a3a3c" # GRIS

                caja = f'<div style="width: 50px; height: 50px; background-color: {bg_color}; color: white; display: flex; align-items: center; justify-content: center; font-size: 24px; font-weight: bold; border-radius: 4px;">{palabra[j].upper()}</div>'
                html += caja
        else:
            # casillas vacías
            for j in range(longitud_palabra):
                caja_vacia = f'<div style="width: 50px; height: 50px; background-color: transparent; border: 2px solid #565758; display: flex; align-items: center; justify-content: center; border-radius: 4px;"></div>'
                html += caja_vacia

        html += '</div>'
    html += '</div>'
    return html

# interfaxz gráfica
# textos y mensajes
titulo = widgets.HTML()
mensaje = widgets.HTML()
grilla_visual = widgets.HTML()

# tontroles de entrada
entrada_texto = widgets.Text(placeholder='Escribe tu palabra aquí...')
boton_enviar = widgets.Button(description='Enviar', button_style='primary')
caja_entrada = widgets.HBox([entrada_texto, boton_enviar])

# tontroles de navegación
boton_siguiente = widgets.Button(description='Siguiente Nivel', button_style='success')
boton_reiniciar = widgets.Button(description='Reiniciar Juego', button_style='danger')
caja_navegacion = widgets.HBox([boton_siguiente, boton_reiniciar])

def actualizar_pantalla():
    titulo.value = f"<h2 style='text-align: center; color: #333;'>WORDLE - Nivel {nivel_actual}</h2><p style='text-align: center;'>Adivina la palabra de {len(palabra_secreta)} letras.</p>"
    grilla_visual.value = generar_html_grilla()

def procesar_intento(b=None):
    intento = entrada_texto.value.lower().strip()
    entrada_texto.value = ""

    if len(intento) != len(palabra_secreta):
        mensaje.value = f"<b style='color: orange;'>La palabra debe tener exactamente {len(palabra_secreta)} letras.</b>"
        return

    if not intento.isalpha():
        mensaje.value = "<b style='color: orange;'>Solo se permiten letras.</b>"
        return

    mensaje.value = ""
    intentos_realizados.append(intento)
    actualizar_pantalla()
    verificar_fin_juego(intento)

def verificar_fin_juego(intento):
    if intento == palabra_secreta:
        mensaje.value = "<h3 style='color: green; text-align: center;'>¡Excelente! Has adivinado la palabra.</h3>"
        caja_entrada.layout.display = 'none'
        if nivel_actual < len(palabras_por_nivel):
            boton_siguiente.layout.display = 'block'
        else:
            mensaje.value += "<h3 style='color: blue; text-align: center;'> ¡HAS SUPERADO TODOS LOS NIVELES! </h3>"

    elif len(intentos_realizados) >= max_intentos:
        mensaje.value = f"<h3 style='color: red; text-align: center;'>Fin del juego. La palabra era: {palabra_secreta.upper()}</h3>"
        caja_entrada.layout.display = 'none'

def avanzar_nivel(b):
    global nivel_actual
    if nivel_actual < len(palabras_por_nivel):
        nivel_actual += 1
        iniciar_juego()

def reiniciar_todo(b):
    global nivel_actual
    nivel_actual = 1
    iniciar_juego()

# inicio del juego
def iniciar_juego():
    global palabra_secreta, intentos_realizados

    palabra_secreta = random.choice(palabras_por_nivel[nivel_actual])
    intentos_realizados = []

    caja_entrada.layout.display = 'flex'
    boton_siguiente.layout.display = 'none'
    mensaje.value = ""
    entrada_texto.value = ""

    actualizar_pantalla()

boton_enviar.on_click(procesar_intento)
entrada_texto.on_submit(procesar_intento)
boton_siguiente.on_click(avanzar_nivel)
boton_reiniciar.on_click(reiniciar_todo)

interfaz_principal = widgets.VBox([
    titulo,
    grilla_visual,
    mensaje,
    caja_entrada,
    caja_navegacion
], layout=widgets.Layout(align_items='center'))

# arrancar el juego
iniciar_juego()
display(interfaz_principal)
