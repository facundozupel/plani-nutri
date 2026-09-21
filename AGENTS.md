# AGENTS.md — COMIDA

Herramienta personal de una sola página para armar las comidas del día según el plan
nutricional de Facundo Zupel (Nta. Constanza Zúñiga Burnier, evaluación del 11-09-2026).

## Qué hay acá

| Archivo | Rol |
| --- | --- |
| `index.html` | **La app completa.** Un único HTML autocontenido: markup + `<style>` + `<script>`. Sin build, sin dependencias, sin red. |
| `Plan de alimentación Facundo Zupel.pdf` | Fuente 1: las 4 comidas, sus horarios, las porciones por grupo y las indicaciones generales. |
| `Porciones de intercambio Facundo Zupell .pdf` | Fuente 2: la tabla de intercambios (gramos y medida casera de **1 porción** de cada alimento). |

Los dos PDF son la **fuente de verdad**. El HTML no inventa datos: los transcribe y multiplica.

## Cómo se corre

Abrir `index.html` en el navegador (doble clic o `open index.html`).
No hay servidor, ni `npm`, ni tests. El estado de las elecciones se guarda en
`localStorage` bajo la clave `planFacundoExactoV3`.

> Si cambiás la forma de `state` (los índices de grupos, los ids de comida), **subí la
> versión de la clave** (`...V4`) para que el estado viejo no rompa el render.

## Modelo de datos (todo en el `<script>` final)

- `MASTER` — la tabla de intercambios, agrupada en `panes`, `frutas`, `carnes`,
  `aceites`, `lacteos`. Cada alimento es una tupla `[nombre, gramos, unidad, medida casera]`
  donde los gramos son los de **1 porción**.
- `MEALS` — las 4 comidas (`desayuno`, `almuerzo`, `colacion`, `cena`). Cada una tiene
  `groups`, y cada grupo declara `key` (a qué tabla de `MASTER` apunta) y `p` (cuántas
  porciones corresponden a ese grupo en ese horario).
- `EXTRAS_*` / `REEMPLAZOS_*` — opciones que la pauta permite pero que **no tienen
  equivalencia en gramos** en la tabla (Salmas, compota, proteína en polvo, mantequilla
  de maní). Van aparte, con su `sub` explicando por qué.
- `carnesFuertes()` / `panesFuertes()` / `aceitesBase()` — fábricas de los grupos que se
  repiten idénticos en almuerzo y cena. Si cambian las porciones de almuerzo o cena,
  se tocan acá una sola vez.

## Reglas del dominio (no negociables)

1. **La tabla de intercambios es la aritmética.** Cantidad mostrada = gramos de 1 porción
   × `p` del grupo. Nada de redondear "para que se vea lindo": 4p de pescado son 320 g y
   1½p de huevo son 112,5 g (3 huevos), aunque los ejemplos del PDF del plan muestren menos.
2. **Cuando los dos PDF no coinciden, gana la tabla de intercambios.** Esa decisión está
   documentada en el `<details class="audit">` al pie de la página; si se agrega otra
   discrepancia, se documenta ahí también.
3. **Lo que el PDF no cuantifica, no se estima.** Va como extra con su texto de
   excepción. Nunca inventar gramos.
4. **Meta diaria:** 4p panes y cereales · 2p frutas · 9½p carnes bajas en grasa ·
   4p aceites y grasas · 2p lácteos proteicos. Si editás las `p` de un grupo, recalculá
   esta línea del `<header>` y los `chips` de la comida afectada.
5. El aceite de cocción (1 cdta) **no** se suma automáticamente — punto abierto, pendiente
   de confirmar con la nutricionista. No lo agregues sin esa confirmación.

## Convenciones de código

- **Un solo archivo.** No separar en `.css` / `.js` ni agregar dependencias: la gracia es
  que el HTML se abra desde cualquier lado, incluso sin internet.
- JS plano, `const`/`let`, sin frameworks ni build step.
- Colores y radios salen de las variables CSS en `:root` (`--forest`, `--tomato`,
  `--mustard`, `--paper`…). No hardcodear hex nuevos.
- Todo el texto visible va en **castellano de Chile**, en el tono de la pauta: frases
  cortas, sin jerga técnica y sin voz de asistente.
- Números formateados con `Intl.NumberFormat("es-CL")` — coma decimal, punto de miles.
- Las fracciones se muestran con los glifos de `P_STR` (`½`, `1½`), no como `0.5`.

## Al modificar

- Si agregás un alimento, agregalo a `MASTER` con su gramaje de 1 porción; todos los
  horarios lo heredan solos.
- Si agregás una opción sin gramaje, va a un `EXTRAS_*` con `sub` explicando la excepción.
- Después de tocar el script: abrir la página, recorrer las 4 comidas, verificar que el
  progreso del día llegue a 4/4 y que "Copiar comida" arme el texto completo.
