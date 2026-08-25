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

Esto **no** es necesario para aprobar el curso, pero el enunciado pide "listado de materiales, características, marcas, modelos y presupuesto" para el diseño físico completo de la Fase 1 — se documenta como el proyecto real que Virtual Solutions ejecutaría después de validar el diseño en el laboratorio. Las cantidades de cableado/conectores/patch panels **no son estimaciones al ojo** — salen de la memoria de cálculo en [04-Diseno-Fisico.md](../Fase-1-Diseno-Red-Corporativa/04-Diseno-Fisico.md) §8 (366 drops totales, sobre 166 puestos de trabajo reales, 45m promedio por corrida, 10% de desperdicio).

Las cantidades de switches **no son "1 por piso" al ojo** — salen del dimensionamiento por puntos de red reales (2 drops por puesto, redondeado a SKU comercial de 48 puertos) en [04-Diseno-Fisico.md](../Fase-1-Diseno-Red-Corporativa/04-Diseno-Fisico.md) §1.1, y el Core y la Distribución se implementan en par redundante (§1.1.1) para cerrar el punto único de falla de red que quedaba documentado como pendiente. Los precios de switches y router son **precios reales verificados en vivo** en Pacifiko.com (agosto 2026), no estimaciones de catálogo internacional — ver comparación completa, incluyendo la cotización real Cisco Meraki aportada por el equipo, en [04-Diseno-Fisico.md](../Fase-1-Diseno-Red-Corporativa/04-Diseno-Fisico.md) §1.2.

| Rubro | Ítem de referencia | Cantidad | Precio unitario (Q) | Subtotal (Q) |
|---|---|---|---|---|
| Switches de acceso 48p PoE+ | MikroTik CRS354-48P-4S+2Q+RM — 2 Piso1, 2 Piso2, 3 Piso3, 3 Piso4 (ver doc04 §1.1) | 10 | Q 9,444 (real, Pacifiko.com) | Q 94,440 |
| Switch de distribución (redundante, 2N) | MikroTik CRS326-24S+2Q+RM (MDF/Piso 2, par activo-activo) | 2 | Q 5,714 (real, Pacifiko.com) | Q 11,428 |
| Router Core de producción (redundante, VRRP) | 2× MikroTik CCR2004-16G-2S+ | 2 | ~Q 4,630 (real, Pacifiko.com, rango Q4,484–4,779) | Q 9,260 |
| Cable UTP Cat 6 (caja 305m) | Genérico certificado — **60 cajas** (366 drops × 45m promedio + 10% desperdicio ÷ 305m) | 60 cajas | ~Q 650 | Q 39,000 |
| Fibra óptica OM4 backbone | Carrete 500m (cubre 3 corridas × 2 hilos + margen) | 1 carrete | ~Q 3,500 (cotizar puntual, sin precio GT confirmado) | Q 3,500 |
| Patch panels 24p Cat 6 | Repartidos: 3 IDF-P1, 3 MDF-P2, 5 IDF-P3, 5 IDF-P4 (366 drops ÷ 24p) | 16 | ~Q 350 | Q 5,600 |
| Keystone jacks Cat 6 | 1 por drop | 366 | ~Q 18 | Q 6,588 |
| Faceplates dobles | 1 por puesto (166) + margen áreas comunes | 183 | ~Q 25 | Q 4,575 |
| Racks de pared 12U (IDF) | | 3 | ~Q 900 | Q 2,700 |
| Rack de piso 42U (Data Center) | | 1 | ~Q 4,500 | Q 4,500 |
| UPS de piso 1000VA | | 3 | ~Q 900 | Q 2,700 |
| UPS central Data Center 2N (on-line, dimensionado en 06-Data-Center-Tier4.md §3) | 2× ~6kVA, APC Smart-UPS Rack 6U 208V | 2 | Q 40,113 c/u (real, Kemik.gt) | Ver 06-Data-Center-Tier4.md |
| Teléfonos IP (repartidos en los 4 pisos, ver doc04 §1.3) | Grandstream GXP1610 o similar | 6 (piloto) | ~Q 350 | Q 2,100 |
| Mano de obra de certificación de cableado | Servicio (366 puntos a certificar) | 1 servicio | Variable, cotizar local | — |
| **Subtotal (sin UPS central ni generador del Data Center)** | | | | **≈ Q 186,391** |

**Cómo se compara este número con la realidad del mercado — la pregunta que más importa validar**: este subtotal subió de una versión anterior (~Q146,565) a **≈Q186,391** por tres correcciones, todas con causa identificada, no ajustes cosméticos:

1. **Precios reales, no de catálogo sin ajustar** (+~Q30,000 en switches de acceso): el precio anterior de los MikroTik CRS354 venía de una review internacional en USD sin el margen de importación real a Guatemala. El precio verificado en vivo en Pacifiko.com es Q9,444/u, no Q6,400/u.
2. **Cierre de un punto único de falla en Distribución** (+~Q8,000): se pasó de 1 a 2 switches de distribución (ver doc04 §1.1.1) — un Data Center Tier IV no puede depender de un solo switch de distribución sin importar cuán redundante sea su energía.
3. **Router Core de gama apropiada para producción** (+~Q6,000): el hEX RB750Gr3 es correcto para el laboratorio de Fase 4 (SOHO, ~Q600), pero un rollout de producción real de 184 usuarios usa un equipo de gama Cloud Core (CCR2004), no el mismo modelo económico del laboratorio.
   - Parcialmente compensado por la corrección de la memoria de cálculo de cableado (-~Q5,000: 366 drops reales en vez de 405, ver doc04 §8).

**Frente a la cotización real de Cisco Meraki de referencia (Q714,000–760,000, solo switches+router, aportada por el equipo — ver comparación completa en doc04 §1.2)**: la comparación correcta, alcance por alcance (switches de acceso + distribución redundante + router Core redundante, sin mezclar con el cableado), es **Q115,128 (MikroTik) vs. Q714,000–760,000 (Meraki)** — una brecha real de **~6.2× a 6.6×**, no una fracción menor. La diferencia no es un error de cálculo de ningún lado, ni Meraki "incluye más" para justificar el salto — son dos decisiones de marca y de modelo de licenciamiento distintas (desglose completo de las 3 razones verificadas en doc04 §1.2: modelo de licenciamiento en la nube de Meraki, RouterOS como software de nivel profesional real, y consistencia de ecosistema). Ambas cifras están documentadas con fuente real y verificable — se decidió MikroTik después de comparar ambas, no por defecto.

El detalle de energía/enfriamiento/generador del Data Center Tier 4 (la parte más cara de un rollout real, fuera de este subtotal) está en [06-Data-Center-Tier4.md](../Fase-1-Diseno-Red-Corporativa/06-Data-Center-Tier4.md) §3-4, con su propia memoria de cálculo de carga IT y equipo real (APC Smart-UPS, Kemik.gt).

## 5. Fuentes consultadas

- [Kemik.gt — MikroTik RB750GR3](https://www.kemik.gt/mikrotik-rb750gr3-router-de-4-puertos-gigabit-hex)
- [Kemik.gt — TP-Link Easy Smart Switch 5 puertos, 4 con PoE (switch VLAN elegido)](https://www.kemik.gt/tp-link-easy-smart-switch-5-puertos-4-puertos-poe)
- [Kemik.gt — TP-Link TL-SG105 (no administrable, descartado por no tener VLAN)](https://www.kemik.gt/tp-link-switch-tl-sg105-5-puertos-101001000mbps)
- [Kemik.gt — MikroTik CSS610-8G-2S+IN (plan B si no se consigue el Easy Smart)](https://www.kemik.gt/mikrotik-switch-css610-8g-2s-in)
- [GlobalNetbox — Switch CSS610-8G-2S+IN](https://globalnetbox.net/producto/switch-css610-8g-2sin-mikrotik/)
- [Pacifiko.com — MikroTik CRS354-48P-4S+2Q+RM, Q9,444 (switch de acceso, producción, verificado en vivo)](https://www.pacifiko.com/compras-en-linea/mikrotik-crs354-48p-4s-2q-rm-switch-has-48-x-1g-rj45-ports-and-4-x-10g-sfp-ports-2-x-40g-qsfp-ports-for-extremely-fast-fiber-connections-or-linking-with-other-40-gbps-devices)
- [Pacifiko.com — catálogo MikroTik Guatemala (CRS326-24S+2Q+RM Q5,714, CCR2004-16G-2S+ Q4,484-4,779)](https://www.pacifiko.com/mikrotik)
- [Guatemala Digital — Cisco Business CBS350-48P-4G (descontinuado/sin stock desde 09/2023, verificado en vivo)](https://guatemaladigital.com/Cisco-Business-CBS350-48P-Managed-Switch-48-Port-GE-PoE-4x1G-SFP-Limited-Lifetime-Protection-(CBS350-48P-4G)/Producto/16248335)
- [Kemik.gt — Switches de 24 puertos Gigabit](https://www.kemik.gt/switches-de-red)
- [Kemik.gt — APC Smart-UPS 6kVA Rack 6U 208V, Q40,113 (UPS central Data Center)](https://www.kemik.gt/comprar/apc-smart-ups-6kva-en-rack-6u-208v)
- Cotización Cisco Meraki (MS425-32/MS350-24X/MS120-24P) — GBM Guatemala e IT Solutions Guatemala, aportada por el equipo como referencia real de mercado enterprise.
