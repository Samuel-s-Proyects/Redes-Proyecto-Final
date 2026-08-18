# Módulo de Melany — Fase 1: Diseño de la Red Corporativa

**Entrega 1 — 23 de agosto de 2026 (la más próxima del equipo).**

## Qué es tu módulo

Sos la responsable de la **Fase 1 completa**: el diseño de la red de Virtual Solutions antes de que se implemente nada — el "por qué" técnico detrás de toda la propuesta. Esto incluye 3 documentos que son tuyos:

1. [Fase1-Diseno-Red-Corporativa.md](../Fase-1-Diseno-Red-Corporativa/01-Fase1-Diseno-Red-Corporativa.md) — el documento principal
2. [Politicas-Seguridad.md](../Fase-1-Diseno-Red-Corporativa/09-Politicas-Seguridad.md) — políticas lógicas y físicas
3. [Data-Center-Tier4.md](../Fase-1-Diseno-Red-Corporativa/10-Data-Center-Tier4.md) — diseño del Data Center según estándares

Vas a ser quien defienda esta fase ante el catedrático, así que la idea de este documento es que entendás no solo **qué** se decidió sino **por qué**, para que puedas responder preguntas con seguridad.

## El contexto del caso (por si no lo tenés fresco)

Virtual Solutions: 184 usuarios en 4 pisos (Administración 14, Ventas 30, Desarrollo I/T 110, Soporte I/T 12, Servidores 12, Telefonía IP 6), Internet con 2 enlaces de 10 Mbps redundantes, Data Center exigido en Tier 4, y la red debe estar segmentada por VLAN por razones de seguridad.

## Decisiones clave de tu módulo y por qué se tomaron

### 1. Distribución por piso
Se puso el Data Center/MDF en el **Piso 1** (no en un piso intermedio) para minimizar la longitud del backbone vertical de fibra hacia los demás pisos. Desarrollo I/T (110 personas, el área más grande) se dividió en **2 IDFs** (piso 3 y piso 4, 55 cada uno) — no porque no quepan en un solo rack, sino porque la norma TIA/EIA-568 recomienda no superar 90-100m de cableado horizontal por punto de distribución, y concentrar 110 puestos en un solo IDF generaría demasiados cables largos y un rack sobrecargado.

### 2. Por qué VLAN por área (la justificación que más te van a preguntar)
El enunciado pide separar "ingeniería" de "administración" y que vos definas el criterio. Los 5 argumentos que armamos (están completos en [00-Arquitectura-General.md](../00-Documentacion-General/00-Arquitectura-General.md) sección 3):
- **Superficie de ataque**: si comprometen una PC de Desarrollo (que tiene más privilegios técnicos), un VLAN separado evita que salte directo a Administración/Finanzas.
- **Perfil de tráfico distinto**: Desarrollo genera tráfico pesado interno (builds, CI); Administración/Ventas es más liviano — mezclarlos degrada a todos.
- **Cumplimiento**: la empresa maneja "recursos monetarios" (dato explícito del enunciado) — Administración necesita controles más estrictos.
- **Contención de broadcast**: con 184 dispositivos, un solo dominio de broadcast ya es mala práctica.
- **VoIP necesita su propio VLAN** por calidad de servicio (QoS/jitter), no se puede mezclar con datos.

### 3. Cableado: fibra en el backbone, Cat 6 en el horizontal
Fibra OM4 multimodo para las corridas verticales (MDF↔IDF) por ser más resistente a distancia e interferencia; Cat 6 en el horizontal porque soporta 1 Gbps y PoE (necesario para alimentar los teléfonos IP sin cableado eléctrico aparte) — y porque el enunciado ya exige Cat 6 como mínimo para la conexión R1↔VR1 en la Fase 4, así que se estandarizó en todo el diseño para no mezclar categorías de cable sin necesidad.

### 4. Data Center Tier 4 — qué significa realmente
No es solo "ponerle UPS" — Tier 4 exige **2N** (todo componente crítico duplicado y activo, no solo de respaldo): 2 acometidas eléctricas, 2 UPS, 2 sistemas de enfriamiento, rutas de distribución independientes. En el documento 10 vas a ver también la distinción entre **lo que se diseña en papel** (Tier 4 completo, para cuando Virtual Solutions lo construya de verdad) y **lo que se demuestra en el laboratorio** (un UPS pequeño, backups automatizados) — no vamos a construir un Data Center Tier 4 real, pero el diseño documentado sí tiene que estar completo y correcto.

### 5. Presupuesto
Los materiales y precios del rollout de producción completo (switches de piso, fibra, racks, UPS, teléfonos IP) están en [Equipo-Fisico-Presupuesto.md](../00-Documentacion-General/08-Equipo-Fisico-Presupuesto.md) sección 4 — ese documento es de Luis, pero como tu Fase 1 lo referencia, coordiná con él si algo del diseño físico cambia (afecta directamente su lista de materiales).

## Diagramas — ya están 100% definidos, solo falta pasarlos a una herramienta visual

Vos sos la persona con más diagramas pendientes para esta entrega. Los tres ya existen como diagramas de código (Mermaid) dentro de los documentos — **no hay ninguna decisión de diseño pendiente**, es puramente trabajo de convertirlos a una herramienta visual profesional (recomendado: **draw.io / diagrams.net**, es gratis y tiene librería de íconos de red y de racks). Podés copiar la estructura exacta de cada Mermaid, no hay que inventar nada nuevo:

1. **Diagrama general de la solución** — está en [00-Arquitectura-General.md](../00-Documentacion-General/00-Arquitectura-General.md) sección 2 (bloque ```mermaid). Muestra Internet → Data Center (R1) → Nube Privada (SV1/VR1/VMs) → LAN corporativa.
2. **Diagrama de planta por piso** — está en [Fase1-Diseno-Red-Corporativa.md](../Fase-1-Diseno-Red-Corporativa/01-Fase1-Diseno-Red-Corporativa.md), sección 4.1, justo después de la tabla de distribución por piso. Muestra los 4 pisos con sus áreas, IDFs, y el backbone de fibra conectándolos al MDF.
3. **Elevación de rack del Data Center** — está en la misma sección 4.2, como tabla (no Mermaid) — esta la tenés que convertir en un diagrama de rack visual (draw.io tiene plantillas de "rack" listas), respetando el orden de posiciones U que ya está definido en la tabla.

**Tip práctico**: en draw.io podés pegar el código Mermaid directo (Extras → Edit Diagram → pegar el mermaid) y te lo convierte automáticamente a forma editable — desde ahí solo le das estilo visual, no tenés que redibujar de cero.

## Qué te toca hacer ahora

1. Leer tus 3 documentos completos.
2. Pasar los 3 diagramas/elevación de rack a draw.io (o la herramienta que prefieras).
3. Avisar si algo del diseño no te convence — todavía hay tiempo antes del 23 de agosto para ajustar contenido, no solo forma.
4. Prepararte para defender el "por qué" de la segmentación VLAN y del Tier 4 — son los dos puntos que más suelen preguntar en este tipo de proyecto.
