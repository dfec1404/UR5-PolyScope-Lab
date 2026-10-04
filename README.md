# UR5-PolyScope-Lab

Laboratorio de programación del robot **UR5 (CB3)** en PolyScope, usando URSim en Docker e integración con ROS 2 Jazzy.
Curso de Robótica (RBTC) — Escuela Colombiana de Ingeniería Julio Garavito.

**Integrantes:**
- Daniel Felipe Escobar Castellanos
- Juan Diego García Molina

## Entorno

| Componente | Versión |
|---|---|
| Sistema operativo | Ubuntu 24.04 LTS (VirtualBox) |
| ROS 2 | Jazzy |
| Simulador del controlador | URSim CB3 — PolyScope 3.15 (`universalrobots/ursim_cb3`) |
| URCap | [External Control 1.0.5](https://github.com/UniversalRobots/Universal_Robots_ExternalControl_URCap/releases) |
| Driver ROS 2 | `ros-jazzy-ur` (Universal_Robots_ROS2_Driver) |

## Programas

### Programa 1 — Iniciales (`program1.urp`)
Simula la escritura de las iniciales **D E** (Daniel Escobar) y **J G** (Juan García).
- Antes de cada letra, un `Popup` indica qué letra se va a simular.
- Las trayectorias están en `letra_D.script`, `letra_E.script`, `letra_J.script` y `letra_G.script` (URScript, comandos `movel`), cargadas con nodos `Script` en modo *File*.
- Letras de 80 × 100 mm en el plano horizontal: lápiz abajo a Z = 100 mm, lápiz arriba a Z = 150 mm.

### Programa 2 — Control compartido (`program2.urp`)
1. `Set`: salida digital `DO[0]` en alto.
2. `Popup`: avisa que el control será compartido con un controlador externo.
3. `Wait`: 5 s después de confirmar el popup.
4. `External Control`: cede el control a ROS 2 (host `172.17.0.1`, puerto `50002`).

**Bono:** planificación y ejecución de una trayectoria con MoveIt 2 desde ROS 2 (`ur_moveit_config`).

### Programa 3 — Pick and place (`program3.urp`)
- Efector final simulado con la salida digital `DO[1]` (HI = agarra, LOW = suelta).
- Dos recogidas (P1, P2) y dos llegadas (L1, L2), con puntos de aproximación arriba de cada una y pausas en los puntos intermedios.
- Payload de 1.0 kg al agarrar y 0.0 kg al soltar.
- Al terminar, espera la entrada digital `DI[0]` en alto para repetir (programa en *loop*).

## Estructura del repositorio

| Archivo | Descripción |
|---|---|
| `programas/program*.urp` | Programas de PolyScope |
| `programas/program*.txt` | Árbol de cada programa en texto |
| `programas/program*.script` | URScript generado por PolyScope |
| `programas/letra_*.script` | Trayectorias de las letras (Programa 1) |
| `programas/default.installation` | Instalación del robot (configuración de External Control) |

## Cómo ejecutar

```bash
docker run -it \
  -v ~/ursim_cb3_data/programs:/ursim/programs \
  -v ~/ursim_cb3_data/urcaps:/urcaps \
  -p 6080:6080 -p 29999:29999 -p 30001-30004:30001-30004 \
  --name ursim_cb3 universalrobots/ursim_cb3
```

Copiar el contenido de `programas/` a `~/ursim_cb3_data/programs`, abrir PolyScope en `http://localhost:6080/vnc.html?host=localhost&port=6080` y cargar el programa.

Para el Programa 2, lanzar el driver antes de dar Play:

```bash
ros2 launch ur_robot_driver ur_control.launch.py ur_type:=ur5 robot_ip:=172.17.0.2 launch_rviz:=false
```

## Problemas encontrados y soluciones

- **Volumen de Docker:** montar una carpeta sobre `/ursim` ocultaba el simulador. Se montaron solo `/ursim/programs` y `/urcaps`.
- **IP del driver:** con `robot_ip:=127.0.0.1` el URCap intentaba conectarse a sí mismo dentro del contenedor. Se usó la IP del contenedor (`172.17.0.2`), con la IP de `docker0` (`172.17.0.1`) configurada en el URCap.
- **Singularidad de hombro:** un `MoveL` entre (300, 400) y (-300, -400) cruzaba el eje de la base. Se reemplazó por un `MoveJ` para los traslados.
- **Límites de articulación:** algunos waypoints quedaron guardados con ángulos fuera de rango (por ejemplo, base en 400.52°). Se reescribieron con ángulos equivalentes (±360°), todos en la misma rama de la cinemática inversa.
- **Rendimiento en la VM:** RViz consumía más de un núcleo y el `controller_manager` no alcanzaba los 125 Hz, lo que cortaba la conexión. Se lanzó el driver con `launch_rviz:=false`.
cat > README.md << 'EOF'
# UR5-PolyScope-Lab

Laboratorio de programación del robot **UR5 (CB3)** en PolyScope, usando URSim en Docker e integración con ROS 2 Jazzy.
Curso de Robótica (RBTC) — Escuela Colombiana de Ingeniería Julio Garavito.

**Integrantes:**
- Daniel Felipe Escobar Castellanos
- Juan Diego García Molina

## Entorno

| Componente | Versión |
|---|---|
| Sistema operativo | Ubuntu 24.04 LTS (VirtualBox) |
| ROS 2 | Jazzy |
| Simulador del controlador | URSim CB3 — PolyScope 3.15 (`universalrobots/ursim_cb3`) |
| URCap | [External Control 1.0.5](https://github.com/UniversalRobots/Universal_Robots_ExternalControl_URCap/releases) |
| Driver ROS 2 | `ros-jazzy-ur` (Universal_Robots_ROS2_Driver) |

## Programas

### Programa 1 — Iniciales (`program1.urp`)
Simula la escritura de las iniciales **D E** (Daniel Escobar) y **J G** (Juan García).
- Antes de cada letra, un `Popup` indica qué letra se va a simular.
- Las trayectorias están en `letra_D.script`, `letra_E.script`, `letra_J.script` y `letra_G.script` (URScript, comandos `movel`), cargadas con nodos `Script` en modo *File*.
- Letras de 80 × 100 mm en el plano horizontal: lápiz abajo a Z = 100 mm, lápiz arriba a Z = 150 mm.

### Programa 2 — Control compartido (`program2.urp`)
1. `Set`: salida digital `DO[0]` en alto.
2. `Popup`: avisa que el control será compartido con un controlador externo.
3. `Wait`: 5 s después de confirmar el popup.
4. `External Control`: cede el control a ROS 2 (host `172.17.0.1`, puerto `50002`).

**Bono:** planificación y ejecución de una trayectoria con MoveIt 2 desde ROS 2 (`ur_moveit_config`).

### Programa 3 — Pick and place (`program3.urp`)
- Efector final simulado con la salida digital `DO[1]` (HI = agarra, LOW = suelta).
- Dos recogidas (P1, P2) y dos llegadas (L1, L2), con puntos de aproximación arriba de cada una y pausas en los puntos intermedios.
- Payload de 1.0 kg al agarrar y 0.0 kg al soltar.
- Al terminar, espera la entrada digital `DI[0]` en alto para repetir (programa en *loop*).

## Estructura del repositorio

| Archivo | Descripción |
|---|---|
| `programas/program*.urp` | Programas de PolyScope |
| `programas/program*.txt` | Árbol de cada programa en texto |
| `programas/program*.script` | URScript generado por PolyScope |
| `programas/letra_*.script` | Trayectorias de las letras (Programa 1) |
| `programas/default.installation` | Instalación del robot (configuración de External Control) |

## Cómo ejecutar

```bash
docker run -it \
  -v ~/ursim_cb3_data/programs:/ursim/programs \
  -v ~/ursim_cb3_data/urcaps:/urcaps \
  -p 6080:6080 -p 29999:29999 -p 30001-30004:30001-30004 \
  --name ursim_cb3 universalrobots/ursim_cb3
```

Copiar el contenido de `programas/` a `~/ursim_cb3_data/programs`, abrir PolyScope en `http://localhost:6080/vnc.html?host=localhost&port=6080` y cargar el programa.

Para el Programa 2, lanzar el driver antes de dar Play:

```bash
ros2 launch ur_robot_driver ur_control.launch.py ur_type:=ur5 robot_ip:=172.17.0.2 launch_rviz:=false
```

## Problemas encontrados y soluciones

- **Volumen de Docker:** montar una carpeta sobre `/ursim` ocultaba el simulador. Se montaron solo `/ursim/programs` y `/urcaps`.
- **IP del driver:** con `robot_ip:=127.0.0.1` el URCap intentaba conectarse a sí mismo dentro del contenedor. Se usó la IP del contenedor (`172.17.0.2`), con la IP de `docker0` (`172.17.0.1`) configurada en el URCap.
- **Singularidad de hombro:** un `MoveL` entre (300, 400) y (-300, -400) cruzaba el eje de la base. Se reemplazó por un `MoveJ` para los traslados.
- **Límites de articulación:** algunos waypoints quedaron guardados con ángulos fuera de rango (por ejemplo, base en 400.52°). Se reescribieron con ángulos equivalentes (±360°), todos en la misma rama de la cinemática inversa.
- **Rendimiento en la VM:** RViz consumía más de un núcleo y el `controller_manager` no alcanzaba los 125 Hz, lo que cortaba la conexión. Se lanzó el driver con `launch_rviz:=false`.

### Programa 3 — Pick and place (`program3.urp`)
- Efector final simulado con la salida digital `DO[1]` (HI = agarra, LOW = suelta).
- Dos recogidas (P1, P2) y dos llegadas (L1, L2), con puntos de aproximación arriba de cada una y pausas en los puntos intermedios.
- Payload de 1.0 kg al agarrar y 0.0 kg al soltar.
- Al terminar, espera la entrada digital `DI[0]` en alto para repetir (programa en *loop*).

## Estructura del repositorio

| Archivo | Descripción |
|---|---|
| `programas/program*.urp` | Programas de PolyScope |
| `programas/program*.txt` | Árbol de cada programa en texto |
| `programas/program*.script` | URScript generado por PolyScope |
| `programas/letra_*.script` | Trayectorias de las letras (Programa 1) |
| `programas/default.installation` | Instalación del robot (configuración de External Control) |

## Cómo ejecutar

```bash
docker run -it \
  -v ~/ursim_cb3_data/programs:/ursim/programs \
  -v ~/ursim_cb3_data/urcaps:/urcaps \
  -p 6080:6080 -p 29999:29999 -p 30001-30004:30001-30004 \
  --name ursim_cb3 universalrobots/ursim_cb3
```

Copiar el contenido de `programas/` a `~/ursim_cb3_data/programs`, abrir PolyScope en `http://localhost:6080/vnc.html?host=localhost&port=6080` y cargar el programa.

Para el Programa 2, lanzar el driver antes de dar Play:

```bash
ros2 launch ur_robot_driver ur_control.launch.py ur_type:=ur5 robot_ip:=172.17.0.2 launch_rviz:=false
```

## Problemas encontrados y soluciones

- **Volumen de Docker:** montar una carpeta sobre `/ursim` ocultaba el simulador. Se montaron solo `/ursim/programs` y `/urcaps`.
- **IP del driver:** con `robot_ip:=127.0.0.1` el URCap intentaba conectarse a sí mismo dentro del contenedor. Se usó la IP del contenedor (`172.17.0.2`), con la IP de `docker0` (`172.17.0.1`) configurada en el URCap.
- **Singularidad de hombro:** un `MoveL` entre (300, 400) y (-300, -400) cruzaba el eje de la base. Se reemplazó por un `MoveJ` para los traslados.
- **Límites de articulación:** algunos waypoints quedaron guardados con ángulos fuera de rango (por ejemplo, base en 400.52°). Se reescribieron con ángulos equivalentes (±360°), todos en la misma rama de la cinemática inversa.
- **Rendimiento en la VM:** RViz consumía más de un núcleo y el `controller_manager` no alcanzaba los 125 Hz, lo que cortaba la conexión. Se lanzó el driver con `launch_rviz:=false`.
