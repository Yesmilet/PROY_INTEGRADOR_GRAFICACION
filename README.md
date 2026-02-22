
#  Proyecto Integrador – Unidad 1  
##  Escenario Procedural con Animación de Cámara

Este proyecto consiste en la **generación procedural de un escenario 3D** en Blender, específicamente un **pasillo curvo**, al cual se le añade una **animación de cámara** que recorre automáticamente todo el camino.

La principal característica es que **todo el entorno se crea mediante código**, sin modelar manualmente, lo que demuestra el uso de programación para generar contenido visual.

---

#  1. Creación de Materiales

```python
def crear_material(nombre, color_rgb):
    mat = bpy.data.materials.new(name=nombre)
    mat.use_nodes = True
    bsdf = mat.node_tree.nodes.get("Principled BSDF")
    if bsdf:
        bsdf.inputs["Base Color"].default_value = (*color_rgb, 1.0)
    mat.diffuse_color = (*color_rgb, 1.0)
    return mat
````

###  Explicación:

* Se crea un **material nuevo** con un nombre específico.
* Se activan los nodos (`use_nodes = True`).
* Se usa el shader **Principled BSDF**, que es el estándar en Blender.
* Se asigna un color RGB al material.

###  Materiales usados:

* `ParedOscura` → gris oscuro
* `ParedDetalle` → rojo
* `Suelo` → gris


El código comienza definiendo una función para automatizar la creación de colores. En lugar de crear materiales manualmente en la interfaz de Blender, usa el motor de nodos:

- use_nodes = True: Activa el sistema de nodos para que el material sea compatible con el motor de renderizado (Eevee/Cycles).

- Principled BSDF: Es el nodo estándar de Blender. El script busca este nodo y cambia su Base Color usando valores RGB normalizados (0.0 a 1.0).

---

#  2. Limpieza de la Escena

```python
bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete()
```

###  Explicación:

* Se seleccionan todos los objetos.
* Se eliminan para empezar desde cero.

---

# 3. Parámetros del Escenario

```python
ancho = 3.0
paso = 3.0
total_bloques = 60
altura_pared = 3.0
grosor_pared = 1.0
```

### Explicación:

Estos valores controlan el tamaño del pasillo:

* `ancho` → separación entre paredes
* `paso` → distancia entre segmentos
* `total_bloques` → longitud del pasillo
* `altura_pared` → altura
* `grosor_pared` → grosor

---

#  4. Generación de la Curvatura

##  Función de desplazamiento

```python
def offset_x(i):
    x = 0.0
    if 15 <= i <= 30:
        t = (i - 15) / 15.0
        x += 6.0 * (0.5 - 0.5 * math.cos(t * math.pi))
    elif i > 30:
        x += 6.0
    if 38 <= i <= 53:
        t = (i - 38) / 15.0
        x -= 6.0 * (0.5 - 0.5 * math.cos(t * math.pi))
    elif i > 53:
        x -= 6.0
    return x
```

###  Explicación:

* Calcula el desplazamiento en el eje X.
* Usa una función de coseno para suavizar la curva.

```python
0.5 - 0.5 * math.cos(t * math.pi)
```

###  Resultado:

* El pasillo primero es recto
* Luego se curva suavemente
* Finalmente regresa

---

##  Ángulo de la trayectoria

```python
def angulo_tangente(i):
    dx = offset_x(min(i + 1, total_bloques - 1)) - offset_x(max(i - 1, 0))
    dy = paso * 2
    return math.atan2(dx, dy)
```

###  Explicación:

* Calcula la dirección del camino.
* Permite rotar correctamente paredes y cámara.

---

#  5. Creación de Paredes

```python
bpy.ops.mesh.primitive_cube_add(location=(cx - ancho, cy, altura_pared / 2))
```

###  Proceso:

Para cada bloque:

1. Se calcula posición y rotación
2. Se crean dos paredes (izquierda y derecha)
3. Se ajusta escala, rotación y material

###  Detalle visual:

```python
if i % 2 == 0:
```

* Alterna materiales
* Algunas paredes son más altas

---

#  6. Creación del Suelo Procedural

```python
mesh.from_pydata(verts, [], faces)
```

###  Explicación:

* Se crea una malla continua.
* No se usan planos individuales.

###  Funcionamiento:

```python
faces.append((a, b, c, d))
```

* Se conectan vértices formando caras.

###  Resultado:

* Suelo continuo
* Sin cortes
* Sigue la curva


A diferencia de las paredes, que son objetos separados, el suelo se construye como una sola malla:

- Vértices y Caras: El script calcula dos puntos (izquierda y derecha) por cada paso del pasillo. Luego, conecta estos puntos con los del paso anterior para formar "caras" (cuadriláteros).

- from_pydata: Esta es una función avanzada de la API de Blender que toma una lista de coordenadas y genera la geometría desde cero.
---

#  7. Creación de la Cámara

```python
cam_data = bpy.data.cameras.new("CamaraPasillo")
```

###  Explicación:

* Se crea una cámara nueva.
* Se asigna como cámara principal.

```python
cam_data.lens = 50
```

* Vista natural.

---

#  8. Animación de la Cámara

##  Configuración

```python
fps = 24
duracion_s = 2
total_frames = fps * duracion_s
```

* 24 FPS
* Duración: 2 segundos

---

##  Movimiento

```python
cam_obj.location = (cx, cy, cam_z)
cam_obj.rotation_euler = (math.radians(90), 0, rot)
```

* La cámara sigue el centro del pasillo.
* Altura a nivel de ojos.

---

##  Keyframes

```python
cam_obj.keyframe_insert(data_path="location", frame=frame)
cam_obj.keyframe_insert(data_path="rotation_euler", frame=frame)
```

* Guarda posición y rotación en cada frame.

---

##  Suavizado

```python
kp.interpolation = 'BEZIER'
```

* Movimiento fluido.

---

#  9. Configuración de Render

```python
bpy.context.scene.render.resolution_x = 1280
bpy.context.scene.render.resolution_y = 720
```

* Resolución HD.

---

#  Resultado Final

* ✔ Pasillo generado automáticamente
* ✔ Curvatura suave
* ✔ Suelo continuo
* ✔ Cámara en movimiento
* ✔ Animación fluida

---

#  Conclusión

Este proyecto demuestra:

* Programación procedural
* Uso de matemáticas (coseno, tangente)
* Automatización en Blender
* Animación con keyframes

---
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/5285fde7-e62b-49a9-943a-2e546b7b927a" />


#  Código Completo

```python
import bpy
import math

def crear_material(nombre, color_rgb):
    mat = bpy.data.materials.new(name=nombre)
    mat.use_nodes = True
    bsdf = mat.node_tree.nodes.get("Principled BSDF")
    if bsdf:
        bsdf.inputs["Base Color"].default_value = (*color_rgb, 1.0)
    mat.diffuse_color = (*color_rgb, 1.0)
    return mat

def generar_pasillo_curvo():
    bpy.ops.object.select_all(action='SELECT')
    bpy.ops.object.delete()

    mat_a = crear_material("ParedOscura",  (0.1, 0.1, 0.1))
    mat_b = crear_material("ParedDetalle", (0.8, 0.2, 0.0))
    mat_s = crear_material("Suelo",        (0.15, 0.15, 0.15))

    ancho         = 3.0
    paso          = 3.0
    total_bloques = 60
    altura_pared  = 3.0
    grosor_pared  = 1.0

    def offset_x(i):
        x = 0.0
        if 15 <= i <= 30:
            t = (i - 15) / 15.0
            x += 6.0 * (0.5 - 0.5 * math.cos(t * math.pi))
        elif i > 30:
            x += 6.0
        if 38 <= i <= 53:
            t = (i - 38) / 15.0
            x -= 6.0 * (0.5 - 0.5 * math.cos(t * math.pi))
        elif i > 53:
            x -= 6.0
        return x

    def angulo_tangente(i):
        dx = offset_x(min(i + 1, total_bloques - 1)) - offset_x(max(i - 1, 0))
        dy = paso * 2
        return math.atan2(dx, dy)

    # ── Paredes ────────────────────────────────────────────────────────────
    for i in range(total_bloques):
        cx  = offset_x(i)
        cy  = i * paso
        rot = angulo_tangente(i)
        fill_y = paso / max(math.cos(rot), 0.5)

        if i % 2 == 0:
            mat   = mat_a
            esc_z = 1.0
        else:
            mat   = mat_b
            esc_z = 1.5

        bpy.ops.mesh.primitive_cube_add(location=(cx - ancho, cy, altura_pared / 2))
        p = bpy.context.active_object
        p.scale = (grosor_pared, fill_y / 2 + 0.1, esc_z)
        p.rotation_euler.z = rot
        p.data.materials.append(mat)

        bpy.ops.mesh.primitive_cube_add(location=(cx + ancho, cy, altura_pared / 2))
        p = bpy.context.active_object
        p.scale = (grosor_pared, fill_y / 2 + 0.1, 1.0)
        p.rotation_euler.z = rot
        p.data.materials.append(mat_a)

    # ── Suelo: malla procedural continua ──────────────────────────────────
    verts = []
    faces = []

    for i in range(total_bloques):
        cx  = offset_x(i)
        cy  = i * paso
        rot = angulo_tangente(i)

        tx = math.sin(rot)
        ty = math.cos(rot)
        px = -ty
        py =  tx

        verts.append((cx + px * (-ancho), cy + py * (-ancho), 0))
        verts.append((cx + px * ( ancho), cy + py * ( ancho), 0))

    for i in range(total_bloques - 1):
        a = i * 2
        b = i * 2 + 1
        c = i * 2 + 3
        d = i * 2 + 2
        faces.append((a, b, c, d))

    mesh = bpy.data.meshes.new("SueloCurvo")
    mesh.from_pydata(verts, [], faces)
    mesh.update()
    obj_suelo = bpy.data.objects.new("SueloCurvo", mesh)
    bpy.context.collection.objects.link(obj_suelo)
    obj_suelo.data.materials.append(mat_s)

    # ── Cámara con recorrido por el pasillo ───────────────────────────────
    cam_data = bpy.data.cameras.new("CamaraPasillo")
    cam_data.lens = 50
    cam_obj  = bpy.data.objects.new("CamaraPasillo", cam_data)
    bpy.context.collection.objects.link(cam_obj)
    bpy.context.scene.camera = cam_obj

    # Crear curva path que sigue el centro del pasillo
    puntos_path = []
    fps        = 30
    duracion_s =  2 # segundos para recorrer todo el pasillo
    total_frames = fps * duracion_s
    bpy.context.scene.frame_start = 1
    bpy.context.scene.frame_end   = total_frames

    # Altura de cámara dentro del pasillo (ojos)
    cam_z = 1.6

    # Keyframe de posición y rotación en cada bloque
    bloques_kf = total_bloques
    for i in range(bloques_kf):
        frame = int(1 + (i / (bloques_kf - 1)) * (total_frames - 1))

        cx  = offset_x(i)
        cy  = i * paso
        rot = angulo_tangente(i)

        cam_obj.location = (cx, cy, cam_z)
        cam_obj.rotation_euler = (math.radians(90), 0, rot)

        cam_obj.keyframe_insert(data_path="location",        frame=frame)
        cam_obj.keyframe_insert(data_path="rotation_euler",  frame=frame)

    # Suavizar interpolación de todos los keyframes
    action = cam_obj.animation_data.action
    for fcurve in action.fcurves:
        for kp in fcurve.keyframe_points:
            kp.interpolation = 'BEZIER'

    # Ajustes de render para previsualización
    bpy.context.scene.render.fps = fps
    bpy.context.scene.render.resolution_x = 900
    bpy.context.scene.render.resolution_y = 920

    print("✅ Pasillo generado. Pulsa ESPACIO en Blender para ver el recorrido.")

generar_pasillo_curvo()
```
<img width="1024" height="544" alt="image" src="https://github.com/user-attachments/assets/564f1b59-b97e-4752-8722-f8fed684cad2" />



---


