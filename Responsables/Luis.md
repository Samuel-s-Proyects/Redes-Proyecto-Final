# Módulo de Luis — Core Físico (Fase 4) + Presupuesto y Equipo

**Entrega 3 — 17 de octubre de 2026, pero la COMPRA de equipo no puede esperar hasta esa fecha.**

## Qué es tu módulo

Sos el responsable de la **parte física de la Fase 4**: el router Core R1, el switch físico, el cableado, y la configuración de enrutamiento (OSPF) del lado físico. También sos el dueño del presupuesto completo del proyecto. Documentos que son tuyos:

1. [Fase4-Nube-Privada-SDN.md](../Fase-4-Nube-Privada-SDN/04-Fase4-Nube-Privada-SDN.md) — secciones 3.6 y 3.7 (Core físico, switch, host de prueba) y sección 6 (checklist de pruebas de conectividad)
2. [Equipo-Fisico-Presupuesto.md](../00-Documentacion-General/08-Equipo-Fisico-Presupuesto.md) — el documento completo de presupuesto

## Decisiones clave de tu módulo y por qué se tomaron

### 1. Por qué comprar equipo físico real y no usar GNS3
El enunciado permite GNS3 para el Core "si no cuenta con equipo físico" — pero comprar es más barato de lo que parece (~Q600 el router) y evita meterse en el problema de "puentear" una red virtual (GNS3) con la red real de Proxmox, que es una fuente típica de horas perdidas debuggeando para alguien que recién empieza en esto. Con hardware real, conectás un cable y listo — la demo también se ve más convincente con tráfico real entre fierro real.

### 2. Por qué MikroTik hEX (RB750Gr3) para R1
Es la opción de mejor relación costo/funcionalidad en Guatemala (~Q585-625): soporta OSPF (obligatorio, no se aceptan rutas estáticas), VLANs 802.1Q, firewall stateful — todo lo que RouterOS necesita para cumplir el rol de Core, a una fracción del costo de un equipo Cisco equivalente.

### 3. Por qué un switch NO administrable (esto cambió a mitad de camino, importante que lo sepas)
Al principio se había propuesto un switch MikroTik administrable con VLAN (~Q750-1,200), pero al releer el enunciado con cuidado: la Fase 4 solo pide *"1 switch físico para conectar clientes"* — no exige VLANs en esa pieza específica (las VLANs de la Fase 1/3 son de diseño, no algo que haya que demostrar físicamente en ese switch). Por eso se cambió a un **TP-Link TL-SG105** no administrable (~Q214) — cumple la letra del requisito sin pagar de más por una función que no se pide ahí. Esta fue una decisión explícita de optimizar presupuesto sin sacrificar cumplimiento — bueno tenerla clara por si el catedrático pregunta por qué no es administrable.

### 4. Por qué no se compra un mini PC dedicado
En vez de eso, el "servidor" es un **SSD externo portátil** que se mueve entre la laptop de Samuel (desarrollo, 8GB RAM) y una laptop prestada de 16GB el día de la demo. Esto evita gastar Q1,800+ en un mini PC — el detalle completo de por qué y cómo está en [Terraform-Ansible-IaC.md](../00-Documentacion-General/05-Terraform-Ansible-IaC.md) sección 8, pero como dueño del presupuesto te conviene entender el ahorro: de un total inicial estimado de Q4,000-5,600 (con mini PC nuevo) se bajó a **Q1,380-1,790 total**.

### 5. Por qué OSPF y no otro protocolo
El enunciado deja el protocolo "a discreción" pero prohíbe rutas estáticas. OSPF se eligió porque lo soportan bien tanto RouterOS (tu lado, R1) como VyOS (lado de Sergio, VR1), es más simple de configurar y depurar que BGP para una topología de 2 routers, y es exactamente lo que se espera demostrar en un curso de Redes 1.

## Presupuesto — ya decidido, resumen para que lo tengas a mano

| Ítem | Precio | Quién lo compra/gestiona |
|---|---|---|
| Router MikroTik hEX RB750Gr3 | ~Q600 | Vos |
| Switch TP-Link TL-SG105 | ~Q214 | Vos |
| Cable UTP Cat 6 (patch cords) | ~Q100-150 | Vos |
| SSD externo + enclosure + adaptador USB-Ethernet | ~Q480-800 | Coordinado con Samuel (él lo usa a diario) |
| **Total** | **~Q1,380-1,790** | **Partes iguales entre los 5** (~Q280-360 c/u, ya decidido) |

## Diagramas de tu módulo

No tenés un diagrama propio pendiente de crear — tu parte física ya está representada dentro del diagrama de topología de Sergio (Fase 4) y en la elevación de rack de Melany (Fase 1). Tu tarea con los diagramas es de **validación, no de creación**: cuando Sergio y Melany pasen sus diagramas a draw.io, revisá que el lado físico (R1, switch, cableado) quede exactamente como vos lo vas a instalar de verdad — si compraste un modelo distinto o cambiaste algún puerto, avisales para que ajusten el diagrama.

## Qué te toca hacer ahora (esto sí es urgente, aunque la entrega sea en octubre)

1. Coordinar la compra del equipo (lista de arriba) — cobrar la parte de cada quien.
2. Apenas llegue el router, familiarizarte con RouterOS y empezar a armar el script `.rsc` de OSPF + firewall de R1 (coordinando con Sergio para que su config de VyOS calce).
3. El día que se prueben conectividad física, sos vos + Samuel los que corren el checklist de la sección 6 del documento de Fase 4.

**Cuanto antes se compre el equipo, más tiempo de prueba real tiene el proyecto — no lo dejes para las últimas semanas antes de octubre.**
