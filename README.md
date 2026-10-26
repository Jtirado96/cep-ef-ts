# Excedente fiscal y tolerancia superficial · CEP CABA

Herramienta web para agrimensores que automatiza el **informe comparativo de excedente fiscal y tolerancia superficial** en trámites de **Constitución de Estado Parcelario (CEP)** en la Ciudad Autónoma de Buenos Aires.

A partir del **archivo de relevamiento (DXF)** y del **Certificado de Empadronamiento de AGIP (PDF)**, la página calcula las superficies, aplica la normativa, valida la parcela contra **Ciudad 3D** y genera el informe en PDF.

**Usar la herramienta:** https://jtirado96.github.io/cep-ef-ts/

---

## Qué hace

1. **Lee el DXF de relevamiento**: toma los polígonos de mensura, título y superficies por planta según sus capas y colores. Calcula las áreas teniendo en cuenta los tramos en arco, como las ochavas.
2. **Lee el Certificado de Empadronamiento de AGIP**: extrae dirección, partida, sección, manzana, parcela, superficie del terreno y unidades constructivas. Si el PDF es un escaneo, los datos se pueden cargar a mano.
3. **Valida la nomenclatura**: compara el SMP de la pestaña del layout del DXF con el del certificado. Si no coinciden, detiene el análisis y pide subir archivos de la misma parcela.
4. **Valida contra Ciudad 3D**: consulta la parcela en el catastro del GCBA, compara su georreferenciación con la del DXF y muestra los datos registrados.
5. **Terreno**: Mensura contra Título, según el art. 23 de la Ley 6.437 (excedente fiscal).
6. **Superficie edificada**: relevamiento contra empadronamiento AGIP, según el Título 5 del Reglamento Técnico (tolerancia superficial). Indica si corresponde declarar un alta o una baja ante AGIP.
7. **Genera el informe comparativo en PDF**, en una hoja A4.

## Cómo se usa

1. Abrí la página y subí el **archivo de relevamiento (.dxf)** y el **Certificado de Empadronamiento (.pdf)**.
2. Revisá las secciones que se despliegan: ficha de la parcela, ubicación y Ciudad 3D, terreno, superficie edificada y conclusión.
3. Si hace falta, corregí los datos del certificado desde **"Corregir datos del certificado"**.
4. Descargá el informe con **"Descargar PDF"**.

Para ver un caso resuelto sin subir archivos, usá **"Ver un ejemplo resuelto (Bogotá 3922)"**.

## Requisitos del DXF

El DXF debe respetar los niveles de información del documento **Detalle_Layers_Bloques v4** (GCBA, marzo 2025) y estar guardado en formato **DXF ASCII**.

| Capa | Uso |
|---|---|
| `09-M-PARCELA` | Polígono de mensura (SSM) |
| `14-M-TITULO` | Polígono de título (SST) |
| `10-M-MURO-SEPARATIVO-PARC` | Muros: si hay trazos, se considera límite cerrado por muros |
| `07-M-MANZANA` | Manzana (se usa en el mapa) |
| `M-M-PB-SUP`, `M-M-01P-SUP`, … `M-M-01S-SUP`, `M-M-01E-SUP` | Superficies por planta, clasificadas por color |
| `11-M-EXCEDENTE` | Debe quedar vacía en un CEP; si tiene elementos, se avisa |

- **Pestaña del layout:** debe llamarse con el formato `SSS-MMM-PPP` (por ejemplo `077-092-001b`).
- **Bloque `PARCELA_SURGENTE`:** su atributo `NPARC` también se verifica.
- **Coordenadas:** para ubicar el DXF en el mapa, debe estar en **CABA 2019 (POSGAR 2007, EPSG:9498)**.

### Clasificación por color en las capas `*-SUP`

| Color | Superficie | Cómputo |
|---|---|---|
| Rojo (1) | Cubierta | Construido |
| Amarillo (2) | Semicubierta | Construido (100 %) |
| Cyan (4) | Balcón semicubierto | Construido |
| 30 | Cuerpo saliente | Construido |
| 52 | Precaria | Construido |
| 104 | En contravención | Construido |
| Verde (3) | Descubierta | Informativa |
| 204 | Balcón descubierto | Informativa (suma a descubierta) |
| 200 / 220 | Superpuesta interior / exterior | No se computa |
| 32 | Vano a descontar | No se computa |
| 150 | Pileta | No se computa |

## Criterios que aplica

**Terreno (art. 23, Ley 6.437):**
- Tolerancia % = (SSM − SST) × 100 / SST, aplicable solo si SSM > SST.
- Límite cerrado por muros: **5 %**. Cerramiento no materializado o precario: **1 %**.
- Si se supera la tolerancia, se configura **excedente fiscal**. En un CEP su ubicación no se dibuja, porque se determina mediante mensura.

**Superficie edificada (Título 5 del Reglamento Técnico):**

| Superficie según certificado | Tolerancia |
|---|---|
| ≤ 500 m² | 5 % |
| > 500 m² y ≤ 5.000 m² | 3 % |
| > 5.000 m² | 2 % |

Si la diferencia supera la tolerancia: relevado mayor que AGIP → **alta**; relevado menor → **baja**.

**Georreferenciación (DXF vs Ciudad 3D),** según el desplazamiento entre los centros de las dos geometrías:
- hasta 1 m: coincide;
- hasta 3 m: desplazamiento leve;
- más de 3 m: desplazamiento mayor.

## Fuentes de datos

- **Catastro de Ciudad 3D (GCBA):** servicio `epok.buenosaires.gob.ar/catastro` (parcela y geometría por SMP). Los datos registrados de la parcela provienen de la fuente que informa el propio servicio, por ejemplo AGIP (2008).
- **Mapa:** fotografía aérea 2021 y Mapa BA del GCBA (USIG), según [datosgcba/tiles-gcba](https://github.com/datosgcba/tiles-gcba), y OpenStreetMap.
- **Normativa:** Ley 6.437 de Catastro de la CABA (art. 23), Título 5 del Reglamento Técnico y Detalle_Layers_Bloques v4.

## Privacidad

- **Los archivos no salen de tu computadora:** el DXF y el certificado se procesan en el navegador y no se envían a ningún servidor.
- **Consulta a Ciudad 3D:** se envía únicamente el SMP de la parcela.
- **Estadísticas de uso:** la página registra de forma anónima, sin cookies, las visitas y la cantidad de informes generados, mediante [GoatCounter](https://www.goatcounter.com/). No se registra ningún dato de los archivos ni de las parcelas.

## Tecnología

- **Página única:** HTML, CSS y JavaScript en un solo archivo (`index.html`), sin servidor propio. Se publica con GitHub Pages.
- **Lector de DXF y cálculo de superficies:** propios, con arcos (bulge) y conversión de coordenadas CABA 2019 ↔ WGS84.
- **Librerías:** [pdf.js](https://mozilla.github.io/pdf.js/) (lectura del certificado), [Leaflet](https://leafletjs.com/) (mapa), [html2canvas](https://html2canvas.hertzen.com/) y [jsPDF](https://github.com/parallax/jsPDF) (generación del PDF).

## Limitaciones

- **DXF binario:** no se admite; hay que guardarlo como DXF ASCII.
- **Certificados escaneados:** no se leen automáticamente; sus datos se cargan a mano.
- **Ciudad 3D:** la consulta y el mapa dependen de la disponibilidad de los servicios del GCBA.
- **Zoom del mapa:** la fotografía aérea del GCBA se publica hasta zoom 18.

## Aviso

Esta herramienta realiza un **control técnico preliminar**. No reemplaza la evaluación del agrimensor ni el dictamen del organismo de aplicación (DGROC / AGIP).

---

Desarrollado por **Ing. Agrim. Junior Tirado**.
