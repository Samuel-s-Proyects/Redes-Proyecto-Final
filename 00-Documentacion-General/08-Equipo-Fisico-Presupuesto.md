# 08 — Equipo Físico y Presupuesto (precios reales Guatemala)

Precios investigados en tiendas guatemaltecas en línea al momento de escribir este documento (agosto 2026). **Verificar disponibilidad/precio actual antes de comprar** — los precios de importados cambian con el tipo de cambio y stock.

## 1. Laboratorio de demostración (lo mínimo para cumplir Fase 4) — decisión final del equipo

Se descarta GNS3 por completo: se compra equipo físico real para R1 y el switch (ver conversación de decisión — GNS3 solo hubiera ahorrado ~Q600 a cambio de meter puentes de red virtuales innecesarios). También se descarta el switch administrable caro: la Fase 4 no exige VLANs en ese switch específico, solo "conectar clientes", así que un switch básico no administrable cumple la regla igual y cuesta una fracción.

| Ítem | Modelo sugerido | Precio aprox. (Q) | Fuente | Notas |
|---|---|---|---|---|
| Router Core físico (R1) | MikroTik hEX (RB750Gr3) — 5 puertos Gigabit, RouterOS L4, soporta OSPF | **Q 585 – Q 625** | [Kemik.gt](https://www.kemik.gt/mikrotik-rb750gr3-router-de-4-puertos-gigabit-hex) | La opción más barata y confiable para Core con OSPF en Guatemala |
| Switch físico de clientes | TP-Link TL-SG105 — 5 puertos Gigabit, no administrable | **Q 214** (o Tenda SG105 a Q 157) | [Kemik.gt](https://www.kemik.gt/tp-link-switch-tl-sg105-5-puertos-101001000mbps) | Cumple el requisito literal de la Fase 4 sin pagar de más por VLANs que no se piden en esa pieza |
| Cable UTP Cat 6 (para R1↔Servidor y R1↔Switch↔Host) | Caja o retazos, patch cords prefabricados | **Q 100 – Q 150** | Ferretería/tienda de redes local | Con 3–4 patch cords Cat 6 de 2-3m alcanza para el laboratorio |
| **Subtotal equipo de red** | | **≈ Q 900 – Q 990** | | |

## 2. Servidor: laptops del equipo + SSD externo portátil (no se compra mini PC)

Decisión final: **no hace falta comprar un mini PC dedicado**. El equipo usa la laptop de Samuel (i5 6ta gen, 8GB RAM) para desarrollo durante el semestre, y una laptop prestada de 16GB RAM el día de la demo — ambas bootean el mismo ambiente desde un **SSD externo** (ver estrategia completa de portabilidad en [05-Terraform-Ansible-IaC.md](05-Terraform-Ansible-IaC.md) sección 8).

| Ítem | Modelo/especificación | Precio aprox. (Q) | Notas |
|---|---|---|---|
| SSD 256–500GB (SATA o NVMe) | Cualquier marca confiable (Kingston, Crucial, WD) | **Q 300 – Q 500** | Es el componente que más impacta el rendimiento con varias VMs corriendo a la vez — más importante que CPU/RAM extra |
| Case/enclosure USB 3.0 o USB-C para el SSD | Genérico, según tipo de SSD (SATA 2.5" o M.2 NVMe) | **Q 100 – Q 150** | Convierte el SSD en un disco de arranque portátil entre laptops |
| Adaptador USB-Ethernet (respaldo de conectividad) | Genérico chipset Realtek | **Q 80 – Q 150** | Para asegurar salida de red en la laptop prestada el día de la demo, sin depender del driver de su NIC integrada |
| **Subtotal "servidor portátil"** | | **≈ Q 480 – Q 800** | Cero costo de mini PC — se reutiliza el hardware que el equipo ya tiene |

## 3. Resumen del presupuesto del laboratorio (lo que se necesita para cumplir el proyecto)

| Rubro | Precio aprox. (Q) |
|---|---|
| Equipo de red físico (R1 + switch + cableado) | Q 900 – Q 990 |
| SSD externo + enclosure + adaptador USB-Ethernet | Q 480 – Q 800 |
| Software | Q 0 (100% open source) |
| **Total laboratorio** | **≈ Q 1,380 – Q 1,790** |

Entre 5 personas ([ver equipo completo](11-Equipo-y-Responsabilidades.md)), esto es **≈ Q 276 – Q 358 por persona** si se reparte en partes iguales — mucho más manejable que los ~Q4,000–5,600 que hubiera costado comprando un mini PC nuevo además del equipo de red. Queda pendiente que el equipo decida si se reparte en partes iguales o si Samuel cubre el hardware (por tenerlo físicamente) a cambio de que los demás asuman más carga de documentación/configuración — ver checklist en el documento 11.

## 4. Presupuesto de referencia — rollout de producción real (184 usuarios, 4 pisos)

Esto **no** es necesario para aprobar el curso, pero el enunciado pide "listado de materiales, características, marcas, modelos y presupuesto" para el diseño físico completo de la Fase 1 — se documenta como el proyecto real que Virtual Solutions ejecutaría después de validar el diseño en el laboratorio.

| Rubro | Ítem de referencia | Cantidad | Precio unitario aprox. (Q) | Subtotal (Q) |
|---|---|---|---|---|
| Switches de piso PoE+ | MikroTik CRS326-24G-2S+ (o equivalente 48p en pisos grandes) | 3 (pisos 2, 3, 4) | ~Q 2,100 | Q 6,300 |
| Switch de distribución/core secundario | MikroTik CRS309 o similar con SFP+ | 1 | ~Q 3,300 | Q 3,300 |
| Router Core de producción (redundante) | 2x MikroTik CCR o RB de gama media | 2 | ~Q 1,500 | Q 3,000 |
| Cable UTP Cat 6 (caja 305m) | Genérico certificado | 6 cajas | ~Q 650 | Q 3,900 |
| Fibra óptica OM4 backbone | Corridas MDF→IDF | 3 | ~Q 800 | Q 2,400 |
| Patch panels 24p Cat 6 | | 4 | ~Q 350 | Q 1,400 |
| Racks de pared 12U (IDF) | | 3 | ~Q 900 | Q 2,700 |
| Rack de piso 42U (Data Center) | | 1 | ~Q 4,500 | Q 4,500 |
| UPS de piso 1000VA | | 3 | ~Q 900 | Q 2,700 |
| UPS central Data Center (on-line, dimensionado en doc. 10) | | 1 (o 2N) | Q 15,000+ | Ver doc. 10 |
| Teléfonos IP | Grandstream GXP1610 o similar | 6 (piloto) | ~Q 350 | Q 2,100 |
| Mano de obra de certificación de cableado | Servicio | 1 | Variable, cotizar local | — |
| **Subtotal aproximado (sin UPS central ni generador)** | | | | **≈ Q 32,300** |

El detalle de energía/enfriamiento/generador del Data Center Tier 4 (la parte más cara de un rollout real) está en [10-Data-Center-Tier4.md](../Fase-1-Diseno-Red-Corporativa/10-Data-Center-Tier4.md), separado porque depende de cálculos de carga IT específicos.

## 5. Fuentes consultadas

- [Kemik.gt — MikroTik RB750GR3](https://www.kemik.gt/mikrotik-rb750gr3-router-de-4-puertos-gigabit-hex)
- [Kemik.gt — TP-Link TL-SG105 (switch no administrable elegido)](https://www.kemik.gt/tp-link-switch-tl-sg105-5-puertos-101001000mbps)
- [Kemik.gt — MikroTik CSS610-8G-2S+IN (referencia, no elegido para el laboratorio)](https://www.kemik.gt/mikrotik-switch-css610-8g-2s-in)
- [GlobalNetbox — Switch CSS610-8G-2S+IN](https://globalnetbox.net/producto/switch-css610-8g-2sin-mikrotik/)
- [Kemik.gt — Switches de 24 puertos Gigabit](https://www.kemik.gt/switches-de-red)
