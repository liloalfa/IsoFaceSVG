# IsoFace SVG

Aplicación web autónoma de un solo archivo (`index.html`, sin backend ni paso de compilación) que reconstruye el **contorno SVG plano 2D de la cara frontal** de logos con letras extruidas en vista isométrica utilizando **OpenCV.js 4.9.0** directamente en el navegador.

## Uso rápido

1. Abre `index.html` directamente en un navegador moderno (o sírvelo con cualquier servidor estático).
2. **Paso 1 — Cargar**: Arrastra una captura (PNG/JPG/WEBP), pégala desde el portapapeles (`Ctrl+V`) o pulsa **"Probar con ejemplo isométrico"**. Las imágenes grandes se escalan a un máximo de 2000 px en su lado mayor.
3. **Paso 2 — Aislar cara frontal**: Ajusta `K` y el filtro bilateral en espacio Lab, haz clic sobre la cara frontal del logo para elegir el cluster semilla y usa los pinceles **Incluir (+)** / **Excluir (-)** con **GrabCut** y limpieza morfológica para eliminar reflejos o caras laterales.
4. **Paso 3 — Rectificar en 2D, reconocer letras y exportar SVG**: El modo automático detecta la orientación isométrica, endereza la línea base a **0° horizontal** sobre un lienzo 2D plano normal y reconoce cada letra (`A–Z`, `a–z`, `0–9`, editable) asignando su identificador al `<path>` del `.svg`.
5. **Paso 4 — Extruir 3D (GLB / STL)**: Triangula los contornos SVG (respetando agujeros), extruye cada letra con profundidad y color lateral configurables en un visor WebGL interactivo (con vista previa coincidente con el mockup original) y exporta a **`.glb` (glTF 2.0)** o **`.stl` binario**.

## Consejos para obtener mejores resultados

- **Separación tonal cara frontal vs. extrusión**: Funcionan mejor las capturas donde la cara frontal tiene un tono o iluminación claramente distinta de las caras laterales de profundidad.
- **Reflejos especulares y degradados**: Si una letra tiene brillo blanco o degradado fuerte, sube **Atenuar reflejos (Bilateral)** a `2` o `3` y baja **Tolerancia a degradados (L*)** hacia `0.45–0.65` para que K-means agrupe toda la cara frontal en un mismo cluster antes de aplicar GrabCut.
- **Bordes limpios sin compresión excesiva**: Capturas PNG o WebP de mockups digitales producen bordes subpíxel mucho más fieles (desviación < 1–2 px) que capturas JPG muy comprimidas o fotografías con ruido.
- **Uso correcto del modo Manual (4 pts)**: Alinea los puntos 1→2 con la dirección horizontal real de las letras y 1→4 con la dirección vertical de sus trazos ascendentes.

## Limitaciones conocidas

- **Oclusiones severas o colores idénticos**: Si el reflejo especular o la cara lateral tienen exactamente la misma cromaticidad y luminancia que la cara frontal, K-means y GrabCut requerirán trazos manuales de pincel para separar las regiones.
- **Perspectiva cónica fuerte**: El modo automático asume proyección ortográfica/isométrica paralela; si el mockup tiene punto de fuga marcado, utiliza el modo **Manual (4 pts)**.
- **Dependencia inicial de red para OpenCV.js**: La primera carga requiere acceso a CDN (`jsDelivr` / `docs.opencv.org`) para descargar el binario WebAssembly de OpenCV.js `4.9.0`.

## 3 mejoras futuras sugeridas

1. **Detección automática del plano y ángulo isométrico mediante Transformada de Hough / LSD**: Estimar los ángulos dominantes de los bordes rectos de la extrusión y la cara frontal para preseleccionar automáticamente el plano (`left`/`right`/`top`) y el ángulo exacto sin intervención manual.
2. **Alineación tipográfica y restricción de simetría/ortogonalidad en la homografía**: Permitir que el usuario marque dos segmentos perpendiculares reales de una letra (por ejemplo, el asta vertical y la barra horizontal de una `E` o `T`) para resolver tanto la rotación como la relación de aspecto exacta de la fuente.
3. **Empaquetado offline opcional con Web Worker + Service Worker**: Mover la ejecución de `cv.kmeans` y `cv.grabCut` a un Web Worker dedicado para mantener la interfaz a 60 fps en imágenes de 2000 px y cachear el `.wasm` de OpenCV.js localmente para uso 100% sin conexión.
