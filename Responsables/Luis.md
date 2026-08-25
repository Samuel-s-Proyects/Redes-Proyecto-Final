# Módulo de Luis — Fase 1 Punto 4 (Diseño Físico) + Core Físico (Fase 4) + Presupuesto y Equipo

**Entrega 1 — 23 de agosto de 2026 (tu punto de Fase 1) — Entrega 3 — 17 de octubre de 2026 (tu Fase 4), pero la COMPRA de equipo no puede esperar hasta esa fecha.**

## Qué es tu módulo

Tu Fase 1 y tu Fase 4 son la misma disciplina (físico + materiales + presupuesto) aplicada en dos escalas distintas: primero a toda la empresa (184 usuarios, diseño de producción) y después al laboratorio de demo (ver por qué te tocó a vos en [11-Equipo-y-Responsabilidades.md](../00-Documentacion-General/11-Equipo-y-Responsabilidades.md) §1.1). Documentos que son tuyos:

1. **Fase 1, Punto 4 — Diseño Físico** (Entrega 1, la más próxima): [04-Diseno-Fisico.md](../Fase-1-Diseno-Red-Corporativa/04-Diseno-Fisico.md) — planta, cuartos de telecomunicaciones, cableado estructurado, rack, BOM con memoria de cálculo.
2. **Fase 4, parte física** (Entrega 3): [Fase4-Nube-Privada-SDN.md](../Fase-4-Nube-Privada-SDN/04-Fase4-Nube-Privada-SDN.md) — secciones 3.6 y 3.7 (Core físico, switch, host de prueba) y sección 6 (checklist de pruebas de conectividad).
3. **El presupuesto completo del proyecto**, ambas escalas: [Equipo-Fisico-Presupuesto.md](../00-Documentacion-General/08-Equipo-Fisico-Presupuesto.md).

## Tu punto de Fase 1 (Punto 4 — Diseño Físico) — es tu entrega más próxima

[04-Diseno-Fisico.md](../Fase-1-Diseno-Red-Corporativa/04-Diseno-Fisico.md) es el diseño físico completo de los 184 usuarios en 4 pisos — distinto del laboratorio de demo de Fase 4 (que solo conecta 2 hosts de prueba). Es, literalmente, la versión a escala completa de lo que después vas a comprar en miniatura para la Fase 4.

### Decisiones clave de este punto y por qué se tomaron

**Dimensionamiento de cuartos de telecomunicaciones (TR/IDF) con norma, no al ojo**: TIA-569-D exige que el tamaño de cada IDF se calcule en función de los puestos que atiende, no un tamaño arbitrario — doc. 04 §3 tiene la memoria de cálculo completa (área servida → dimensión mínima del cuarto) para los 3 IDF de piso.

**Memoria de cálculo del cableado — la corrección más importante que vas a defender**: la primera versión del presupuesto estimaba "6 cajas" de cable UTP al ojo, un número irreal para 184 usuarios. Se rehizo con metodología explícita, y pasó por varias correcciones sucesivas — cada una real, con causa identificada, no un ajuste cosmético: (1) memoria de cálculo de cableado con 405 drops (184 usuarios × 2, sin distinguir puestos de servidores/teléfonos) → ~Q88,765; (2) switches dimensionados por conteo real de puertos → ~Q120,965 con 6+1 switches; (3) adoptando la regla del equipo (2 drops/puesto) → 10 switches de acceso, ~Q146,565; (4) **corrección final, la que vas a defender ahora**: el conteo de drops se recalculó sobre 166 puestos de trabajo reales (no 184 — los servidores se cablean directo al rack, los teléfonos comparten el drop del puesto), dando 366 drops/60 cajas (antes 405/66, un ajuste **a la baja**); en paralelo se cerró un punto único de falla que quedaba documentado como pendiente (switch de distribución duplicado a 2, ver doc. 04 §1.1.1) y se corrigieron los precios de switches/router a los **precios reales verificados en tienda** (Pacifiko.com, no una review internacional sin margen de importación) — el neto de estas dos fuerzas (cable más barato, pero switches más caros y ahora redundantes) deja el total en **~Q186,391**, la cifra final. Detalle completo en doc. 04 §8 y §1.2, que es la misma memoria de cálculo que sostiene tu [08-Equipo-Fisico-Presupuesto.md](../00-Documentacion-General/08-Equipo-Fisico-Presupuesto.md) §4.

**Parámetros reales de certificación del cableado**: no basta con que el cable "parezca funcionar" — se exigen 4 parámetros medidos con certificadora (NEXT, Insertion Loss, Return Loss, Delay Skew) conforme TIA-568-C.2, detallados en doc. 04 §4.3. Si te preguntan cómo se garantiza una instalación de 400+ puntos bien hecha, la respuesta es esa.

### Diagramas de este punto — ya están definidos, solo falta pasarlos a visual

Ahora **sí tenés 2 diagramas propios** (antes eran de Melany, se reasignaron porque el contenido físico es tu punto):

1. **Diagrama de planta por piso** — doc. 04 §2. Muestra los 4 pisos con sus áreas, IDFs, y el backbone de fibra conectándolos al MDF.
2. **Elevación de rack del Data Center** — doc. 04 §5, como tabla (no Mermaid) — convertila en un diagrama de rack visual en draw.io (tiene plantillas de "rack" listas), respetando el orden de posiciones U ya definido.

Pasalos a draw.io esta semana — son tu entrega más próxima, antes incluso que comprar el equipo de Fase 4.

## Decisiones clave de tu módulo de Fase 4 y por qué se tomaron

### 1. Por qué comprar equipo físico real y no usar GNS3
El enunciado permite GNS3 para el Core "si no cuenta con equipo físico" — pero comprar es más barato de lo que parece (~Q600 el router) y evita meterse en el problema de "puentear" una red virtual (GNS3) con la red real de Proxmox, que es una fuente típica de horas perdidas debuggeando para alguien que recién empieza en esto. Con hardware real, conectás un cable y listo — la demo también se ve más convincente con tráfico real entre fierro real.

### 2. Por qué MikroTik hEX (RB750Gr3) para R1
Es la opción de mejor relación costo/funcionalidad en Guatemala (~Q585-625): soporta OSPF (obligatorio, no se aceptan rutas estáticas), VLANs 802.1Q, firewall stateful — todo lo que RouterOS necesita para cumplir el rol de Core, a una fracción del costo de un equipo Cisco equivalente.

### 3. Por qué el switch SÍ es administrable con VLAN (esto cambió dos veces, importante que lo tengas claro)
Historial completo por si el catedrático pregunta:
1. Primero se propuso un MikroTik administrable con VLAN (~Q750-1,200).
2. Después, al releer el enunciado literal ("1 switch físico para conectar clientes", sin mencionar VLAN ahí), se bajó a un **TP-Link TL-SG105 no administrable** (~Q214) para ahorrar — cumplía la letra del requisito.
3. **Se revirtió esa segunda decisión**: aunque el PDF no lo exige literalmente, sin VLAN en el switch físico no se puede demostrar en vivo la segmentación por departamento — que es el hilo conductor de todo el proyecto (Fase 1 la propone, Fase 3 la direcciona). Se corrigió a un **TP-Link Easy Smart con VLAN 802.1Q** (~Q318) — sigue siendo barato, y ahora sí sostiene un puerto trunk hacia R1 más puertos de acceso por VLAN.

⚠️ Ojo al comprar: **`TL-SG105` y `TL-SG105E`/Easy Smart son productos distintos**, no el mismo switch con un modo oculto. El que hay que comprar es el que trae VLAN — confirmar el modelo exacto en la tienda antes de pagar.

### 4. Por qué no se compra un mini PC dedicado
En vez de eso, el "servidor" es un **SSD externo portátil** que se mueve entre la laptop de Samuel (desarrollo, 8GB RAM) y una laptop prestada de 16GB el día de la demo. Esto evita gastar Q1,800+ en un mini PC — el detalle completo de por qué y cómo está en [Terraform-Ansible-IaC.md](../00-Documentacion-General/05-Terraform-Ansible-IaC.md) sección 8, pero como dueño del presupuesto te conviene entender el ahorro: de un total inicial estimado de Q4,000-5,600 (con mini PC nuevo) se bajó a **Q1,480-1,890 total** (subió un poco respecto a la primera versión del presupuesto por el cambio de switch del punto 3, pero sigue siendo una fracción del costo original).

### 5. Por qué OSPF y no otro protocolo
El enunciado deja el protocolo "a discreción" pero prohíbe rutas estáticas. OSPF se eligió porque lo soportan bien tanto RouterOS (tu lado, R1) como VyOS (lado de Sergio, VR1), es más simple de configurar y depurar que BGP para una topología de 2 routers, y es exactamente lo que se espera demostrar en un curso de Redes 1.

## Presupuesto — ya decidido, resumen para que lo tengas a mano

| Ítem | Precio | Quién lo compra/gestiona |
|---|---|---|
| Router MikroTik hEX RB750Gr3 | ~Q600 | Vos |
| Switch TP-Link Easy Smart (VLAN 802.1Q, 4p PoE) | ~Q318 | Vos — confirmar modelo exacto (no el TL-SG105 simple) |
| Cable UTP Cat 6 (patch cords) | ~Q100-150 | Vos |
| SSD externo + enclosure + adaptador USB-Ethernet | ~Q480-800 | Coordinado con Samuel (él lo usa a diario) |
| **Total** | **~Q1,480-1,890** | **Partes iguales entre los 5** (~Q296-378 c/u, ya decidido) |

## Diagrama de tu módulo de Fase 4

Para la Fase 4 no tenés un diagrama propio adicional que crear — tu parte física ya está representada dentro del diagrama de topología de Sergio (con 2 hosts de prueba en VLANs distintas). Tu tarea ahí es de **validación, no de creación**: cuando Sergio pase su diagrama a draw.io, revisá que el lado físico (R1, switch, cableado) quede exactamente como vos lo vas a instalar de verdad — si compraste un modelo distinto o cambiaste algún puerto, avisale para que lo ajuste.

## Qué te toca hacer ahora

**Para el 23 de agosto (urgente, tu Fase 1):**
1. Revisar/pulir tu [04-Diseno-Fisico.md](../Fase-1-Diseno-Red-Corporativa/04-Diseno-Fisico.md) — es tu entrega más próxima.
2. Pasar los 2 diagramas (planta por piso, elevación de rack) a draw.io.
3. Confirmar con Melany y Jeferson que tu memoria de cálculo (366 puntos, 60 cajas) sigue cuadrando con lo que ellos definieron en sus puntos.

**Para el 17 de octubre (tu Fase 4, con más tiempo pero no lo dejes para lo último):**
4. Coordinar la compra del equipo del laboratorio (tabla más abajo) — cobrar la parte de cada quien.
5. Apenas llegue el router, familiarizarte con RouterOS y empezar a armar el script `.rsc` de OSPF + firewall de R1 (coordinando con Sergio para que su config de VyOS calce).
6. Apenas llegue el switch, configurar: 1 puerto trunk (VLANs 10, 20, 30, 31, 40, 60) hacia R1, y al menos 2 puertos de acceso en VLANs distintas (ej. VLAN 10 y VLAN 30) para los 2 hosts de prueba — es lo que le da fuerza real a la demo de segmentación.
7. El día que se prueben conectividad física, sos vos + Samuel los que corren el checklist de la sección 6 del documento de Fase 4 (ahora incluye validar que los 2 hosts NO se alcancen entre sí salvo reglas explícitas).

**Cuanto antes se compre el equipo del laboratorio, más tiempo de prueba real tiene el proyecto — no lo dejes para las últimas semanas antes de octubre.**
