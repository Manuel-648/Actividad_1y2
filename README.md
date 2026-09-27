# Actividad_1y2

## Informe de resultados

Durante la práctica probé las diferentes formas de cargar JavaScript en el HTML usando los archivos `script1.js`, `script2.js` y `script3.js`. Los resultados fueron los siguientes:

### A. `<script>` en `<head>`

```html
<script src="script1.js"></script>
```

El script se ejecuta antes de que se haya creado el `<h1>`, por lo que no puede encontrar el elemento y se produce un error.

**Resultado: No funciona.**

### B. `<script>` al final del `<body>`

```html
<script src="script1.js"></script>
```

En este caso el `<h1>` ya está creado cuando se ejecuta el script, por lo que puede modificarlo sin problemas.

**Resultado: Funciona correctamente.**

### C. `<script async>` en `<head>`

```html
<script async src="script1.js"></script>
```

El archivo se descarga mientras se carga la página y se ejecuta en cuanto termina de descargarse. Por eso, dependiendo del momento en que se ejecute, el `<h1>` puede estar creado o no.

**Resultado: Puede funcionar o dar error.**

### D. `<script defer>` en `<head>`

```html
<script defer src="script1.js"></script>
```

El script se descarga mientras se carga la página, pero se ejecuta después de procesar el HTML. De esta forma, el `<h1>` ya está disponible.

**Resultado: Funciona correctamente.**

### E. `<script type="module>` en `<head>`

```html
<script type="module" src="script1.js"></script>
```

Los módulos se ejecutan de forma diferida, por lo que el HTML ya ha sido procesado cuando se ejecuta el script.

**Resultado: Funciona correctamente.**

### Conclusión

La posición y el tipo de carga del `<script>` afectan al momento en que se ejecuta JavaScript. El script tradicional en el `<head>` puede ejecutarse demasiado pronto, mientras que `defer` y `module` esperan a que el HTML esté procesado. `async` no garantiza este orden.
