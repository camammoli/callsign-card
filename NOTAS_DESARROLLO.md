# Callsign Card Generator — Notas de desarrollo

## Descripción

Herramienta web pública para que cualquier radioaficionado genere imágenes de su
indicativo en cuatro formatos, y calcule localizadores Maidenhead, zonas CQ/ITU
y distancias entre grids.

**URL producción:** https://mammoli.ar/callsign-card/

## Stack

- HTML5 + CSS3 + JavaScript vanilla — sin frameworks, sin dependencias, sin build step.
- Canvas 2D API para renderizado de imágenes a 2× DPR (descarga HiDPI).
- Google Fonts CDN: Oswald 600/700, Rajdhani 400/600/700, Share Tech Mono.
- Deploy: archivo único `index.html` + `.htaccess`.

## Deploy

```bash
lftp -c "
set ssl:verify-certificate no
open ftp://carlos%40mammoli.ar:CLAVE@mammoli.ar
mirror --reverse --delete /home/carlos/Scripts/callsign-card/ /callsign-card/
"
```

Las credenciales FTP están en `/home/carlos/Scripts/lu2mca-site/NOTAS_DESARROLLO.md`.

## Arquitectura

Archivo único `index.html`. Todo client-side: las imágenes se generan en el browser
con `canvas.toDataURL()` y se descargan vía anchor click. **Nada se guarda en el servidor.**

### Carga de fuentes

Tres `<span class="font-pre">` invisibles con cada fuente fuerzan al browser a
cargarlas antes del primer render. El canvas sólo dibuja después de `document.fonts.ready`.

---

## Funcionalidades

### 1. Detección automática de indicativo

Al escribir el indicativo se dispara `onCallsignInput()`:

1. **País**: búsqueda del prefijo más largo en `PREFIX_MAP` (≈40 prefijos ITU).
   Cubre: Argentina (LU/LW/LV/LT/LP/LO/AY), Brasil (PY/PP/PU/PT), Chile (CE/CA…),
   Uruguay (CX), Paraguay (ZP), Perú (OA), Colombia (HK), Ecuador (HC), Venezuela (YV),
   Bolivia (CP), USA (W/K/N), Canadá (VE), México (XE), Alemania (DL), Francia (F),
   UK (G/M), Italia (I), España (EA), Japón (JA), Australia (VK), NZ (ZL), Rusia (UA/RA),
   Sudáfrica (ZS) y más.

2. **Provincia argentina**: para prefijos LU/LW/LV/LT/LP/LO/AY, la letra en posición 4
   del indicativo (inmediatamente después del dígito de zona) identifica la provincia.
   Ejemplo: `LU2MCA` → posición 4 = `M` → Mendoza.

   | Letra | Provincia              | Letra | Provincia         |
   |-------|------------------------|-------|-------------------|
   | A, B, D, G, H | Buenos Aires  | N     | Neuquén           |
   | C     | Córdoba                | P     | La Pampa          |
   | E     | Entre Ríos             | Q     | San Luis          |
   | F     | Santa Fe               | R     | Río Negro         |
   | G     | Santiago del Estero    | S     | Salta             |
   | H     | Chaco                  | T     | Tucumán           |
   | I     | Misiones               | U     | Corrientes        |
   | J     | San Juan               | V     | Tierra del Fuego  |
   | K     | Catamarca              | X     | Formosa           |
   | L     | La Rioja               | Y     | Jujuy             |
   | M     | **Mendoza** ✓ confirmado | Z   | Santa Cruz        |

   Los campos auto-llenados se marcan con badge verde "AUTO" y borde verde.
   Si el usuario edita el campo manualmente, el auto-fill se desactiva para ese campo.

### 2. Generadores de imagen (4 estilos)

Todos renderizan a 2× resolución lógica. Dimensiones en píxeles de descarga:

| Estilo          | Canvas lógico | PNG descargado |
|-----------------|---------------|----------------|
| Patente         | 760 × 240     | 1520 × 480     |
| QSL Header      | 760 × 320     | 1520 × 640     |
| ID Card         | 680 × 400     | 1360 × 800     |
| Banner redes    | 760 × 280     | 1520 × 560     |

#### Patente Argentina (Mercosur)
- Fondo blanco, borde negro redondeado.
- Franja lateral azul (`#002B7F → #003DA5`) con estrellas Mercosur, texto
  "MERCOSUR" vertical y franjas de la bandera argentina al pie.
- "ARGENTINA" en azul centrado arriba + líneas decorativas.
- Indicativo en Oswald 700, tamaño auto-ajustado para no desbordar.
- Pie: "RADIOAFICIONADO · [ciudad] · [grid]".

#### QSL Header
- Fondo negro degradado, ondas de radio a los lados.
- Franjas doradas (`#BFA040`) superior e inferior con degradado transparente en extremos.
- "CQ CQ CQ DE [CS] CQ CQ CQ" en dorado tenue arriba.
- Indicativo en dorado con glow (shadowBlur 22).
- Nombre, QTH/Grid, DMR-ID/modos en capas de color decreciente.
- Campos QSO al pie: TO RADIO / DATE / TIME UTC / BAND FREQ / MODE / RST con líneas.

#### ID Card
- Panel izquierdo oscuro (`#0f172a → #1a3358`, 220px): indicativo en dorado con glow,
  "RADIOAFICIONADO" y país arriba, franjas de bandera argentina al pie.
- Panel derecho claro: nombre, QTH, país, grid, DMR-ID, modos en filas label+valor.
- Código de barras visual del DMR-ID al pie derecho (no escaneable, decorativo).
  Algoritmo: patrón start + cada dígito codificado en 4 barras + patrón stop.

#### Banner para redes sociales
- Ratio ~2.7:1 (compatible con Twitter/X header, Facebook cover).
- Fondo oscuro con starfield (80 estrellas, posiciones reproducibles por seed).
- Ondas de radio y antena yagi en silueta (decorativos).
- Franjas de bandera argentina con opacidad 35%.
- Indicativo grande con glow dorado, luego nombre, QTH, DMR-ID/modos.
- Watermark `mammoli.ar/callsign-card` en esquina inferior derecha.

### 3. Calculadora Grid & Zonas

#### Algoritmo Maidenhead (lat/lon ↔ grid)

```
// Lat/Lon → Grid 6 chars
lon += 180; lat += 90;          // normalizar a positivos
f_lon = floor(lon / 20)         // letra de campo (A–R)
f_lat = floor(lat / 10)
s_lon = floor((lon % 20) / 2)   // dígito de cuadrado (0–9)
s_lat = floor(lat % 10)
ss_lon = floor((lon % 2) * 12)  // letra de subcuadrado (a–x)
ss_lat = floor((lat % 1) * 24)
```

Verificación con LU2MCA (Ugarteche, Mendoza):
- lat: -33.10°, lon: -68.85° → FF56nv ✓

#### Zonas CQ/ITU (aproximación geográfica)
- Default: CQ 13, ITU 14 (mayoría de Sudamérica).
- Si lat < -40°: ITU 16 (Patagonia).
- Si lat > 12° y lon > -82°: CQ 9, ITU 12 (norte SA / Caribe).

#### Distancia y rumbo entre dos grids
- Convierte cada grid al centro de su celda.
- Distancia: fórmula de Haversine, resultado en km.
- Rumbo: `atan2(sin(dLon)·cos(lat2), cos(lat1)·sin(lat2) − sin(lat1)·cos(lat2)·cos(dLon))`.
- Rumbo en grados + etiqueta cardinal (N/NE/E/SE/S/SO/O/NO, 8 sectores de 45°).

---

## Historial

### v1.0 — 2026-05-03
- Patente Argentina y QSL Header.
- Deploy inicial a `mammoli.ar/callsign-card/`.
- Enlace desde `mammoli.ar/lu2mca/` sección "Enlaces útiles".

### v2.0 — 2026-05-03
- Agregados ID Card y Banner para redes sociales.
- Detección automática de país y provincia argentina por prefijo.
- Calculadora Grid & Zonas: Maidenhead bidireccional, zonas CQ/ITU, distancia + rumbo.
- Badge "AUTO" en campos auto-llenados; se desactiva al editar manualmente.
