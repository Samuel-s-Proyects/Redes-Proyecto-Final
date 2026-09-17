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

### 1.1 Checklist de cumplimiento — punto por punto, contra los criterios reales de Tier IV

Esta tabla existe para responder explícitamente la pregunta "¿este diseño sí cumple Tier IV?" — criterio por criterio, con la referencia exacta a la sección de este documento que lo sostiene:

| Criterio Tier IV (Uptime Institute / ANSI-TIA-942-B) | ¿Se cumple? | Cómo se cumple en este diseño |
|---|---|---|
| **Tolerancia a fallas** — una falla no planificada de cualquier componente, en cualquier momento, no debe afectar la carga | ✅ Sí | 2N en energía (§3.3: 2 UPS on-line independientes, cada uno al 100% de la carga) y en enfriamiento (§4.2: 2 CRAC, cada uno al 100%) |
| **Mantenimiento concurrente** — se puede intervenir cualquier componente sin apagar la carga | ✅ Sí | Cada rama eléctrica (A/B) y cada unidad CRAC puede aislarse individualmente para servicio porque su gemela sostiene el 100% de la carga mientras tanto |
| **Múltiples rutas activas simultáneas** (no solo una activa + una en espera) | ✅ Sí | Las 2 ramas A/B trabajan **en paralelo activo todo el tiempo** — no es un esquema primario/respaldo, ambas alimentan carga real de forma constante (§3.3) |
| **Compartimentación** (fault containment — una falla en una zona no se propaga a otra) | ✅ Sí | Muros con resistencia al fuego ≥1h (§2); circuitos y PDU de la rama A físicamente separados de los de la rama B dentro del mismo rack (§3.3) |
| **Puesta a tierra y bonding dedicados** (TIA-607-C / J-STD-607-A) | ✅ Sí | TGB exclusiva del Data Center, ≤5Ω, bonding por soldadura exotérmica (§7) |
| **Detección y extinción de incendio sin dañar equipo activo** | ✅ Sí | Detección por aspiración (VESDA) + agente limpio gaseoso Novec 1230/FM-200, sin agua ni CO2 puro (§5) |
| **Control de acceso multifactor y anti-tailgating** | ✅ Sí | Tarjeta + PIN + biometría, mantrap de doble puerta (§6) |
| **Autonomía ante falla total de acometida** (batería + generador) | ✅ Sí | UPS 15-30 min de autonomía + generador con ATS, transferencia <15s (§3.4) |
| **Disponibilidad anual objetivo ≈99.995% (~26.3 min/año de inactividad)** | ⚠️ Objetivo de diseño | Se diseña para ese estándar (redundancia 2N en cada capa), pero **no es una certificación formal** — ver nota abajo |

**Nota importante — "diseñado conforme a Tier IV" no es lo mismo que "certificado Tier IV"**: Uptime Institute certifica instalaciones reales en **tres** etapas, todas pagadas y ejecutadas por auditores propios de Uptime Institute — *Tier Certification of Design Documents* (TCDD, revisión de planos), *Tier Certification of Constructed Facility* (TCCF, auditoría física del edificio ya construido) y *Tier Certification of Operational Sustainability* (TCOS, evalúa que la *operación* del día a día, no solo la infraestructura, sostenga el Tier declarado). Este documento cumple el estándar de **diseño de ingeniería** (topología, memoria de cálculo, criterios 2N en cada subsistema) — es exactamente lo que se puede entregar en un proyecto académico sin construir el Data Center físico. Ninguna de las tres certificaciones formales aplica ni se reclama aquí; lo que se sostiene es que **la arquitectura diseñada satisface los criterios técnicos que Uptime Institute exige para Tier IV**, punto por punto, como muestra la tabla anterior.

### 1.2 Terminología de redundancia — para no confundir "2N" con "Tier IV"

Un error común es usar "N+1" y "2N" como sinónimos, o asumir que cualquier sistema "2N" automáticamente es Tier IV. Uptime Institute distingue explícitamente ambos conceptos: la notación describe **cuántos componentes de respaldo existen**, el Tier describe **qué pasa operativamente cuando fallan** (tolerancia a fallas + mantenimiento concurrente simultáneos, ver §1). Un sistema puede ser "2N" en papel y no calificar para Tier IV si, por ejemplo, ambas rutas comparten un punto de transferencia único (un solo ATS, un solo tablero aguas arriba):

| Notación | Qué significa | Nivel de tolerancia típico |
|---|---|---|
| N | Solo los componentes necesarios para la carga, sin respaldo | Ninguno — cualquier falla afecta la carga |
| N+1 | Un componente de respaldo compartido entre todo el sistema | Tolera la falla de **un** componente, pero no durante mantenimiento de otro |
| N+2 | Dos componentes de respaldo compartidos | Mayor margen que N+1, sigue sin ser "ruta completa duplicada" |
| **2N** | **Dos sistemas completos e independientes**, cada uno capaz de cargar el 100% por sí solo | Es la base de Tier IV, pero **no es automáticamente Tier IV** — depende de que ambas rutas sean realmente independientes de principio a fin (sin puntos de convergencia compartidos) |
| 2(N+1) | Dos sistemas completos, cada uno con su propio N+1 interno | Redundancia adicional dentro de cada rama — típico en instalaciones hyperscale, exceden lo que este proyecto requiere |

Este diseño usa **2N** en energía (§3.3), enfriamiento (§4.2) y ahora también en Core/Distribución de red ([04-Diseno-Fisico.md](04-Diseno-Fisico.md) §1.1.1) — con rutas verificadamente independientes desde la acometida hasta el equipo (dos tableros distintos, no un tablero con dos breakers), que es precisamente el matiz que distingue un "2N de verdad" de un "2N de nombre".

### 1.3 Qué se puede afirmar y qué no — lenguaje permitido vs. sobre-reclamo

| ❌ No se puede afirmar aquí | ✅ Sí se puede afirmar aquí |
|---|---|
| "Data Center certificado Tier IV" | "Data Center **diseñado conforme a** los criterios técnicos de Tier IV" |
| "2N implica Tier IV" | "2N es la base necesaria de Tier IV, verificada con rutas independientes reales" |
| "99.995% de disponibilidad garantizada porque es Tier IV" | "99.995% es el objetivo de diseño asociado a Tier IV — una garantía real requeriría certificación TCOS y operación medida en el tiempo" |
| "Cumple Tier IV en toda la red" | "Cumple los criterios de Tier IV en el Data Center y en Core/Distribución de red; el acceso de piso usa redundancia simple por decisión de costo-beneficio (§1.1)" |

Esta distinción no es un tecnicismo — es la diferencia entre un documento de ingeniería defendible y uno que sobre-promete algo que nunca se auditó.

## 2. Ubicación y compartimentación

- El Data Center ocupa un espacio dedicado en el **Piso 2** — deliberadamente **no** en planta baja ni en sótano. Uptime Institute y BICSI 002 desaconsejan ambas ubicaciones para instalaciones críticas: la planta baja está más expuesta a inundación (escorrentía superficial, rotura de tubería municipal, cercanía a la calle) y a mayor tránsito/acceso no controlado (recepción, entregas); el sótano añade el riesgo de nivel freático y drenaje deficiente. El Piso 2 es el primer nivel elevado del edificio — reduce el riesgo de inundación sin llevar equipo pesado (rack 42U, UPS, futura planta eléctrica) más arriba de lo necesario.
- Sin ventanas al exterior, sin tuberías de agua ajenas al propio sistema de extinción/enfriamiento pasando por el cielo o piso del cuarto (requisito común de TIA-942/BICSI 002 para reducir riesgo de daño por agua).
- Muros con resistencia al fuego de al menos 1 hora (compartimentación — si hay un incendio en un área adyacente del edificio, el Data Center debe resistir su propagación el tiempo suficiente para actuar).
- Acceso único controlado (ver §6), sin rutas alternas de entrada no monitoreadas.

## 3. Energía — memoria de cálculo (2N)

### 3.1 Estimación de carga IT crítica

Para dimensionar el UPS hay que partir de una estimación realista de consumo del equipo que alimentará. Esta memoria de cálculo ya refleja el diseño final de switching (§1.1-§1.2 de [04-Diseno-Fisico.md](04-Diseno-Fisico.md)): **10 switches de acceso + 2 de distribución**, no el conteo preliminar de una versión anterior:

| Equipo | Cantidad | Consumo unitario estimado | Subtotal |
|---|---|---|---|
| Router Core (R1, redundante, MikroTik CCR2004-16G-2S+) | 2 | 30 W | 60 W |
| Switch de distribución (redundante, MikroTik CRS326-24S+2Q+RM — ver [04-Diseno-Fisico.md](04-Diseno-Fisico.md) §1.1.1) | 2 | 60 W | 120 W |
| Switches de acceso 48p PoE+ (MikroTik CRS354-48P-4S+2Q+RM) | 10 | 120 W | 1,200 W |
| Servidores físicos (host de virtualización, redundantes) | 2 | 450 W | 900 W |
| Almacenamiento (NAS/SAN si aplica) | 1 | 300 W | 300 W |
| PBX/gateway de telefonía IP (`vm-voip` + ATA) | 1 | 50 W | 50 W |
| **Total carga IT crítica estimada** | | | **≈ 2,630 W** |

**Nota sobre los 120 W/switch de acceso**: el CRS354-48P-4S+2Q+RM tiene una placa de potencia máxima de 700 W, pero ese número es el techo teórico con los 48 puertos entregando PoE+ a plena carga simultáneamente — algo que no ocurre en este diseño, donde solo 6 teléfonos IP en total (§1.3 de doc04) consumen PoE real y el resto de los ~330 puntos son datos puros. 120 W/switch es una estimación conservadora de operación típica (switching + un puñado de puertos PoE activos), no el máximo de placa; usar el máximo de placa (700 W × 10 = 7,000 W) sobredimensionaría el UPS varias veces por encima de la necesidad real.

### 3.2 Cálculo de capacidad del UPS

1. Convertir a VA usando un factor de potencia típico de UPS moderno (0.9): `2,630 W ÷ 0.9 ≈ 2,922 VA`.
2. Aplicar margen de crecimiento/seguridad del 30% (estándar de la industria para no operar un UPS al límite, lo que reduce su vida útil): `2,922 × 1.3 ≈ 3,799 VA`.
3. Redondear al tamaño comercial disponible más próximo: **UPS de 5-6 kVA** como unidad base — se elige 6 kVA porque es el tamaño confirmado disponible en el mercado guatemalteco (ver equipo real abajo) y deja margen adicional para crecimiento (nuevos servicios, un tercer host de virtualización) sin tener que re-dimensionar toda la rama eléctrica en el corto plazo.

### 3.3 Aplicación del criterio 2N — equipo real, no genérico

No se instala **un** UPS de 6 kVA — se instalan **dos, independientes** (rama eléctrica A y rama B), cada uno capaz de sostener el 100% de la carga por sí solo y trabajando **en paralelo activo** (no primario/respaldo, ambos entregan carga real todo el tiempo — es lo que exige el criterio de "múltiples rutas activas simultáneas" de Tier IV, ver §1.1).

| Elemento | Modelo real | Cantidad | Precio unitario (Guatemala) | Subtotal |
|---|---|---|---|---|
| UPS on-line doble conversión, rack 6U, 208V | **APC Smart-UPS 6kVA en Rack 6U 208V** | 2 (rama A + rama B) | **~Q40,113** ([Kemik.gt](https://www.kemik.gt/comprar/apc-smart-ups-6kva-en-rack-6u-208v)) | **~Q80,226** |
| PDU conmutada por rama, monitoreo remoto SNMP | **APC Rack PDU Switched AP7920** (1U, 8×C13, 208/230V) | 2 (una por rama, en el rack del Data Center) | ~Q3,500 – Q5,000 (estimado — confirmar cotización directa en Kemik/GBM antes de comprar) | ~Q7,000 – Q10,000 |
| Transferencia automática a nivel de rack para equipo de una sola fuente | **Sistema ATS para Rack APC 208V 30A** ([Kemik.gt](https://www.kemik.gt/comprar/sistema-ats-para-rack-marca-apc-208v-30a-l6-30p-en-entrada-l6-30r-de-salida)) | 1-2 (según cuántos equipos *single-corded* haya en el rack) | Cotizar directo en Kemik | — |

*Por qué el ATS de rack importa*: no todo equipo trae doble fuente de poder de fábrica (un switch de acceso barato, por ejemplo, suele traer una sola). Conectar ese equipo directo a una sola rama anularía el 2N para ese componente específico. El ATS de rack resuelve esto sin tener que comprar solo equipo *dual-corded* (más caro): recibe alimentación de la rama A y la rama B, y conmuta a la que esté viva en milisegundos si una falla — el equipo aguas abajo nunca nota la diferencia.

- **Topología**: 2 acometidas eléctricas independientes desde tableros distintos (no el mismo tablero con dos breakers — un solo tablero sigue siendo un punto único de falla) → 2 UPS on-line de doble conversión (aíslan completamente la carga de las variaciones de la red eléctrica de entrada, a diferencia de un UPS line-interactive) → 2 PDU por rack (A/B) → doble fuente en cada equipo crítico, o ATS de rack donde el equipo no la tenga de fábrica.
- **Compartimentación física**: el cableado de la rama A y el de la rama B se tienden por canalizaciones separadas dentro del rack — un daño físico (roedor, corto, error de instalación) que afecte un cable de la rama A no debe poder tocar el de la rama B.
- **Autonomía de batería objetivo**: 15–30 minutos a plena carga — tiempo suficiente para que arranque la planta eléctrica (30–60 segundos típico de arranque automático) con margen amplio, no para operar indefinidamente sin generador.

### 3.4 Planta eléctrica (generador)
- Generador diésel con **ATS (Automatic Transfer Switch)** a nivel de acometida general, que detecta la caída del suministro comercial y transfiere la carga automáticamente — tiempo de transferencia típico 10–15 segundos, cubierto sin interrupción por la autonomía del UPS (§3.3).
- **Dimensionamiento**: la carga IT crítica calculada (§3.1, ≈2,630 W ≈ 2.9 kVA) es solo una fracción de lo que el generador debe cubrir — en un Data Center Tier IV real el generador también respalda el propio sistema de enfriamiento (§4, que consume bastante más que el equipo de TI que enfría) y, típicamente, cargas del edificio con continuidad crítica (iluminación de emergencia, VoIP, control de acceso). Con ese criterio, un generador de referencia en el rango de **10-15 kVA** dedicado al Data Center (no al edificio completo, que es alcance de ingeniería eléctrica aparte y excede este documento) da margen realista para IT + HVAC + cargas de emergencia sin sobredimensionar innecesariamente. El dimensionamiento final de detalle (curva de arranque de los compresores del CRAC, factor de arranque de motores, etc.) requiere ingeniería eléctrica de detalle con firma de ingeniero colegiado, como cualquier instalación de esta naturaleza.
- Proveedores/distribuidores de generadores diésel con ATS en Guatemala para cotización directa: Kemik.gt, MacroCity Guatemala, DECA (Distribuidora Eléctrica Centroamericana) — el precio varía fuertemente según marca, nivel de insonorización y costo de instalación/obra civil, por lo que no se fija aquí una cifra única sin cotización directa.
- Autonomía de combustible objetivo: 8–24 horas continuas, con contrato de reabastecimiento de emergencia para eventos prolongados (práctica frecuente en diseños Tier IV reales).

## 4. Enfriamiento — memoria de cálculo (2N)

### 4.1 Carga térmica
Toda la energía eléctrica que consume el equipo de TI se convierte casi en su totalidad en calor. Conversión estándar: **1 W ≈ 3.412 BTU/hr**.

`2,630 W × 3.412 ≈ 8,974 BTU/hr` solo de carga IT. A esto se le suma la ganancia térmica del propio cuarto (personas, iluminación, envolvente del edificio) — para un cuarto de Data Center pequeño (~15–20 m²) se estima un adicional de ~4,500–6,000 BTU/hr por estos factores.

`Total ≈ 13,500–15,000 BTU/hr`, equivalente a **≈1.12–1.25 toneladas de refrigeración** (1 tonelada = 12,000 BTU/hr).

### 4.2 Selección de equipo y redundancia
- Se redondea hacia arriba a unidades comerciales disponibles: **2 unidades CRAC/CRAH de 2 toneladas cada una**, en configuración **2N** (cada unidad sola cubre el 100% de la carga calculada, con margen).
- **Distribución en pasillo frío/pasillo caliente (cold/hot aisle containment)**: los racks se orientan enfrentados por su cara fría, con contención física (cortinas o paneles) para no mezclar el aire frío de suministro con el caliente de retorno — mejora sustancialmente la eficiencia (menos BTU desperdiciados) y es la práctica estándar de BICSI 002 para cualquier Data Center moderno, independientemente del Tier.
- Rango objetivo de operación: **18–27 °C, 40–60% HR**, según las guías térmicas ASHRAE TC9.9 (clase A1, el rango recomendado para equipo empresarial estándar) — con alarma automática (Zabbix) ante desviación antes de que se vuelva crítico.

**Métrica de eficiencia — PUE (Power Usage Effectiveness)**: PUE = energía total consumida por el Data Center ÷ energía consumida solo por el equipo de TI. Un PUE de 1.0 sería un Data Center perfectamente eficiente (toda la energía va al equipo de cómputo, cero overhead de enfriamiento/distribución); en la práctica, el rango objetivo de diseño para una instalación de este tamaño con contención de pasillo frío/caliente (arriba) es **1.2–1.4** — es decir, por cada kW que consumen los equipos de TI, se consume entre 0.2 y 0.4 kW adicionales en enfriamiento y distribución eléctrica. Un PUE más bajo no solo reduce el costo operativo (OPEX) de energía, también reduce la carga térmica total que debe disipar el sistema de enfriamiento, con lo cual ambos objetivos (costo y disponibilidad) se refuerzan mutuamente en vez de competir.

## 5. Sistema de extinción de incendios

- **Detección temprana por aspiración (ASD — Aspirating Smoke Detection, ej. tecnología VESDA)**: el sistema aspira continuamente muestras de aire del cuarto y las analiza en un sensor central, detectando partículas de combustión **antes** de que haya humo visible o suficiente para activar un detector puntual convencional — crítico en un cuarto con flujo de aire forzado del HVAC, que dispersa y diluye el humo rápidamente.
- **Agente extintor limpio gaseoso**, conforme **NFPA 2001**: se especifica **Novec 1230** (concentración de diseño típica 4.5–5.9% v/v según la clase de riesgo) como primera opción por su perfil ambiental (bajo GWP) y de seguridad para el personal en concentración de diseño, con **FM-200** (concentración típica ~7–8.5% v/v) como alternativa equivalente. **No se usa agua** (dañaría irreversiblemente el equipo activo) **ni CO2 puro como agente de inundación total** (desplaza el oxígeno a niveles peligrosos para cualquier persona presente).
  - **Nota de disponibilidad a futuro (verificada agosto 2026)**: 3M anunció en 2022 su salida de la manufactura de PFAS ("forever chemicals") para finales de 2025, lo que incluye a Novec 1230 (comercialmente FK-5-1-12) — 3M dejó de tomar nuevos pedidos del producto, aunque honra contratos existentes. La química FK-5-1-12 en sí **no desaparece**: la fabrican también otros proveedores bajo otras marcas (ej. Fike SF 1230, Kidde Fluoro-K), compatibles con el mismo diseño de sistema y concentración. Implicación práctica para este proyecto: al momento de comprar, **confirmar qué fabricante de FK-5-1-12 está vigente** en vez de asumir 3M/Novec por nombre — el diseño (concentración, volumen a inundar) no cambia según el fabricante.
  - **Precaución de cálculo**: la masa de agente requerida se calcula sobre el volumen real del cuarto a inundar — **no se debe descontar el volumen ocupado por los racks como si fueran "volumen sellado"**, ya que el aire circula libremente a través de ellos; descontarlo subdimensiona el sistema.
- Extintores portátiles de agente limpio o CO2 como respaldo manual en puntos de acceso, señalización de evacuación, y procedimiento documentado de despresurización/ventilación post-descarga antes de reingresar al cuarto.
- Cumplimiento de **NFPA 75/76** en cuanto a resistencia al fuego de la envolvente del cuarto (ver §2) y separación de cargas combustibles.

## 6. Seguridad de acceso físico

- **Control de acceso multifactor**: tarjeta de proximidad + PIN como mínimo, con **biometría** (huella o reconocimiento facial) como estándar objetivo del diseño de producción — es lo esperado en una instalación certificable como Tier IV, donde el acceso no autorizado es en sí mismo un riesgo de disponibilidad (sabotaje, error humano).
- **Antesala de acceso único (mantrap/interlocking doors)**: dos puertas en serie que nunca se abren simultáneamente, forzando el paso de una persona a la vez y evitando el *tailgating* (que alguien no autorizado entre pegado a alguien que sí tiene acceso) — práctica típica en instalaciones Tier IV reales.
- CCTV, bitácora de acceso y gestión de visitantes: ver [05-Politicas-Seguridad.md](05-Politicas-Seguridad.md) §2.
- **Zonificación de acceso** — no todo el Data Center tiene el mismo nivel de restricción; zonificarlo evita que cualquier persona con acceso al edificio termine con acceso al rack de producción:

| Zona | Área | Quién entra |
|---|---|---|
| 0 | Exterior / recepción del edificio | Público general, con control de recepción |
| 1 | Piso 2 (área general de oficinas: Administración, Soporte I/T) | Personal de Virtual Solutions con credencial de piso |
| 2 | Antesala del Data Center (mantrap) | Solo personal con permiso explícito de Data Center |
| 3 | Sala de Data Center (racks, MDF) | Soporte I/T y personal técnico autorizado, biometría + PIN |
| 4 | Rack físico individual (si se usa cerradura de gabinete) | Solo la persona ejecutando el mantenimiento específico, registrado por ticket |

El acceso se otorga por rol (RBAC), no por persona individual — cuando alguien cambia de rol o se desvincula, se revoca el acceso a la zona correspondiente sin tener que reconstruir listas de personas caso por caso (ver política de baja inmediata en [05-Politicas-Seguridad.md](05-Politicas-Seguridad.md) §1.1).

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
