## Actividad 1: Laboratorio de Auditoría y Rendimiento Web

# 1. Auditoría de Red (Network): Análisis de Carga Inicial (Captura_1 y Captura_2)

## Análisis de las capturas de pantalla

* **Documento HTML inicial (Filtro Doc):** La petición raíz a `www.youtube.com` devuelve una respuesta HTML de solo **1.1 kB** transferidos[cite: 1].
* **Carga de scripts (Filtro JS):** Se descargan múltiples librerías y módulos ejecutables (`m=kevlar_base_module...`, `spf.js`, `base.js`)[cite: 2], sumando un peso total de recursos que supera los **20 MB**[cite: 1, 2].

## Justificación técnica: SSR vs. CSR (CE a)

Los datos obtenidos confirman que la aplicación utiliza **CSR (Client-Side Rendering)**:

1. **HTML mínimo:** El servidor solo entrega un cascarón o esqueleto HTML sin la interfaz ni el contenido de la página[cite: 1]. Si utilizara SSR, el servidor renderizaría los componentes en sus nodos y enviaría un HTML mucho más pesado con todo el contenido ya maquetado.
2. **Renderizado delegado al navegador:** El cliente recibe este HTML base y asume el trabajo de descargar y ejecutar megabytes de código JavaScript[cite: 2]. Estos scripts realizan las peticiones a la API y generan el DOM dinámicamente en el propio navegador.

# 2. Destripando el Motor (Performance) (Grabación_1)

## Análisis de la captura de rendimiento

* **Métricas Web Vitals registradas:** Durante la interacción se observa un **LCP (Largest Contentful Paint)** de **2.66 s** y un **INP (Interaction to Next Paint)** de **192 ms**, además del registro de eventos de interacción sobre elementos dinámicos (`video-in-sequence-thumbnail`, `video-stream`)[cite: 3].
* **Actividad del hilo principal:** En la parte inferior se aprecia la traza de eventos de interacción (`pointer` / `Layout shifts`)[cite: 3], reflejando la respuesta de la interfaz tras la ejecución del código.

## Fases del motor JavaScript en el ciclo de ejecución (CE b, CE f)

Durante el perfilado de rendimiento, el motor JS del navegador (como V8 en Chrome) ejecuta de forma continua las siguientes tareas:

1. **Parsing HTML:** El motor lee la estructura del código recibido y construye la representación en árbol del DOM. Si encuentra etiquetas `<script>`, interrumpe o posterga el parseo para ceder el control al motor de JavaScript.
2. **Compile Code (JIT - Just-In-Time):** El motor toma el código fuente de JavaScript y lo traduce rápidamente en código máquina ejecutable por la CPU mediante compilación en tiempo real (JIT), optimizando las funciones de uso frecuente.
3. **Evaluate Script:** Se ejecuta el código JavaScript ya compilado. En este punto el motor procesa los eventos de usuario (como los *clicks* o *hovers* capturados en el panel)[cite: 3], actualiza el estado de la aplicación SPA y manipula el DOM para repintar la pantalla.


# 3. El Sandbox en acción (Consola)

## Análisis de las ejecuciones en la Consola

* **Ejecución estándar:** La sentencia `const a = "eoo"; console.log(a);` se ejecuta en el entorno del navegador e imprime el valor `"eoo"` en salida estándar sin restricciones[cite: 4, 5].
* **Intento de acceso al sistema local:** Al ejecutar `r.readAsText("C:/Windows/system.ini")`, la API `FileReader` lanza una excepción de tipo `Uncaught TypeError: Failed to execute 'readAsText' on 'FileReader': parameter 1 is not of type 'Blob'`[cite: 5]. 

## Restricciones de seguridad y Sandbox (CE b)

1. **Aislamiento del Sandbox:** El navegador ejecuta el código JavaScript web dentro de un entorno totalmente aislado (*Sandbox*). La API `FileReader` está diseñada para procesar únicamente objetos de tipo `Blob` o `File` seleccionados de forma explícita por el usuario mediante controles de interfaz (como `<input type="file">`). No es posible pasar una ruta de sistema de archivos local (`C:/...`) en formato cadena de texto para forzar una lectura arbitraria.
2. **Importancia para la seguridad del usuario:** Esta restricción impide que cualquier sitio web malicioso pueda explorar o extraer datos confidenciales del disco duro de la víctima (claves privadas, documentos o archivos del sistema) de forma silenciosa e inadvertida.

---

# 4. Análisis de Bloqueo (Captura_3 y Captura_4)

## Impacto de la ejecución síncrona en scripts de gran tamaño (CE d)

* **Comportamiento síncrono tradicional (Bloqueante):** Si un script masivo (superior a 1 MB) se cargara de forma síncrona en el hilo principal sin atributos como `async` o `defer`, el navegador congelaría por completo la construcción del árbol DOM y el procesado de estilos CSS. Durante la descarga y parseo del script, la página quedaría inactiva con la pantalla en blanco (*render-blocking*), impidiendo cualquier interacción por parte del usuario.
* **Modelo asíncrono y orientado a eventos en SPAs:** Las arquitecturas web modernas mitigan esta problemática descargando los scripts en segundo plano sin interrumpir el hilo principal. Gracias al bucle de eventos (*Event Loop*), la interfaz se mantiene responsiva de forma continua, delegando la ejecución de la lógica a eventos no bloqueantes.
