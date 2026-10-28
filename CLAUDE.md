# CLAUDE.md — Auditoría Final (lives-auditoria-final)

Las reglas generales de trabajo están en el repo privado `lives-instrucciones` (CLAUDE.md). Este archivo solo describe esta herramienta.

## Qué hace
Auditoría final de calidad (AQL) de un lote, desde el portal LIVES. Tres pantallas:
- `index.html` — **formulario** "Auditoría de Calidad". La auditora busca la OP, se llenan grupo, estilo, color, PO, destino y pedido, calcula la muestra AQL (1.0, 1.5 o 2.5) y registra defectos, derivados, medidas, reproceso, licencia y liquidado, con fotos. Incluye "Mis auditorías", donde se puede corregir, reprocesar, aprobar medida o anular. Funciona sin señal: guarda una cola en IndexedDB y la envía cuando vuelve la conexión.
- `buscar.html` — **consulta** "Buscar auditorías", con filtros, regla 500, Excel LIVES y enlace a `ver.html`. Si no se elige fecha, muestra los últimos 3 meses.
- `ver.html` — **reporte** de una auditoría (`?id=`), con el formato de la hoja de papel (desde el 28/10/2026): Costura (taller) = FO-293 (AC-006-25), Acabados = FO-271 (AC-009-25), En proceso = FO-263 (AC-023-05, id que empieza con P, A4 horizontal). Solo salen los defectos registrados del catálogo (no la lista completa de la hoja); las fotos van aparte con "Ver fotos". Compartir PDF (una hoja A4, html2canvas + jsPDF) e imprimir.

## Base de datos (Supabase lives-datos)
- Guardar (`index.html`): `final_guardar_directo` (sirve para auditorías nuevas y para correcciones), `final_aprobar_medida` y `registro_anular_directo`.
  - Escriben en `auditorias`, `auditoria_defectos`, `auditoria_colores`, `auditoria_auditoras`, `auditoria_destinos` y `fotos_auditoria`.
- Leer (vistas):
  - Catálogos: `publico_ref_ops` (sale de la tabla `ops`), `publico_ref_personal`, `publico_ref_talleres`, `publico_ref_clientes`, `publico_ref_lineas`, `publico_ref_destinos` y `publico_ref_defectos` (salen de las tablas `cat_*`).
  - Auditorías: `publico_vista_piloto` (de `auditorias`), `publico_detalle_piloto` (de `auditoria_defectos`) y `publico_fotos_auditoria`, que cuenta las fotos.
  - `ver.html` además lee directo la tabla `fotos_auditoria`, para encontrar dónde quedaron guardadas las fotos antiguas que estaban en Drive.
- Fotos: bucket privado `auditorias-fotos`. Se suben a `nuevas/<fecha>/<id_envio>/` y se ven con enlaces firmados que vencen en 1 hora.
- La base de OPs (`ops`) se actualiza en `lives-registro-auditorias/ops.html`.
- Las mismas tablas `auditorias*` las usa también `lives-registro-auditorias/registro.html` (tipo FINAL y otros).

## Quién la usa
Según las entradas por el portal de los últimos 30 días al 21/10/2026: 27 personas. Son sobre todo auditoras de calidad (20) y supervisoras de calidad (3); también entran la digitadora de Calidad Manufactura y una analista de Planeamiento.

## Quién puede guardar
- La auditora sale de la sesión del portal y no se puede cambiar. Para guardar, tiene que estar en el catálogo de personal (`cat_personal`).
- Cada auditora anula solo sus propias auditorías.
- A Sistemas: aquí va su control de roles (ver o registrar).

## Pendientes vistos al escribir esto (21/10/2026)
- `index.html` tiene `PORTAL_URL` apuntando al portal antiguo (`lives-portal`). Revisar si se muestra en algún lado.
- Quedan nombres heredados del Apps Script (`gviz()`, `leerHojaScript()`) que en realidad solo leen Supabase.

## Cambios del 28/10/2026 (pedido de Christian)
- **Formulario por pestañas** (`index.html`): arriba fijos "¿Corregir una auditoría?" (plegado hasta tocar su título) y una tarjeta con Proceso, Taller y OP que se reduce a una línea con **Cambiar** al elegir la OP; un resumen de una línea queda pegado sobre las pestañas. Pestañas: Lote, Defectos, Medidas, Fotos y Cierre (con ‹ Anterior / Siguiente ›). Al guardar con faltantes salta a la primera pestaña con error y marca con punto rojo las que tienen (`calcularFaltantes`, mapeo en `tabDe`). Los scripts `finTabsJs` y `finProcesoJs` van antes del bloque lives-tema; las tarjetas se asignan con `data-tab`, sin tocar la lógica de guardado.
- **Ventana del buscador** (`?modal=1`): el formulario avisa `sucio`, `guardado` y `cerrar` (mensajes `lsFin`) al buscador; tras guardar ofrece "Registrar otra" y "Cerrar".
- **Buscador** (`buscar.html`): botón **Registrar** (abre el formulario en una ventana; activo solo para auditoras activas del catálogo —rol AUDITORA o AUDITORA E INSPECTORA—, Christian siempre y Rodrigo nunca; fuera del portal queda gris), botón **Dashboard** (abre `lives-dashboards-calidad/final/dashboard-final.html`, o `proceso/dashboard-proceso.html` si el filtro Proceso es "En proceso (costura)") y **Ver PDF** en cada fila (abre el reporte en la ventana; tocar la fila también). Sin la nota de arriba. La entrada del portal sigue siendo el formulario (decisión de Christian).
- **Opción "En proceso (costura)"** (hoja FO-263): una opción más del selector Proceso, en el mismo formulario. Pestañas: Lote, Paquetes, Fotos y Cierre. Una fila por paquete (N° paquete, talla, cantidad del paquete e inspeccionada con la muestra sugerida por la tabla AQL hasta 51-90, operación, defectos del catálogo de Costura con cantidad, y **A/R que marca la auditora** en la 1ra y la 2da auditoría; el resultado del paquete es el de la 2da si existe). Nivel AQL 1.0/2.0/2.5/4.0. Fotos de la hoja y del proceso al almacén `auditorias-fotos`. Si no hay señal queda en la cola del equipo (IndexedDB `lives_proc_pend`) y se envía sola. **Corregir** (`index.html?proc=ID`) y **Anular** desde su reporte, solo quien la registró (y Christian). Borrador en el equipo (`lives_fin_proc_borrador`).
- **Base de datos de Proceso** (registro `Fp01 final proceso fo263`): tablas `fproc_auditorias`, `fproc_paquetes` y `fproc_defectos`, aparte de `auditorias` a propósito (no entra en el dashboard de Final ni en Desempeño de Personal), y puerta única `fproc_api(p_accion, p)` con `inicio`, `guardar`, `ver`, `anular` y `datos` (lista para el buscador y el dashboard; `detalle:true` trae paquetes y defectos). Permiso: `fproc_puede`. Las tablas tienen RLS cerrada: solo se entra por `fproc_api`. El buscador suma sus filas a la lista (etiqueta "En proceso").
- A Sistemas: `fproc_puede` es la pieza que reemplazan con su seguridad; en SQL Server la herramienta exige usuario para abrir y leer. Exclusivo de Postgres: jsonb en la puerta y RLS.
- Pendiente: la tabla AQL de la hoja de proceso solo se leyó hasta el lote 51-90 (se muestra así); la hoja trae más filas si Calidad las confirma. La hoja de Costura (FO-293) tiene una tabla de tallas que la base no guarda por talla y no se reproduce.
