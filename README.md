# Callsign Card Generator

Generador de imágenes para radioaficionados, sin cuenta ni servidor.

**🌐 [mammoli.ar/callsign-card](https://mammoli.ar/callsign-card/)**

## Generadores

| Estilo | Dimensión (descarga) | Descripción |
|---|---|---|
| Patente Argentina | 1520 × 480 px | Placa estilo Mercosur con franjas de bandera |
| QSL Header | 1520 × 640 px | Fondo negro con ondas de radio y campos de QSO |
| ID Card | 1360 × 800 px | Tarjeta de identificación con código de barras DMR |
| Banner redes sociales | 1520 × 560 px | Compatible con Twitter/X header y Facebook cover |

## Detección automática

Al escribir el indicativo el generador detecta automáticamente:
- **País** por prefijo ITU (~40 prefijos cubiertos)
- **Provincia argentina** por la letra de sufijo (LU/LW/LV/LT/LP/LO/AY)

Los campos auto-completados se marcan con badge verde "AUTO".

## Calculadora Maidenhead

- Latitud/Longitud → Grid 6 caracteres (y viceversa)
- Zonas CQ e ITU aproximadas
- Distancia y rumbo entre dos grids (fórmula de Haversine)

## Stack

HTML5 + CSS + JavaScript vanilla — sin frameworks, sin build step, sin servidor.
Las imágenes se generan en el navegador con Canvas 2D API y se descargan client-side.

## Licencia

MIT License — Carlos Ariel Mammoli (LU2MCA), Mendoza, Argentina
