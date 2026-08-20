# Módulo de Melany — Fase 1: Puntos 1-2 (Necesidades + Requerimientos) + Coordinación general

**Entrega 1 — 23 de agosto de 2026 (la más próxima del equipo).**

## Qué es tu módulo

La Fase 1 se dividió entre los 5 (cada uno con el punto más relacionado a lo que va a hacer después — ver [11-Equipo-y-Responsabilidades.md](../00-Documentacion-General/11-Equipo-y-Responsabilidades.md) §1.1). A vos te tocan:

1. **Puntos 1 y 2 como responsable directa**: [01-Analisis-Necesidades-Tecnologicas.md](../Fase-1-Diseno-Red-Corporativa/01-Analisis-Necesidades-Tecnologicas.md) y [02-Requerimientos-Costos-Trafico-Cultura.md](../Fase-1-Diseno-Red-Corporativa/02-Requerimientos-Costos-Trafico-Cultura.md) — el análisis de negocio que sostiene todo lo demás.
2. **Coordinación general de toda la Fase 1**: armás/mantenés [00-Introduccion-y-Metodologia.md](../Fase-1-Diseno-Red-Corporativa/00-Introduccion-y-Metodologia.md) (el índice) y das seguimiento a que los puntos 3-6 (de Jeferson, Luis, Samuel y Sergio) queden consistentes entre sí y con tus puntos 1-2.

Los puntos 3-6 (diseño lógico, físico, políticas de seguridad, Data Center) ya no son tuyos para escribir — son de Jeferson, Luis, Samuel y Sergio respectivamente (ver sus propios módulos) — pero como coordinadora sí te conviene tenerlos leídos, porque el día de la presentación probablemente seas quien presente la introducción y amarre el hilo conductor entre los 6 puntos.

## El contexto del caso (por si no lo tenés fresco)

Virtual Solutions: 184 usuarios en 4 pisos (Administración 14, Ventas 30, Desarrollo I/T 110, Soporte I/T 12, Servidores 12, Telefonía IP 6), Internet con 2 enlaces de 10 Mbps redundantes, Data Center exigido en Tier 4, y la red debe estar segmentada por VLAN por razones de seguridad.

## Decisiones clave de tu módulo y por qué se tomaron

### 1. Metodología: negocio → requerimiento → diseño (no al revés)
Los 6 documentos de la fase siguen un orden deliberado: primero se levantan las necesidades del negocio por stakeholder (tu doc. 01), luego se cuantifican como requerimientos técnicos con SLA objetivo (tu doc. 02), y solo entonces se diseña (docs. 03-06, de tus compañeros). Como coordinadora, tu trabajo es justamente cuidar que ese hilo no se rompa — si algo en el diseño de Jeferson o Luis no traza claramente hasta una necesidad que vos identificaste, es una señal de alerta a conversar con ellos antes de entregar.

### 2. Perfilamiento por stakeholder, no "la empresa" como bloque único
En vez de tratar a Virtual Solutions como un solo interesado homogéneo, el doc. 01 levanta necesidades por separado para Dirección, Administración, Ventas, Desarrollo I/T, Soporte I/T y empleados en general — cada uno con necesidades distintas y hasta en tensión entre sí (ej. Desarrollo quiere agilidad, Administración/Finanzas quiere control). Esta es la metodología estándar de un levantamiento de requerimientos real, no una lista de tecnologías inventada de antemano.

### 3. Priorización MoSCoW — por qué no todo es "urgente"
El doc. 01 clasifica cada necesidad como Must/Should/Could/Won't have — permite justificar, por ejemplo, por qué el proyecto sí implementa VPN y correo propio (Must have) pero deja el clúster redundante de Proxmox como roadmap futuro (Won't have por ahora) sin que se lea como una omisión no analizada. Si te preguntan "¿por qué no hicieron X?", la respuesta está en esa tabla.

### 4. Por qué el tráfico interno "supera" el ancho de banda de Internet (y por qué está bien)
Es el punto más contraintuitivo del doc. 02 — la demanda interna en hora pico (~321 Mbps) supera los 20 Mbps de Internet contratados, y **eso es correcto por diseño**: la mayoría del tráfico pesado (Git, CI/CD de Desarrollo) es tráfico este-oeste que nunca sale a Internet, y el concepto de sobre-suscripción (oversubscription) es estándar en cualquier diseño jerárquico — no todos los usuarios saturan su enlace simultáneamente. Detalle completo en doc. 02 §2.3.

### 5. Cultura organizacional — el punto que más se profundizó
El 66% de la plantilla es perfil técnico (Desarrollo + Soporte I/T), lo cual se traduce en decisiones concretas que aparecen en TODOS los demás documentos de la fase: por qué el proyecto puede exigir 100% open source sin generar resistencia de adopción, por qué el rigor de seguridad se aplicó de forma diferenciada por área (estricto en Administración/Servidores, más ágil en Desarrollo) en el doc. 05 de Samuel, y por qué la política de baja de accesos exige revocación el mismo día. Las 7 dimensiones completas con la tabla de síntesis "hallazgo cultural → decisión técnica" están en tu doc. 02 §4 — es probablemente el punto que más te va a distinguir en la defensa, porque casi nadie lo desarrolla más allá de 2-3 líneas, y como coordinadora es un buen punto para abrir tu presentación (explica el "por qué" de decisiones que después van a defender tus compañeros).

## Diagrama — el tuyo es 1, ya está 100% definido

A diferencia de la versión anterior de este documento, ya no tenés los 4 diagramas de la fase — cada quien pasa a visual el diagrama de su propio punto (Jeferson el lógico, Luis la planta y el rack). El tuyo:

**Diagrama general de la solución** — está en [00-Arquitectura-General.md](../00-Documentacion-General/00-Arquitectura-General.md) §2. Muestra Internet → Data Center (R1) → Nube Privada (SV1/VR1/VMs) → LAN corporativa — es el diagrama de apertura que amarra visualmente los 6 puntos de la fase con el resto del proyecto, apropiado para quien coordina. Pasalo a draw.io (Extras → Edit Diagram → pegar el Mermaid, te lo convierte a forma editable).

## Qué te toca hacer ahora

1. Leer tus 2 documentos (01 y 02) a fondo — sos su responsable directa.
2. Pasar tu diagrama (arquitectura general) a draw.io.
3. Leer los docs. 03-06 de tus compañeros (aunque no los escribiste) para poder coordinar la presentación conjunta y detectar inconsistencias a tiempo.
4. Armar/revisar el documento 00 (índice) para que quede como la introducción que amarra los 6 puntos.
5. Prepararte para presentar la apertura de la Fase 1 (contexto + metodología + cultura organizacional) y coordinar el orden en que cada quien expone su punto.
