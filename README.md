# Proyecto-Integrador

Este script utiliza la API de Python de Blender (bpy) junto con cálculos trigonométricos (math) y manipulación de geometría de bajo nivel (bmesh) para construir un escenario 3D dinámico. A continuación, se detalla la lógica paso a paso basada en los comentarios del código fuente.

 # Crear material
Esta sección define una función modular para la creación y gestión de texturas.

Lógica: La función crear_material(nombre, color_rgb) busca si un material con ese nombre ya existe en la base de datos de Blender para evitar duplicados. Si no existe, lo crea y activa use_nodes = True, lo cual es una buena práctica para asegurar la compatibilidad con motores de renderizado modernos como Cycles o Eevee.

Color: Recibe una tupla RGB y la desempaqueta añadiendo 1.0 al final, correspondiente al canal Alfa (opacidad total).

 # Generar Pasillo y Animación
Esta es la función principal que orquesta toda la creación del escenario.

# Limpiar escena: Ejecuta un barrido inicial seleccionando y eliminando todos los objetos previos (bpy.ops.object.delete()), garantizando un lienzo en blanco cada vez que se ejecuta el script.

Variables de control: Define parámetros clave del entorno: la cantidad de segmentos rectos y curvos, el ancho del pasillo, la longitud de cada paso y el ángulo de giro en radianes (math.radians(6)).

Listas de coordenadas: Inicializa las listas puntos_izq, puntos_der y puntos_centro que almacenarán los vectores XYZ de la geometría.

 # 1. Calcular puntos
Es el núcleo matemático del generador procedural. Iterando a través del total de segmentos:

Trayectoria: Comprueba si la iteración actual ha superado los segmentos_rectos. Si es así, comienza a sumar el angulo_giro a la rotación, creando la curvatura del pasillo.

Trigonometría: Utiliza math.sin() y math.cos() multiplicados por el tamaño del paso para calcular el avance en los ejes X e Y, guardando esta ruta central (elevada a 1.5 en Z) en puntos_centro.

Vectores Perpendiculares: Para calcular las paredes, suma 90 grados (π/2 radianes) al ángulo actual de rotación. Esto permite proyectar vértices paralelos hacia la izquierda y la derecha de la trayectoria central, independientemente de hacia dónde esté girando el pasillo.

 # 2. Crear Suelo
Abandona las "primitivas" (como los cubos básicos) y utiliza bmesh para construir geometría desde cero.

Lógica: Crea una malla vacía y la vincula a la escena. Luego, toma la lista de puntos izquierdos y la concatena con la lista de puntos derechos invertida (reversed()). Esto genera un contorno cerrado perfecto sin que los vértices se crucen en forma de moño. A partir de esos vértices, genera una única gran cara poligonal que sirve de suelo.

 # 3. Función para paredes (CORREGIDA)
Define una sub-función interna llamada crear_pared_alternada que se ejecutará dos veces (una para la izquierda y otra para la derecha).

Gestión de Materiales: Al objeto recién creado se le añaden los dos materiales. El orden es vital: el material oscuro toma el Índice 0 y el naranja el Índice 1.

Extrusión Matemática: Toma los puntos 2D calculados en el paso 1 y genera dos conjuntos de vértices: v_inf (en el suelo, Z=0) y v_sup (elevados, Z=3.0).

Generación de Caras y Alternancia: Un bucle une estos vértices en grupos de cuatro para formar polígonos verticales (paredes). Al momento de crear la cara, utiliza el operador módulo (i % 2 == 0) para alternar el material_index entre 0 y 1, creando el patrón visual oscuro/naranja a lo largo de la pared.

 # 4. RECORRIDO DE CÁMARA
Automatiza la creación de una cinemática a través del pasillo generado.

Creación de la Curva: Toma los puntos_centro (que habíamos elevado a la altura de los ojos) y genera un objeto de tipo CURVE (spline polinomial) invisible en el entorno 3D.

Restricción (Constraint): Instancia una cámara y le aplica una restricción de tipo FOLLOW_PATH, definiendo la curva anterior como su objetivo. Se ajustan los ejes (TRACK_NEGATIVE_Z, UP_Y) para que la cámara mire correctamente hacia el frente.

# Animación: Inserta fotogramas clave (keyframes) mediante código. En el frame 1, establece el offset_factor (progreso de la ruta) en su inicio, y en el frame 200 lo lleva a su final, obligando a Blender a interpolar el movimiento y generar un video de recorrido automático de 200 fotogramas.


<img width="1919" height="1022" alt="Captura de pantalla 2026-02-25 215845" src="https://github.com/user-attachments/assets/1f41aab7-1ede-4d50-aaf5-628c52df82d0" />
