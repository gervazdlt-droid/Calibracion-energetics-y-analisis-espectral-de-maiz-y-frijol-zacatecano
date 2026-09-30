# Metodología resumida

## 1. Procesamiento inicial en Python

El desarrollo inicial se realizó en Python para explorar y validar el procesamiento de los espectros gamma. La secuencia contempló la lectura de archivos MCA, extracción de metadatos, corrección por tiempo vivo, sustracción del fondo, calibración de energía y análisis de picos.

## 2. Calibración energética

La calibración relaciona la posición de un evento en el espectro, expresada como canal, con su energía en keV. Se utiliza un ajuste lineal con puntos de referencia conocidos:

```text
E = a · canal + b
```

El RMS reportado resume la discrepancia entre las energías de referencia y las energías calculadas por el ajuste.

## 3. Conversión a aplicación web

Una vez validada la lógica en Python, el flujo se implementó en HTML, CSS y JavaScript para permitir un análisis interactivo en el navegador. Los archivos se procesan localmente en el navegador; no se envían a un servidor.

## 4. Archivos de entrada

- Archivo de muestra.
- Archivo de fondo.
- Archivo de estándar o referencia.

Los archivos deben contener una sección `<<DATA>>` con las cuentas por canal y, cuando esté disponible, una sección `<<CALIBRATION>>` con pares canal–energía.

## 5. Interpretación

Los radionúclidos mostrados son identificaciones orientativas basadas en energías gamma de referencia. La confirmación y cuantificación final requieren revisar eficiencia, geometría, autoabsorción, tiempo muerto, incertidumbres, equilibrio secular y demás controles del protocolo experimental.
