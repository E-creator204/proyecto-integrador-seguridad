# 02 — Plan de redes

## Topología general

Entorno segmentado en 4 redes host-only sobre VMware Workstation Pro,
con pfSense como único punto de tránsito entre segmentos.

## Segmentos en VMware

| Segmento | Red VMware | Subred | DHCP | Adaptador de host |
|---|---|---|---|---|
| DMZ (Servicios) | VMnet4 | 10.10.10.0/24 | No | Desconectado |
| RRHH (LAN) | VMnet2 | 10.10.20.0/24 | Sí (100–200) | Desconectado |
| IT | VMnet3 | 10.10.30.0/24 | Sí (100–200) | Desconectado |
| SOC | VMnet5 | 10.10.40.0/24 | No | Desconectado |
| WAN (NAT) | VMnet8 | DHCP | — | — |

> Las 4 redes host-only se crean desde **Edit → Virtual Network Editor**,
> sin DHCP local de VMware y sin adaptador de host conectado, para que
> el host no se mezcle con los segmentos del laboratorio.

## Direccionamiento

| Equipo | IP | Máscara | Método | Rol |
|---|---|---|---|---|
| pfSense LAN (RRHH) | 10.10.20.1 | /24 | Estática | Gateway |
| pfSense OPT1 (IT) | 10.10.30.1 | /24 | Estática | Gateway |
| pfSense OPT2 (DMZ) | 10.10.10.1 | /24 | Estática | Gateway |
| pfSense OPT3 (SOC) | 10.10.40.1 | /24 | Estática | Gateway |
| Servidor DMZ (Odoo + SafeLine + Grafana) | 10.10.10.15 | /24 | Estática | Servicios |
| Servidor Wazuh | 10.10.40.10 | /24 | Estática | SIEM |
| Cliente RRHH | DHCP 10.10.20.100–200 | /24 | DHCP | Usuario |
| Cliente IT | DHCP 10.10.30.100–200 | /24 | DHCP | Admin |

## Matriz de puertos y reglas

| # | Origen | Destino | Puerto | Protocolo | Acción | Propósito |
|---|---|---|---|---|---|---|
| 1 | RRHH | SafeLine (10.10.10.15) | 6778 | TCP | Allow | Acceso a Odoo vía WAF |
| 2 | IT | Grafana (10.10.10.15) | 6779 | TCP | Allow | Monitoreo |
| 3 | IT | pfSense | 443 | TCP | Allow | Gestión firewall |
| 4 | IT | SafeLine | 6778 | TCP | Allow | Administración WAF |
| 5 | RRHH | Grafana | 6779 | TCP | **Deny** (regla temporal Allow) | Caso de colaboración |
| 6 | * | Odoo (10.10.10.15) | 8069 | TCP | **Deny** | Nunca expuesto directo |
| 7 | Cualquier segmento | Wazuh (10.10.40.10) | 1514/1515 | TCP | Allow | Agentes Wazuh |
| 8 | pfSense | Wazuh | 514 | UDP | Allow | Syslog del firewall |
| 9 | WAN | Cualquiera | * | * | **Deny** (default) | Hardening |


## Reglas de filtrado por defecto

Todas las interfaces terminan con una regla **deny all** implícita o explícita.
Solo el tráfico listado en la matriz de arriba se permite.
