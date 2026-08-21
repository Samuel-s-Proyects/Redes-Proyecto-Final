# 08 — Equipo Físico y Presupuesto (precios reales Guatemala)

Precios investigados en tiendas guatemaltecas en línea al momento de escribir este documento (agosto 2026). **Verificar disponibilidad/precio actual antes de comprar** — los precios de importados cambian con el tipo de cambio y stock.

## 1. Laboratorio de demostración (lo mínimo para cumplir Fase 4) — decisión final del equipo (revisada)

Se descarta GNS3 por completo: se compra equipo físico real para R1 y el switch (ver conversación de decisión — GNS3 solo hubiera ahorrado ~Q600 a cambio de meter puentes de red virtuales innecesarios).

**Corrección sobre el switch (revisión del equipo):** inicialmente se había elegido un switch 100% no administrable (TP-Link TL-SG105, sin VLAN) razonando que la Fase 4 no exige VLANs en esa pieza específica — literalmente cierto, el PDF solo pide "1 switch físico para conectar clientes". Pero el equipo detectó que esto desaprovecha la pieza más visual del proyecto: sin VLAN en el switch físico, no se puede *demostrar en vivo* la segmentación por departamento que es el hilo conductor de todo el diseño (Fase 1 la propone, Fase 3 la direcciona, y aquí es donde se vería funcionando de verdad). Se corrige a un switch **Easy Smart con VLAN 802.1Q** — sigue siendo barato, y ahora si soporta un puerto trunk hacia R1 + puertos de acceso por VLAN.

⚠️ **Ojo con el nombre del modelo**: `TL-SG105` (sin sufijo) y `TL-SG105E` son productos **distintos** — el primero es 100% no administrable (ni interfaz web tiene), el segundo sí es "Easy Smart" con VLAN. No son el mismo switch con un modo oculto — hay que comprar específicamente el que trae VLAN.

| Ítem | Modelo sugerido | Precio aprox. (Q) | Fuente | Notas |
|---|---|---|---|---|
| Router Core físico (R1) | MikroTik hEX (RB750Gr3) — 5 puertos Gigabit, RouterOS L4, soporta OSPF | **Q 585 – Q 625** | [Kemik.gt](https://www.kemik.gt/mikrotik-rb750gr3-router-de-4-puertos-gigabit-hex) | La opción más barata y confiable para Core con OSPF en Guatemala |
| Switch físico de clientes | TP-Link Easy Smart, 5 puertos Gigabit (4 con PoE), VLAN 802.1Q | **Q 318** | [Kemik.gt](https://www.kemik.gt/tp-link-easy-smart-switch-5-puertos-4-puertos-poe) | Confirmar el modelo exacto (línea Easy Smart TL-SG105PE/TL-SG105E) al comprar — el PoE es un extra útil para los teléfonos IP de la Fase 1, no imprescindible para la demo |
| Cable UTP Cat 6 (para R1↔Servidor y R1↔Switch↔Host) | Caja o retazos, patch cords prefabricados | **Q 100 – Q 150** | Ferretería/tienda de redes local | Con 3–4 patch cords Cat 6 de 2-3m alcanza para el laboratorio |
| **Subtotal equipo de red** | | **≈ Q 1,000 – Q 1,090** | | |

**Alternativa de respaldo** si el Easy Smart no se consigue fácil: MikroTik CSS610-8G-2S+IN (Q750–1,203, confirmado en Kemik/GlobalNetbox) — también VLAN-capaz vía SwOS, mismo ecosistema/vendor que R1, pero más caro. Se deja como plan B, no como elección primaria.

## 2. Servidor: laptops del equipo + SSD externo portátil (no se compra mini PC)

Decisión final: **no hace falta comprar un mini PC dedicado**. El equipo usa la laptop de Samuel (i5 6ta gen, 8GB RAM) para desarrollo durante el semestre, y una laptop prestada de 16GB RAM el día de la demo — ambas bootean el mismo ambiente desde un **SSD externo** (ver estrategia completa de portabilidad en [05-Terraform-Ansible-IaC.md](05-Terraform-Ansible-IaC.md) sección 8).

| Ítem | Modelo/especificación | Precio aprox. (Q) | Notas |
|---|---|---|---|
| SSD 256–500GB (SATA o NVMe) | Cualquier marca confiable (Kingston, Crucial, WD) | **Q 300 – Q 500** | Es el componente que más impacta el rendimiento con varias VMs corriendo a la vez — más importante que CPU/RAM extra |
| Case/enclosure USB 3.0 o USB-C para el SSD | Genérico, según tipo de SSD (SATA 2.5" o M.2 NVMe) | **Q 100 – Q 150** | Convierte el SSD en un disco de arranque portátil entre laptops |
| Adaptador USB-Ethernet (respaldo de conectividad) | Genérico chipset Realtek | **Q 80 – Q 150** | Para asegurar salida de red en la laptop prestada el día de la demo, sin depender del driver de su NIC integrada |
| USB/disco externo para backups (`vzdump`) | 32-64GB USB 3.0, cualquier marca | **Q 80 – Q 150** | Debe ser un disco **distinto** del SSD de arranque — respaldar en el mismo disco que falla no protege de nada |
| **Subtotal "servidor portátil"** | | **≈ Q 560 – Q 950** | Cero costo de mini PC — se reutiliza el hardware que el equipo ya tiene |

## 3. Resumen del presupuesto del laboratorio (lo que se necesita para cumplir el proyecto)

| Rubro | Precio aprox. (Q) |
|---|---|
| Equipo de red físico (R1 + switch VLAN + cableado) | Q 1,000 – Q 1,090 |
| SSD externo + enclosure + adaptador USB-Ethernet + disco de backup | Q 560 – Q 950 |
| Software | Q 0 (100% open source) |
| **Total laboratorio** | **≈ Q 1,560 – Q 2,040** |

Entre 5 personas ([ver equipo completo](11-Equipo-y-Responsabilidades.md)), esto es **≈ Q 312 – Q 408 por persona** en partes iguales (ya decidido) — sigue siendo mucho más manejable que los ~Q4,000–5,600 que hubiera costado comprando un mini PC nuevo además del equipo de red.

## 4. Presupuesto de referencia — rollout de producción real (184 usuarios, 4 pisos)

Esto **no** es necesario para aprobar el curso, pero el enunciado pide "listado de materiales, características, marcas, modelos y presupuesto" para el diseño físico completo de la Fase 1 — se documenta como el proyecto real que Virtual Solutions ejecutaría después de validar el diseño en el laboratorio. Las cantidades de cableado/conectores/patch panels **no son estimaciones al ojo** — salen de la memoria de cálculo en [04-Diseno-Fisico.md](../Fase-1-Diseno-Red-Corporativa/04-Diseno-Fisico.md) §8 (405 drops totales, 45m promedio por corrida, 10% de desperdicio).

| Rubro | Ítem de referencia | Cantidad | Precio unitario aprox. (Q) | Subtotal (Q) |
|---|---|---|---|---|
| Switches de piso PoE+ | MikroTik CRS326-24G-2S+ (o equivalente 48p en pisos grandes) | 3 (pisos 2, 3, 4) | ~Q 2,100 | Q 6,300 |
| Switch de distribución/core secundario | MikroTik CRS309 o similar con SFP+ | 1 | ~Q 3,300 | Q 3,300 |
| Router Core de producción (redundante) | 2x MikroTik CCR o RB de gama media | 2 | ~Q 1,500 | Q 3,000 |
| Cable UTP Cat 6 (caja 305m) | Genérico certificado — **66 cajas** (405 drops × 45m promedio + 10% desperdicio ÷ 305m) | 66 cajas | ~Q 650 | Q 42,900 |
| Fibra óptica OM4 backbone | Carrete 500m (cubre 3 corridas × 2 hilos + margen) | 1 carrete | ~Q 3,500 (cotizar puntual, sin precio GT confirmado) | Q 3,500 |
| Patch panels 24p Cat 6 | Repartidos: 3 MDF/Ventas, 3 IDF-P2, 5 IDF-P3, 5 IDF-P4 (405 drops ÷ 24p) | 16 | ~Q 350 | Q 5,600 |
| Keystone jacks Cat 6 | 1 por drop | 405 | ~Q 18 | Q 7,290 |
| Faceplates dobles | 1 por puesto + margen áreas comunes | 195 | ~Q 25 | Q 4,875 |
| Racks de pared 12U (IDF) | | 3 | ~Q 900 | Q 2,700 |
| Rack de piso 42U (Data Center) | | 1 | ~Q 4,500 | Q 4,500 |
| UPS de piso 1000VA | | 3 | ~Q 900 | Q 2,700 |
| UPS central Data Center 2N (on-line, dimensionado en 06-Data-Center-Tier4.md §3) | 2× ~3kVA (memoria de cálculo en 06-Data-Center-Tier4.md) | 2 | Q 15,000+ c/u | Ver 06-Data-Center-Tier4.md |
| Teléfonos IP | Grandstream GXP1610 o similar | 6 (piloto) | ~Q 350 | Q 2,100 |
| Mano de obra de certificación de cableado | Servicio (405 puntos a certificar) | 1 servicio | Variable, cotizar local | — |
| **Subtotal aproximado (sin UPS central ni generador)** | | | | **≈ Q 88,765** |

El salto respecto a una estimación superficial (que rondaría los Q30,000 si se usan cantidades "al ojo" sin memoria de cálculo) es real: cablear correctamente 405 puntos en un edificio de 4 pisos, certificados, es la parte que más se subestima en un presupuesto de red hecho sin metodología. El detalle de energía/enfriamiento/generador del Data Center Tier 4 (la parte más cara de un rollout real, fuera de este subtotal) está en [06-Data-Center-Tier4.md](../Fase-1-Diseno-Red-Corporativa/06-Data-Center-Tier4.md) §3-4, con su propia memoria de cálculo de carga IT.

## 5. Fuentes consultadas

- [Kemik.gt — MikroTik RB750GR3](https://www.kemik.gt/mikrotik-rb750gr3-router-de-4-puertos-gigabit-hex)
- [Kemik.gt — TP-Link Easy Smart Switch 5 puertos, 4 con PoE (switch VLAN elegido)](https://www.kemik.gt/tp-link-easy-smart-switch-5-puertos-4-puertos-poe)
- [Kemik.gt — TP-Link TL-SG105 (no administrable, descartado por no tener VLAN)](https://www.kemik.gt/tp-link-switch-tl-sg105-5-puertos-101001000mbps)
- [Kemik.gt — MikroTik CSS610-8G-2S+IN (plan B si no se consigue el Easy Smart)](https://www.kemik.gt/mikrotik-switch-css610-8g-2s-in)
- [GlobalNetbox — Switch CSS610-8G-2S+IN](https://globalnetbox.net/producto/switch-css610-8g-2sin-mikrotik/)
- [Kemik.gt — Switches de 24 puertos Gigabit](https://www.kemik.gt/switches-de-red)
