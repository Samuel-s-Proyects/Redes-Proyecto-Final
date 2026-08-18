# 10 — Diseño de Data Center (Tier 4, según estándares)

Estándares de referencia: **ANSI/TIA-942** (infraestructura de telecomunicaciones para Data Centers) y **Uptime Institute Tier Classification** (niveles de disponibilidad). El enunciado exige explícitamente **Tier 4**.

## 1. Qué exige Tier 4 (Uptime Institute)

| Criterio | Requisito Tier 4 |
|---|---|
| Disponibilidad anual | 99.995% (≈ 26.3 min de downtime/año) |
| Redundancia | **2N** (todo componente crítico duplicado y activo, no solo N+1) |
| Tolerancia a fallas | Tolerante a fallas — una falla no planificada de cualquier componente no debe causar downtime |
| Rutas de distribución | Múltiples rutas activas independientes de energía y enfriamiento |
| Compartimentación | Fault containment (una falla en una zona no se propaga a otra) |

## 2. Infraestructura física del cuarto de equipos

- **Piso elevado (raised floor)**: mínimo 60cm de altura para distribución de aire frío y cableado bajo piso, con baldosas perforadas frente a los racks (cold aisle).
- **Distribución en pasillo frío/pasillo caliente (cold/hot aisle containment)**: racks enfrentados por su cara fría, con contención física para evitar mezcla de aire frío/caliente — mejora significativamente la eficiencia del aire acondicionado.
- **Rack estándar 19"**, mínimo 42U, con PDUs redundantes (2 PDUs, una por cada rama eléctrica A/B).

## 3. Energía (2N — requisito de Tier 4)

- **2 acometidas eléctricas independientes** (idealmente de subestaciones distintas) alimentando 2 UPS independientes (rama A y rama B).
- **UPS on-line de doble conversión**, dimensionado según carga IT real + margen 30%. Cálculo de referencia:
  - Servidores/switches/router del Data Center: carga estimada ~3–5 kVA para el laboratorio; en producción con 12 servidores físicos/virtuales + red, estimar 8–15 kVA según equipo real cotizado.
  - UPS dimensionado en 2N significa **2 UPS**, cada uno capaz de soportar el 100% de la carga por sí solo.
- **Planta eléctrica (generador diésel)** con arranque automático (ATS — Automatic Transfer Switch) ante corte prolongado, autonomía mínima recomendada 8–24 horas con tanque de combustible dimensionado.
- **PDUs redundantes por rack**, cada servidor con doble fuente de poder conectada a rama A y rama B distintas.

## 4. Enfriamiento (2N)

- **Aire acondicionado de precisión (CRAC/CRAH)**: mínimo 2 unidades redundantes, cada una capaz de cubrir el 100% de la carga térmica sola (N+1 mínimo, 2N ideal para Tier 4).
- Temperatura objetivo: 18–27°C (rango ASHRAE recomendado), humedad relativa 40–60%.
- Sensores de temperatura/humedad distribuidos, integrados a Zabbix (ver documento 03) para alertar ante desviaciones antes de que afecten equipo.

## 5. Sistema de extinción de incendios

- **Detección temprana**: sistema de detección por aspiración (VESDA o equivalente) — detecta humo antes que un detector convencional.
- **Extinción**: agente limpio gaseoso (FM-200 o Novec 1230) — **no agua, no CO2 puro** (el agua daña el equipo; el CO2 puro desplaza oxígeno de forma peligrosa para el personal presente).
- Extintores portátiles de CO2 o agente limpio como respaldo manual, señalización y procedimiento de evacuación documentado.

## 6. Tierra física y protección eléctrica

- **Tierra física dedicada** para el Data Center, separada de la tierra general del edificio, con resistencia objetivo ≤5 ohms (norma NOM/IEEE recomendada para instalaciones críticas).
- **Malla de tierra** conectando racks, bandejas metálicas, PDUs y UPS.
- Protección contra sobretensión (supresores TVSS) en la acometida eléctrica.

## 7. Seguridad física (resumen — detalle en documento 09)

- Control de acceso biométrico o multifactor (tarjeta + PIN como mínimo).
- CCTV con grabación redundante (NVR con RAID, retención mínima 30 días).
- Detección de intrusión física (contactos en puertas, sensores de movimiento fuera de horario).

## 8. Aplicación al laboratorio de este proyecto

Para la demo académica **no es viable ni necesario** construir un Data Center Tier 4 físico real (2 UPS grandes + generador + CRAC redundante tiene un costo de decenas de miles de quetzales). El entregable de esta fase es el **diseño completo documentado** (este documento) más una **demostración simbólica** de los conceptos aplicables al laboratorio:

| Concepto Tier 4 | Cómo se demuestra en el laboratorio |
|---|---|
| Redundancia de energía | UPS pequeño (line-interactive, ~Q800–1500) protegiendo el servidor Proxmox y R1, documentado como "rama A" — se explica cómo escalaría a 2N en producción |
| Redundancia de conectividad WAN | Sí se implementa real: 2 ISP con failover en R1 (ver documento 03) |
| Redundancia de cómputo | Backups automatizados de VMs (Proxmox Backup/`vzdump`) como mitigación práctica de 1 solo nodo físico |
| Monitoreo ambiental | Opcional: sensor USB de temperatura barato integrado a Zabbix, o se documenta como diseño sin implementar físicamente |

Esta distinción (diseño completo para producción vs. demo de laboratorio) se deja explícita para que quede claro ante el catedrático qué se implementó realmente y qué se entregó como diseño de ingeniería.
