# SpectraLab — Calibración energética y análisis espectral

Aplicación web para la **calibración energética y el análisis de espectros gamma** de muestras de maíz y frijol zacatecano, medidas con un detector HPGe (Germanio Hiperpuro) de alta resolución.

## Descripción del proyecto

El proyecto comenzó como un flujo de análisis desarrollado en **Python**. En esa primera etapa se trabajó con los archivos de espectrometría, la lectura de espectros MCA, la normalización del tiempo vivo, la sustracción del fondo, el ajuste de la calibración canal–energía y la identificación de picos gamma.

Después, el procedimiento se trasladó a una página web para que el análisis pudiera ejecutarse desde el navegador, sin instalar Python. La aplicación conserva la lógica principal del procesamiento y presenta los resultados mediante gráficas, tablas y tarjetas de radionúclidos.

## ¿Cómo funciona?

La página permite cargar hasta tres archivos `.mca` o `.txt`:

1. **Muestra:** espectro de maíz o frijol zacatecano.
2. **Fondo:** medición del ruido de fondo, por ejemplo, el recipiente o cartón empleado.
3. **Estándar de referencia:** archivo de referencia para apoyar la calibración energética.

Con los archivos cargados, el usuario indica la masa de la muestra y presiona **Analizar**. El navegador realiza lo siguiente:

1. Lee el formato MCA y obtiene metadatos, tiempo vivo y cuentas por canal.
2. Normaliza el fondo usando la razón de tiempos vivos.
3. Calcula el espectro neto de la muestra.
4. Ajusta una calibración lineal de la forma:

   `E(keV) = a × Canal + b`

5. Estima centroides y anchuras de los picos mediante un ajuste gaussiano simplificado.
6. Convierte los canales a energía.
7. Presenta el RMS de la calibración, los espectros bruto y neto, la curva de calibración y los picos identificados.

## Resultados que muestra

- Tiempo vivo de la muestra y del fondo.
- Factor de normalización.
- Cuentas netas.
- Ecuación canal–energía.
- RMS de la calibración.
- Espectro de la muestra frente al fondo normalizado.
- Espectro neto.
- Curva de calibración.
- Tabla de residuos de calibración.
- Energía, FWHM, área neta y SNR de los picos detectados.

## Ejecución local

La aplicación es estática. Puede abrirse directamente haciendo doble clic en `index.html`; sin embargo, se recomienda usar un servidor local:

```bash
python -m http.server 8000
```

Después, abre <http://localhost:8000>.

## Publicación en GitHub

1. Crea un repositorio vacío en GitHub, por ejemplo `spectralab-maiz-frijol-zacatecas`.
2. Desde esta carpeta ejecuta:

```bash
git init
git add .
git commit -m "Primera versión de SpectraLab"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/spectralab-maiz-frijol-zacatecas.git
git push -u origin main
```

Para activar GitHub Pages: **Settings → Pages → Deploy from a branch → main → / (root)**.

## Estructura

```text
.
├── index.html
├── README.md
└── docs/
    └── metodologia.md
```

## Archivos necesarios para el análisis

Para usar el analizador se recomienda cargar tres archivos: **muestra**, **fondo** y **estándar de calibración**. La muestra contiene el espectro de maíz o frijol; el fondo permite eliminar contribuciones ambientales; el estándar proporciona energías gamma conocidas para validar la relación canal–energía. Los formatos aceptados son `.mca` y `.txt` con una sección `<<DATA>>` y, cuando sea posible, `<<CALIBRATION>>`.

Para conocer la estructura esperada, el procedimiento completo y la explicación de cada resultado consulta [la guía completa de archivos y metodología](docs/14-guia-completa-archivos-y-metodologia.md).

## Galería de resultados

La página principal incluye una galería local con resultados de análisis espectral, calibración energética y comparación muestra–fondo. La interfaz original se conserva en `analyzer.html`; el portal visual se encuentra en `index.html`.

### Archivos visuales

- `assets/analisis_espectral.png` — panorama espectroscópico gamma.
- `assets/analisis_final_Jerez.png` — análisis final de la muestra Jerez.
- `assets/calibracion_GAIN7.png` — calibración con IAEA-434 GAIN 7.
- `assets/resultado_calibracion.png` — curva y tabla de calibración.

## Alcance científico

Esta herramienta es una interfaz de análisis y visualización para datos de espectrometría gamma. Los resultados deben validarse con los procedimientos del laboratorio, las características del detector HPGe, la geometría de medición, los estándares empleados, las incertidumbres y los controles de calidad correspondientes.
