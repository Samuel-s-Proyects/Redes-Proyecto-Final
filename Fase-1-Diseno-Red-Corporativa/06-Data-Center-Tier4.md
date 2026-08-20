# Diseño de Data Center — Tier 4 (según estándares)

## 0. Estándares y marcos normativos aplicados

| Estándar / marco | Qué cubre en este diseño |
|---|---|
| **Uptime Institute Tier Standard: Topology** | Clasificación de disponibilidad (Tier I–IV) — define los criterios de redundancia y tolerancia a fallas que debe cumplir un Tier 4 |
| **ANSI/TIA-942-B** | Infraestructura de telecomunicaciones para Data Centers — arquitectura, cableado, redundancia, clasificación equivalente a los Tiers de Uptime |
| **ANSI/BICSI 002** | Guía de mejores prácticas de diseño y construcción de Data Centers (complementa TIA-942) |
| **NFPA 75 / NFPA 76** | Protección contra incendio de equipo de tecnología de la información / instalaciones de telecomunicaciones |
| **NFPA 2001** | Sistemas de extinción con agente limpio (gaseoso) |
| **ASHRAE TC9.9** | Guías térmicas y de humedad para equipo de TI (rango recomendado y permitido de temperatura/humedad) |
| **TIA-607-C / J-STD-607-A** | Puesta a tierra y bonding de infraestructura de telecomunicaciones |
| **IEEE 446 / NEC (o NOM homóloga en Guatemala)** | Diseño de sistemas de energía de respaldo (UPS, planta eléctrica) |

Este diseño se documenta a nivel de **ingeniería conceptual** (dimensionamiento, criterios, memoria de cálculo) — la ingeniería de detalle final (planos eléctricos certificados, cálculo estructural del piso, memoria de cálculo del sistema contra incendio) requeriría la firma de un ingeniero colegiado activo en las especialidades correspondientes antes de construirse, como en cualquier proyecto real de esta naturaleza.

## 1. Qué exige realmente un Tier 4 (y en qué se diferencia de un Tier 3)

La confusión más común es pensar que "Tier 4 = más UPS". Lo que realmente distingue a Tier 4 según Uptime Institute son **dos criterios simultáneos**, no solo redundancia de componentes:

| Criterio | Tier III | Tier IV |
|---|---|---|
| Mantenimiento concurrente (poder dar mantenimiento a cualquier componente sin afectar la carga) | ✅ Sí | ✅ Sí |
| **Tolerancia a fallas** (una falla no planificada de cualquier componente, en cualquier momento, no debe afectar la carga) | ❌ No garantizado | ✅ **Sí — este es el criterio que distingue a Tier IV** |
| Topología de distribución | Una ruta activa + una de respaldo | **Múltiples rutas activas simultáneas e independientes** |
| Compartimentación (fault containment) | No exigida | Exigida — una falla en una zona no debe propagarse a otra |
| Disponibilidad anual esperada | 99.982% (≈1.6 h/año de inactividad) | **99.995% (≈26.3 min/año)** |

En términos prácticos: en Tier III, si falla un componente mientras otro está en mantenimiento planificado, puede haber una interrupción. En Tier IV, el sistema debe sobrevivir una falla real **en cualquier momento**, incluso durante mantenimiento — por eso la redundancia tiene que ser **2N** (dos sistemas completos, cada uno capaz de cargar el 100%) y no simplemente N+1 (un componente de respaldo compartido).

## 2. Ubicación y compartimentación

- El Data Center ocupa un espacio dedicado en el **Piso 1**, sin ventanas al exterior, sin tuberías de agua ajenas al propio sistema de extinción/enfriamiento pasando por el cielo o piso del cuarto (requisito común de TIA-942/BICSI 002 para reducir riesgo de daño por agua).
- Muros con resistencia al fuego de al menos 1 hora (compartimentación — si hay un incendio en un área adyacente del edificio, el Data Center debe resistir su propagación el tiempo suficiente para actuar).
- Acceso único controlado (ver §6), sin rutas alternas de entrada no monitoreadas.

## 3. Energía — memoria de cálculo (2N)

### 3.1 Estimación de carga IT crítica

Para dimensionar el UPS hay que partir de una estimación realista de consumo del equipo que alimentará (ejemplo de memoria de cálculo para el rollout de producción — no para el laboratorio de demo, que consume una fracción de esto):

| Equipo | Cantidad | Consumo unitario estimado | Subtotal |
|---|---|---|---|
| Router Core (R1, redundante) | 2 | 30 W | 60 W |
| Switch de distribución | 1 | 150 W | 150 W |
| Switches de piso (IDF) | 3 | 100 W | 300 W |
| Servidores físicos (host de virtualización, redundantes) | 3 | 450 W | 1,350 W |
| Almacenamiento (NAS/SAN si aplica) | 1 | 300 W | 300 W |
| PBX/gateway de telefonía IP | 1 | 50 W | 50 W |
| **Total carga IT crítica estimada** | | | **≈ 2,210 W** |

### 3.2 Cálculo de capacidad del UPS

1. Convertir a VA usando un factor de potencia típico de UPS moderno (0.9): `2,210 W ÷ 0.9 ≈ 2,456 VA`.
2. Aplicar margen de crecimiento/seguridad del 30% (estándar de la industria para no operar un UPS al límite, lo que reduce su vida útil): `2,456 × 1.3 ≈ 3,193 VA`.
3. Redondear al tamaño comercial disponible más próximo: **UPS de 3 kVA** como unidad base.

### 3.3 Aplicación del criterio 2N
No se instala **un** UPS de 3kVA — se instalan **dos UPS de 3kVA independientes** (rama eléctrica A y rama B), cada uno capaz de sostener el 100% de la carga por sí solo. Cada equipo crítico (servidor, switch, router) debe tener **doble fuente de poder**, una conectada a cada rama, para que la falla de una rama completa (UPS, PDU o el propio breaker) no cause caída de servicio.

- **Topología**: 2 acometidas eléctricas independientes → 2 UPS on-line de doble conversión (aísla completamente la carga de las variaciones de la red eléctrica de entrada, a diferencia de un UPS line-interactive) → 2 PDU por rack (A/B) → doble fuente en cada equipo.
- **Autonomía de batería objetivo**: 15–30 minutos a plena carga — tiempo suficiente para que arranque la planta eléctrica (30–60 segundos típico de arranque automático) con margen amplio, no para operar indefinidamente sin generador.

### 3.4 Planta eléctrica (generador)
- Generador diésel con **ATS (Automatic Transfer Switch)** que detecta la caída de la acometida comercial y transfiere la carga automáticamente — tiempo de transferencia típico 10–15 segundos, cubierto sin interrupción por la autonomía del UPS.
- Dimensionado con margen sobre la carga total del edificio (no solo el Data Center — VoIP, iluminación de emergencia, HVAC del edificio dependen también de continuidad), lo cual excede el alcance de este documento de red y se remite a ingeniería eléctrica del edificio.
- Autonomía de combustible objetivo: 8–24 horas continuas, con contrato de reabastecimiento de emergencia para eventos prolongados (frecuente en diseños Tier IV reales).

## 4. Enfriamiento — memoria de cálculo (2N)

### 4.1 Carga térmica
Toda la energía eléctrica que consume el equipo de TI se convierte casi en su totalidad en calor. Conversión estándar: **1 W ≈ 3.412 BTU/hr**.

`2,210 W × 3.412 ≈ 7,541 BTU/hr` solo de carga IT. A esto se le suma la ganancia térmica del propio cuarto (personas, iluminación, envolvente del edificio) — para un cuarto de Data Center pequeño (~15–20 m²) se estima un adicional de ~4,500–6,000 BTU/hr por estos factores.

`Total ≈ 12,000–13,500 BTU/hr`, equivalente a **1–1.15 toneladas de refrigeración** (1 tonelada = 12,000 BTU/hr).

### 4.2 Selección de equipo y redundancia
- Se redondea hacia arriba a unidades comerciales disponibles: **2 unidades CRAC/CRAH de 2 toneladas cada una**, en configuración **2N** (cada unidad sola cubre el 100% de la carga calculada, con margen).
- **Distribución en pasillo frío/pasillo caliente (cold/hot aisle containment)**: los racks se orientan enfrentados por su cara fría, con contención física (cortinas o paneles) para no mezclar el aire frío de suministro con el caliente de retorno — mejora sustancialmente la eficiencia (menos BTU desperdiciados) y es la práctica estándar de BICSI 002 para cualquier Data Center moderno, independientemente del Tier.
- Rango objetivo de operación: **18–27 °C, 40–60% HR**, según las guías térmicas ASHRAE TC9.9 (clase A1, el rango recomendado para equipo empresarial estándar) — con alarma automática (Zabbix) ante desviación antes de que se vuelva crítico.

## 5. Sistema de extinción de incendios

- **Detección temprana por aspiración (ASD — Aspirating Smoke Detection, ej. tecnología VESDA)**: el sistema aspira continuamente muestras de aire del cuarto y las analiza en un sensor central, detectando partículas de combustión **antes** de que haya humo visible o suficiente para activar un detector puntual convencional — crítico en un cuarto con flujo de aire forzado del HVAC, que dispersa y diluye el humo rápidamente.
- **Agente extintor limpio gaseoso**, conforme **NFPA 2001**: se especifica **Novec 1230** (concentración de diseño típica 4.5–5.9% v/v según la clase de riesgo) como primera opción por su perfil ambiental (bajo GWP) y de seguridad para el personal en concentración de diseño, con **FM-200** (concentración típica ~7–8.5% v/v) como alternativa equivalente. **No se usa agua** (dañaría irreversiblemente el equipo activo) **ni CO2 puro como agente de inundación total** (desplaza el oxígeno a niveles peligrosos para cualquier persona presente).
- Extintores portátiles de agente limpio o CO2 como respaldo manual en puntos de acceso, señalización de evacuación, y procedimiento documentado de despresurización/ventilación post-descarga antes de reingresar al cuarto.
- Cumplimiento de **NFPA 75/76** en cuanto a resistencia al fuego de la envolvente del cuarto (ver §2) y separación de cargas combustibles.

## 6. Seguridad de acceso físico

- **Control de acceso multifactor**: tarjeta de proximidad + PIN como mínimo, con **biometría** (huella o reconocimiento facial) como estándar objetivo del diseño de producción — es lo esperado en una instalación certificable como Tier IV, donde el acceso no autorizado es en sí mismo un riesgo de disponibilidad (sabotaje, error humano).
- **Antesala de acceso único (mantrap/interlocking doors)**: dos puertas en serie que nunca se abren simultáneamente, forzando el paso de una persona a la vez y evitando el *tailgating* (que alguien no autorizado entre pegado a alguien que sí tiene acceso) — práctica típica en instalaciones Tier IV reales.
- CCTV, bitácora de acceso y gestión de visitantes: ver [05-Politicas-Seguridad.md](05-Politicas-Seguridad.md) §2.

## 7. Infraestructura estructural del cuarto

- **Piso elevado (raised floor)**: mínimo 60 cm de altura libre para distribución de aire frío y paso de cableado, con capacidad de carga puntual del orden de 1,000–1,250 kg/m² (rango típico exigido para racks cargados de equipo en un Data Center moderno) — cifra a confirmar con el fabricante de piso elevado elegido en la ingeniería de detalle.
- Piso con acabado disipativo de estática (control ESD), baldosas perforadas dimensionadas y ubicadas frente a cada rack según el patrón de pasillo frío.
- **Puesta a tierra dedicada (TIA-607-C / J-STD-607-A)**: barra de tierra de telecomunicaciones (TGB) exclusiva del Data Center, separada de la tierra general del edificio pero bonded (enlazada) a ella en el punto único exigido por norma para evitar diferencias de potencial; resistencia objetivo **≤ 5 Ω**, verificada con telurómetro y remedida periódicamente (la resistencia de una puesta a tierra se degrada con el tiempo por corrosión/humedad del terreno). Conexiones de bonding preferentemente por soldadura exotérmica (tipo Cadweld) en vez de solo mecánicas, para minimizar el aumento de resistencia por corrosión en el punto de unión.
- Protección contra sobretensión (TVSS) en la acometida eléctrica y, donde aplique, en las líneas de datos que salen del edificio.

## 8. Aplicación al laboratorio de este proyecto (alcance real vs. diseño de producción)

Construir un Data Center Tier 4 físico completo (2 UPS grandes, generador, 2 CRAC redundantes, mantrap biométrico) cuesta decenas de miles de quetzales y no es viable ni necesario para la demo académica. El entregable de esta fase es el **diseño de ingeniería completo** (documento presente) — la tabla siguiente deja explícito qué se demuestra realmente en el laboratorio y qué queda como diseño documentado para cuando Virtual Solutions ejecute el proyecto real:

| Componente del diseño Tier 4 | Tratamiento en el laboratorio de la Fase 4 |
|---|---|
| Redundancia de energía 2N | 1 UPS pequeño (line-interactive) protegiendo el servidor Proxmox y R1 — se documenta cómo escalaría a 2N en producción, no se construye |
| Redundancia de enfriamiento 2N | No aplica físicamente (laboratorio doméstico) — diseño documentado |
| Redundancia de conectividad WAN | **Sí se implementa real**: 2 ISP con failover en R1 |
| Redundancia de cómputo | Backups automatizados de VMs (Proxmox `vzdump`) como mitigación de tener 1 solo nodo físico |
| Control de acceso físico / mantrap | No aplica (laboratorio doméstico) — diseño documentado |
| Detección/extinción de incendio | No aplica (laboratorio doméstico) — diseño documentado |

Esta distinción se deja explícita para que, ante el catedrático, quede claro qué se implementó realmente (código, VMs, red funcionando) y qué se entregó como diseño de ingeniería con memoria de cálculo — que es exactamente lo que pide el enunciado en este punto ("Diseño de Data Center según estándares"), sin exigir su construcción física.
