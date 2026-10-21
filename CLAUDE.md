# CLAUDE.md — Auditoría Final (lives-auditoria-final)

Las reglas generales de trabajo están en el repo privado `lives-instrucciones` (CLAUDE.md). Este archivo solo describe esta herramienta.

## Qué hace
Auditoría final de calidad (AQL) de un lote, desde el portal LIVES. Tres pantallas:
- `index.html` — **formulario** "Auditoría de Calidad". La auditora busca la OP, se llenan grupo, estilo, color, PO, destino y pedido, calcula la muestra AQL (1.0, 1.5 o 2.5) y registra defectos, derivados, medidas, reproceso, licencia y liquidado, con fotos. Incluye "Mis auditorías", donde se puede corregir, reprocesar, aprobar medida o anular. Funciona sin señal: guarda una cola en IndexedDB y la envía cuando vuelve la conexión.
- `buscar.html` — **consulta** "Buscar auditorías", con filtros, regla 500, Excel LIVES y enlace a `ver.html`. Si no se elige fecha, muestra los últimos 3 meses.
- `ver.html` — **reporte** de una auditoría (`?id=`): datos, defectos y fotos, con opciones para compartir en PDF (html2canvas + jsPDF) e imprimir.

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
