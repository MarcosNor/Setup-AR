# Guía: Mostrar un cubo en una superficie plana con AR

Objetivo: Que aparezca un cubo sobre tu mesa usando la cámara del celular.

---

## Paso 1: Crear el proyecto en Unity (en Windows)

1. Abre **Unity Hub** (el administrador de proyectos).
2. Click en **New Project**.
3. Elige la plantilla **3D Core** (no la de AR, la haremos manual).
4. Nombra el proyecto: `SetupAR` (sin espacios).
5. Click en **Create Project**.

Espera unos minutos mientras Unity abre.

---

## Paso 2: Instalar los paquetes de AR

Unity necesita dos paquetes para AR:

1. En Unity, ve a **Window → Package Manager**.
2. Arriba a la izquierda, cambia de "Packages: In Project" a **"Packages: Unity Registry"**.
3. Busca e instala estos dos:
   - **AR Foundation**
   - **ARKit XR Plugin** (para iOS) o **ARCore XR Plugin** (para Android)

Instala **ARCore** si vas a probar en Android, o **ARKit** si vas a probar en iPhone.

---

## Paso 3: Configurar la escena para AR

1. En la pestaña **Scene**, borra todo lo que haya (el GameObject "Main Camera" y "Directional Light" — los vas a reemplazar).
2. Click derecho en el panel **Hierarchy → XR → AR Session Origin**. Esto crea un objeto especial que maneja la cámara de AR.
3. Click derecho en el panel **Hierarchy → XR → AR Session**. Esto controla la sesión de AR.
4. Haz click en **AR Session Origin** en la Hierarchy. En el Inspector, vas a ver un componente llamado **Camera**. Asegúrate de que el objeto **Main Camera** esté asignado ahí (debería aparecer como "Main Camera (Camera)" — si no, arrastra el Main Camera que está dentro de AR Session Origin al campo).

Tu escena debería verse así en la Hierarchy:

```
AR Session
AR Session Origin
  └── Camera (Main Camera)
```

---

## Paso 4: Configurar Android/iOS para build

1. Ve a **File → Build Settings**.
2. Selecciona **Android** (si tenés Android) o **iOS** (si tenés iPhone).
3. Click en **Switch Platform** (tarda unos segundos).
4. En **Player Settings** (botón que aparece en Build Settings):
   - **Company Name**: pon tu nombre.
   - **Product Name**: `SetupAR`.
   - En **Other Settings → Identification**: cambia el **Bundle Identifier** a algo como `com.tuNombre.setupar`.

---

## Paso 5: Crear el script que pone el cubo

1. En el panel **Project** (abajo), click derecho → **Create → C# Script**.
2. Nómbralo: `CubePlacer`.
3. Haz doble click para abrirlo en tu editor de código (Visual Studio o el que tengas).
4. Reemplaza TODO el contenido con este script:

```csharp
using UnityEngine;
using UnityEngine.XR.ARFoundation;
using UnityEngine.XR.ARSubsystems;

public class CubePlacer : MonoBehaviour
{
    public ARRaycastManager raycastManager;
    public GameObject cubePrefab;

    void Update()
    {
        if (Input.touchCount == 0) return;

        Touch touch = Input.GetTouch(0);
        if (touch.phase != TouchPhase.Began) return;

        List<ARRaycastHit> hits = new List<ARRaycastHit>();
        if (raycastManager.Raycast(touch.screenPosition, hits, TrackableType.PlaneWithinPolygon))
        {
            Pose hitPose = hits[0].pose;
            Instantiate(cubePrefab, hitPose.position, hitPose.rotation);
        }
    }
}
```

5. Guarda el archivo (Ctrl+S).

---

## Paso 6: Asignar el script en la escena

1. En Unity, crea un Cube: **GameObject → 3D Object → Cube**.
2. Escala el cube a algo chico: en el Inspector, en **Transform → Scale**, pon `0.1` en X, Y y Z.
3. En el panel **Project**, click derecho → **Create → Prefab**.
4. Arrastra el Cube que acabaste de crear dentro del Prefab. Se va a volver azul.
5. Borra el Cube de la escena (ya no lo necesitás, el prefab quedó guardado).
6. Crea un GameObject vacío: **GameObject → Create Empty**.
7. Nómbralo: `CubePlacer`.
8. Arrastra el script `CubePlacer` sobre el GameObject `CubePlacer` para asignárselo.
9. Con `CubePlacer` seleccionado, en el Inspector vas a ver los campos del script:
   - **Raycast Manager**: arrastra el objeto `AR Session Origin` de la Hierarchy.
   - **Cube Prefab**: arrastra el Prefab que creaste en el panel Project.

---

## Paso 7: Probar en el celular

1. Conecta el celular a la computadora por USB.
2. En el celular, activa **Opciones de desarrollador → Depuración USB** (en Android).
3. En Unity, en **Build Settings**, debería aparecer tu dispositivo.
4. Haz click en **Build and Run**.
5. Va a compilar y subir la app al celular.

---

## Qué hace el script (explicación rápida)

```
¿El usuario tocó la pantalla?
    ↓ Sí
Lanzar un "rayo" (raycast) desde el dedo hacia la cámara
    ↓
¿El rayo golpeó una superficie plana detectada?
    ↓ Sí
Crear un cubo en el punto exacto donde golpeó el rayo
```

---

## Notas

- El script usa `ARRaycastManager` para detectar superficies planas.
- `TrackableType.PlaneWithinPolygon` significa que solo coloca el cubo si está dentro de un borde detectado (no en el aire).
- `Instantiate` crea una copia del prefab en la posición donde el usuario tocó.

---

## Próximos pasos (cuando esto funcione)

1. Hacer que el cubo aparezca en un lugar fijo (no donde el usuario toca).
2. Crear flechas o indicadores visuales.
3. Cargar un layout de juego de mesa desde un JSON.
4. Identificar el juego automáticamente con ML.
