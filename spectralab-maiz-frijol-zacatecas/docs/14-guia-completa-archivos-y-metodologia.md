# Guía completa de SpectraLab

## 1. ¿Qué es este proyecto?

SpectraLab es una aplicación web para visualizar, calibrar y analizar espectros gamma obtenidos de muestras agrícolas, especialmente maíz y frijol zacatecano, mediante espectrometría gamma con un detector HPGe de alta resolución.

El proyecto reúne dos partes:

1. **El desarrollo científico y computacional**, realizado inicialmente en Python para leer archivos, comparar espectros, sustraer el fondo, localizar picos y validar la calibración.
2. **La interfaz web**, desarrollada posteriormente con HTML, CSS y JavaScript para que el usuario pueda cargar los archivos y ejecutar el flujo desde el navegador.

La página no sustituye la validación metrológica del laboratorio. Es una herramienta de análisis, visualización y documentación que debe usarse junto con el protocolo experimental, la geometría de medición, la eficiencia del detector y las incertidumbres correspondientes.

---

## 2. ¿Qué archivos necesita el usuario?

La aplicación trabaja con archivos de espectrometría en formato `.mca` o `.txt` que contengan las cuentas registradas por canal. Para un análisis completo se recomienda cargar tres archivos.

### Archivo 1: muestra

Es el espectro de la muestra que se desea estudiar.

Ejemplos:

```text
Frijol_Jerez_01_Nov3_2025.mca
Maiz_Zacatecas_Muestra01.mca
Frijol_Muestra_A.txt
```

Este archivo debe corresponder a la muestra colocada frente al detector HPGe. Contiene las cuentas gamma registradas en cada canal, además de información como tiempo vivo y tiempo real.

La muestra puede ser:

- frijol zacatecano;
- maíz zacatecano;
- material vegetal seco;
- suelo asociado a la muestra;
- estándar o material de control, si el protocolo lo indica.

### Archivo 2: fondo

Es el espectro del ruido de fondo medido bajo condiciones comparables a las de la muestra.

Ejemplos:

```text
Carton.mca
Fondo_Laboratorio.mca
Background_146h.txt
```

El fondo permite distinguir las contribuciones ambientales o del recipiente de las señales asociadas con la muestra. La aplicación normaliza el fondo mediante el tiempo vivo antes de realizar la sustracción.

La comparación es conceptualmente:

```text
Espectro neto = Espectro de muestra − Fondo normalizado
```

Es importante que la medición del fondo tenga una geometría, distancia, blindaje y configuración del detector compatibles con la medición de la muestra.

### Archivo 3: estándar de calibración

Es un espectro de referencia con energías gamma conocidas. Puede ser un estándar certificado o una fuente de referencia, como IAEA-434, siempre que corresponda al procedimiento del laboratorio.

Ejemplos:

```text
IAEA_434_GAIN_7.mca
Estandar_Calibracion.mca
Calibracion_HPGe.txt
```

El estándar ayuda a establecer la relación entre el canal del espectro y la energía en keV. Una calibración típica utiliza varios puntos de referencia:

```text
Canal  →  Energía conocida en keV
716    →  511.00 keV
2058   →  1460.83 keV
3693   →  2614.51 keV
```

El nombre del archivo no es lo importante. Lo importante es que los datos estén correctamente adquiridos y que las energías de referencia estén documentadas.

---

## 3. ¿Son obligatorios los tres archivos?

Para una prueba básica, el sistema puede funcionar con:

- archivo de muestra;
- archivo de fondo.

Sin embargo, para un análisis experimental completo se recomienda usar los tres archivos:

| Archivo | ¿Es necesario? | Función |
|---|---:|---|
| Muestra | Sí | Contiene el espectro que se desea analizar |
| Fondo | Sí | Permite eliminar contribuciones ambientales |
| Estándar | Recomendado | Apoya la calibración energética y su validación |

La tercera entrada debe considerarse necesaria cuando el objetivo sea documentar formalmente la calibración del detector. Si el estándar no contiene una sección de calibración válida, la aplicación puede utilizar picos de referencia conocidos, pero ese resultado debe revisarse cuidadosamente.

---

## 4. Estructura interna esperada del archivo

La aplicación busca una estructura tipo MCA. Un archivo puede incluir metadatos y una sección de datos parecida a la siguiente:

```text
LIVE_TIME - 345600
REAL_TIME - 350000
START_TIME - 2025-11-03 08:42:11
<<CALIBRATION>>
716 511.00
2058 1460.83
3693 2614.51
<<END>>
<<DATA>>
120
135
142
...
<<END>>
```

### Metadatos

Los metadatos permiten conocer:

- tiempo vivo (`LIVE_TIME`);
- tiempo real (`REAL_TIME`);
- fecha y hora de inicio;
- información del equipo;
- identificación del detector;
- condiciones de adquisición.

### Sección `<<DATA>>`

Contiene una cuenta por canal. Cada línea representa el número de eventos registrados en ese canal.

Por ejemplo:

```text
<<DATA>>
100
108
97
123
115
<<END>>
```

### Sección `<<CALIBRATION>>`

Contiene pares de canal y energía:

```text
<<CALIBRATION>>
716 511.00
2058 1460.83
3693 2614.51
<<END>>
```

Si el equipo exporta un formato diferente, primero debe convertirse a un archivo compatible o adaptarse el parser de JavaScript/Python.

---

## 5. ¿Qué hace el análisis?

### Paso 1: cargar los archivos

El usuario arrastra cada archivo a la zona correspondiente o presiona el recuadro para seleccionarlo desde el equipo.

La aplicación identifica:

- nombre del archivo;
- cantidad de canales;
- tiempo vivo;
- puntos de calibración disponibles;
- cuentas por canal.

### Paso 2: normalizar el fondo

La muestra y el fondo pueden tener tiempos vivos diferentes. Por esa razón, el fondo se ajusta con un factor de normalización:

```text
Factor = tiempo vivo de la muestra / tiempo vivo del fondo
```

El fondo normalizado se calcula canal por canal y después se resta del espectro de la muestra.

### Paso 3: obtener el espectro neto

El espectro neto representa la señal después de descontar el fondo. En el código se evita trabajar con valores negativos causados por fluctuaciones estadísticas:

```text
Cuenta neta = máximo(0, cuenta de muestra − cuenta de fondo normalizada)
```

Esto facilita la visualización, pero no elimina la necesidad de analizar las incertidumbres estadísticas.

### Paso 4: ajustar la calibración

La calibración energética utiliza una relación lineal:

```text
E(keV) = a × Canal + b
```

Donde:

- `E` es la energía en keV;
- `Canal` es la posición del evento en el espectro;
- `a` es la ganancia energética;
- `b` es el término independiente o desplazamiento.

La aplicación calcula también el RMS:

```text
RMS = raíz cuadrada del promedio de los residuos al cuadrado
```

Un RMS menor indica que los puntos de referencia están más cerca de la recta ajustada, aunque la aceptación final depende del protocolo y de la resolución del detector.

### Paso 5: localizar picos

Para cada energía de referencia se busca una región alrededor del canal esperado. La aplicación estima:

- centroide;
- FWHM;
- área neta;
- error aproximado del área;
- relación señal/ruido, SNR;
- energía ajustada.

La identificación de un pico debe confirmarse observando el espectro, la intensidad, la resolución, el fondo, las interferencias y las líneas gamma asociadas.

---

## 6. ¿Qué muestra la página?

La página principal (`index.html`) funciona como una presentación del proyecto. Incluye:

- descripción del problema;
- explicación del detector HPGe;
- flujo de trabajo;
- relación entre Python y la aplicación web;
- galería de resultados;
- acceso a la interfaz original.

La interfaz interactiva está en:

```text
analyzer.html
```

En ella se encuentran:

- tres zonas de carga;
- configuración de masa y nombre de muestra;
- ventana de búsqueda de picos;
- consola de proceso;
- métricas;
- espectro bruto;
- fondo normalizado;
- espectro neto;
- curva de calibración;
- tabla de residuos;
- tarjetas de radionúclidos;
- tabla detallada de resultados.

---

## 7. Imágenes incluidas en el repositorio

Las imágenes están en la carpeta `assets/` y se muestran en la galería de `index.html`.

```text
assets/analisis_espectral.png
```

Panorama del análisis espectroscópico gamma con líneas de referencia y escala de energía.

```text
assets/analisis_final_Jerez.png
```

Análisis final de la muestra Jerez, incluyendo muestra, fondo normalizado y espectro neto.

```text
assets/calibracion_GAIN7.png
```

Calibración energética del estándar IAEA-434 GAIN 7, con picos medidos y ajuste canal–energía.

```text
assets/resultado_calibracion.png
```

Resultado visual de la calibración con ecuación, RMS y tabla de puntos de calibración.

Si se agregan nuevas imágenes, deben guardarse dentro de `assets/` y referenciarse desde HTML con una ruta relativa:

```html
<img src="assets/nueva-imagen.png" alt="Descripción de la nueva imagen">
```

---

## 8. Cómo ejecutar el proyecto

### Opción A: abrir directamente

Haz doble clic en `index.html`. Esto permite revisar la página principal. Algunas funciones del analizador pueden depender de que el navegador permita cargar recursos externos.

### Opción B: servidor local recomendado

Desde la carpeta raíz del proyecto ejecuta:

```bash
python -m http.server 8000
```

Después abre:

```text
http://localhost:8000
```

La página inicial será:

```text
http://localhost:8000/index.html
```

El analizador será:

```text
http://localhost:8000/analyzer.html
```

### Opción C: GitHub Pages

Sube toda la estructura al repositorio y configura GitHub Pages usando la rama `main` y la carpeta raíz `/root`. GitHub utilizará `index.html` como página principal.

---

## 9. Qué archivos deben subirse a GitHub

Para que la página funcione y se vea completa, sube todo lo siguiente:

```text
index.html
analyzer.html
README.md
assets/analisis_espectral.png
assets/analisis_final_Jerez.png
assets/calibracion_GAIN7.png
assets/resultado_calibracion.png
docs/metodologia.md
docs/14-guia-completa-archivos-y-metodologia.md
```

No cambies los nombres de las carpetas sin actualizar las rutas dentro de `index.html`.

No es suficiente subir solamente `index.html`, porque las imágenes no aparecerán si no se incluye la carpeta `assets/`.

---

## 10. Qué archivos no deben subirse

No subas:

- datos originales de investigación si contienen información restringida;
- contraseñas;
- tokens;
- archivos de configuración privados;
- datos personales;
- resultados que no hayan sido autorizados para publicación;
- archivos temporales o capturas con información sensible.

Antes de publicar, revisa que el repositorio no contenga nombres completos de participantes, ubicaciones sensibles, credenciales o rutas internas del laboratorio.

---

## 11. Interpretación responsable

Los nombres de radionúclidos y las energías mostradas por la interfaz son identificaciones orientativas basadas en referencias gamma. No constituyen por sí solos una certificación de composición o concentración.

Para emitir un resultado científico se deben considerar, como mínimo:

- calibración energética validada;
- calibración de eficiencia;
- geometría de la muestra;
- masa y composición;
- tiempo vivo y tiempo muerto;
- autoabsorción;
- incertidumbres;
- resolución energética;
- interferencias espectrales;
- equilibrio de las series radiactivas;
- controles de calidad;
- revisión del investigador responsable.

La aplicación ayuda a organizar y visualizar el análisis, pero la decisión científica final pertenece al responsable del laboratorio.

---

## 12. Reproducibilidad

Cada análisis debería registrar:

- nombre de la muestra;
- fecha de adquisición;
- detector utilizado;
- geometría;
- tiempo vivo;
- archivo de muestra;
- archivo de fondo;
- archivo de estándar;
- ecuación de calibración;
- RMS;
- ventana de búsqueda;
- versión del código;
- observaciones del investigador.

Una buena práctica es conservar los archivos originales en una carpeta protegida y guardar los resultados derivados en otra carpeta. Nunca se deben sobrescribir los datos originales durante una corrección o una conversión.
