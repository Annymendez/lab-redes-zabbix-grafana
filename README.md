# lab-redes-zabbix-grafana
Proyecto Redes 2 - Zabbix, Grafana, n8n, IA
# 🌐 Sistema de Monitoreo, Visualización, Automatización e IA

## Redes de Comunicaciones 2 — Fundación Universitaria Konrad Lorenz

---

## 📋 Descripción

Proyecto académico que implementa un sistema integral de monitoreo de red en tiempo real sobre una topología emulada en **PNetLab**, integrando múltiples tecnologías profesionales:

- **Zabbix 5.0** — Monitoreo centralizado de dispositivos
- **Grafana 9.5** — Dashboards interactivos de visualización
- **n8n 0.236** — Automatización de alertas en tiempo real
- **Telegram Bot** — Canal de notificaciones instantáneas
- **Groq IA (Llama 3.1)** — Análisis inteligente de alertas con recomendaciones

---

## 🗺️ Topología de Red

```
Docker4/5 ── SW ── R1 ──EIGRP── R2 ──OSPF── R3 ── Docker7 ── Paipa
                         |         |                              |
                      Docker6   Zabbix-SV                      Medellín ── Bogotá ── Cúcuta
                                  |                              |                      |
                                 NAT                           R21-SW              Manizales
                                  |                              |                      |
                               Internet                      Docker8             Bucaramanga
                                                            (n8n/IA)                  |
                                                                              R16 ── Fortinet
                                                                               |
                                                                            Docker-PC
```

---

## 🔧 Tecnologías Implementadas

### Protocolos de Enrutamiento
| Protocolo | AS/Área | Routers | Función |
|-----------|---------|---------|---------|
| EIGRP | AS 50 | R1, R2, Paipa → Bucaramanga | Enrutamiento dinámico red principal y ciudades |
| OSPF | Área 0 | R2, R3 | Enrutamiento entre dominios |
| Redistribución | R2 | EIGRP ↔ OSPF | Interconexión de protocolos |
| NAT | R2 e1/0 | Overload | Salida a Internet |

### Seguridad (ACLs)
| Requisito | Descripción |
|-----------|-------------|
| 1 | Host .50 no accede al servicio web del servidor |
| 2 | Host .51 no puede hacer ping al servidor |
| 3 | El servidor no puede iniciar pings hacia afuera |
| 4 | Red 192.168.0.0/24 no accede a 172.16.200.0/24 |
| 5 | Solo host .50 puede hacer telnet a los routers |

---

## 📊 Componentes del Sistema de Monitoreo

### Zabbix Server
- **Versión:** 5.0.47
- **URL:** `http://192.168.183.128/zabbix`
- **Base de datos:** MySQL 5.7
- **Hosts monitoreados:** 7 (Docker4, Docker5, Docker6, Docker7, DockerMedellín, DockerBucaramanga, Zabbix Server)
- **Métricas:** CPU, memoria, disco, red, disponibilidad

### Grafana
- **Versión:** 9.5.20
- **URL:** `http://192.168.183.128:3000`
- **Plugin:** alexanderzobnin-zabbix-app v4.4.3
- **Dashboard:** Importado desde Grafana.com (ID: 5363)

### n8n
- **Versión:** 0.236.3 (Docker)
- **URL:** `http://192.168.183.128:5678`
- **Workflow:** Webhook → Groq IA → Telegram

### Telegram Bot
- **Bot:** @zabbix_lab_redes_bot
- **Función:** Recibe alertas analizadas por IA con recomendaciones

### Inteligencia Artificial
- **Proveedor:** Groq
- **Modelo:** Llama 3.1 8B Instant
- **Función:** Analiza alertas de Zabbix y genera diagnósticos con recomendaciones en español

---

## 🔄 Flujo de Alertas Automatizadas

```
Zabbix detecta problema (ej: host caído)
        ↓
Zabbix envía alerta al webhook de n8n
        ↓
n8n reenvía la alerta a Groq IA
        ↓
Groq analiza y genera recomendación:
"El Docker4 está caído. Recomiendo:
 1. Verificar la interfaz eth1
 2. Reiniciar el servicio de red
 3. Revisar logs del sistema"
        ↓
n8n envía la respuesta a Telegram
        ↓
El usuario recibe la alerta inteligente en su celular
```

---

## 🖥️ Dispositivos de la Topología

### Routers (IOL L3)
| Router | Función | Interfaces principales |
|--------|---------|----------------------|
| R1 | Gateway LAN + EIGRP | e0/0: 192.168.0.1, e0/2: 10.0.12.1 |
| R2 | Redistribución + NAT | e0/0: 172.16.200.1, e1/0: DHCP |
| R3 | OSPF + rutas estáticas | e0/0: 192.168.200.1 |
| Paipa | EIGRP ciudades | e0/0: 10.0.34.2 |
| Medellín | EIGRP | e0/0: 192.168.17.2 |
| Bogotá | EIGRP | e0/0: 192.168.18.2 |
| Cúcuta | EIGRP | e0/0: 192.168.19.2 |
| Manizales | EIGRP | e0/0: 192.168.20.2 |
| Bucaramanga | EIGRP | e0/0: 192.168.16.2 |
| R16 | Router LAN | e0/3: 192.168.1.2 |
| Fortinet | Firewall (IOL) | e0/0: 10.0.45.2 |

### Contenedores Docker
| Contenedor | IP Red | IP Gestión | Rol |
|-----------|--------|-----------|-----|
| Docker4 | 192.168.0.50 | 10.177.0.2 | Cliente LAN |
| Docker5 | 192.168.0.51 | 10.177.0.3 | Cliente LAN |
| Docker6 | 172.16.200.2 | 10.177.0.4 | Cliente R2 |
| Docker7 | 192.168.200.254 | 10.177.0.5 | Puente R3-Paipa |
| Docker8 | 192.168.22.10 | 10.177.0.7 | Cliente Medellín |
| PC | 192.168.100.10 | 10.177.0.6 | Cliente Bucaramanga |

---

## 🚀 Inicio Rápido

### Al encender PNetLab, ejecutar en MobaXterm:

```bash
# 1. Iniciar servicios de monitoreo
/root/iniciar_todo.sh

# 2. Reconfigurar Docker7 (puente)
docker exec docker47 bash -c "ip addr add 192.168.200.254/24 dev eth1 2>/dev/null; ip addr add 10.0.34.1/30 dev eth2 2>/dev/null; sysctl -w net.ipv4.ip_forward=1; ip route add default via 192.168.200.1 dev eth1 2>/dev/null; ip route add 192.168.16.0/24 via 10.0.34.2 dev eth2 2>/dev/null; ip route add 192.168.17.0/24 via 10.0.34.2 dev eth2 2>/dev/null; ip route add 192.168.18.0/24 via 10.0.34.2 dev eth2 2>/dev/null; ip route add 192.168.19.0/24 via 10.0.34.2 dev eth2 2>/dev/null; ip route add 192.168.20.0/24 via 10.0.34.2 dev eth2 2>/dev/null; ip route add 192.168.21.0/24 via 10.0.34.2 dev eth2 2>/dev/null; ip route add 192.168.22.0/24 via 10.0.34.2 dev eth2 2>/dev/null; ip route add 192.168.1.0/24 via 10.0.34.2 dev eth2 2>/dev/null; ip route add 192.168.100.0/24 via 10.0.34.2 dev eth2 2>/dev/null"

# 3. Reconfigurar Zabbix Agents
for c in docker44 docker45 docker46 docker47 docker59 docker60; do docker exec $c bash -c "sed -i '/^Server=/d; /^ServerActive=/d; /^ListenIP=/d' /etc/zabbix/zabbix_agentd.conf 2>/dev/null; echo 'Server=10.177.0.1' >> /etc/zabbix/zabbix_agentd.conf; echo 'ServerActive=10.177.0.1' >> /etc/zabbix/zabbix_agentd.conf; echo 'ListenIP=0.0.0.0' >> /etc/zabbix/zabbix_agentd.conf; service zabbix-agent restart 2>/dev/null"; done

# 4. Iniciar n8n
docker start n8n
```

### Accesos
| Servicio | URL | Credenciales |
|----------|-----|-------------|
| Zabbix | http://192.168.183.128/zabbix | Admin / zabbix |
| Grafana | http://192.168.183.128:3000 | admin / admin |
| n8n | http://192.168.183.128:5678 | admin / (configurado) |

---

## 📁 Estructura del Repositorio

```
├── README.md                                          # Este archivo
├── Informe_Tecnico_Completo_Zabbix_Grafana_n8n_IA.docx  # Informe técnico completo
└── Configuracion_Topologia_Completa.docx              # Configuración de la topología
```

---

## 👩‍💻 Autora

**Anamaria Mendez** — Ingeniería de Sistemas, Fundación Universitaria Konrad Lorenz

---

## 📌 Nota

Este proyecto fue desarrollado en un entorno académico controlado. El monitoreo se realizó exclusivamente sobre dispositivos autorizados dentro del laboratorio virtual de PNetLab, respetando los principios éticos y de seguridad de la información.
