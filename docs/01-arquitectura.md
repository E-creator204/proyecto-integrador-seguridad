# Arquitectura del entorno

Entorno de laboratorio segmentado con **pfSense** como firewall perimetral,
**Odoo** protegido por **SafeLine** (WAF), **Zabbix** para monitoreo y
**Wazuh** como SIEM.

## Topología

```mermaid
flowchart LR
    %% ============ ESTILOS ============
    classDef red fill:#ffe6e6,stroke:#c0392b,stroke-width:2px,color:#000
    classDef it   fill:#e6f0ff,stroke:#2c5aa0,stroke-width:2px,color:#000
    classDef dmz  fill:#e6ffe6,stroke:#2d7a2d,stroke-width:2px,color:#000
    classDef soc  fill:#f3e6ff,stroke:#6b2fa0,stroke-width:2px,color:#000
    classDef fw   fill:#fff4cc,stroke:#b8860b,stroke-width:3px,color:#000
    classDef link stroke:#888,stroke-width:1.5px

    %% ============ SEGMENTOS ============
    subgraph RRHH["RRHH — 10.10.20.0/24"]
        C1["Cliente RRHH<br/>DHCP 10.10.20.100-200"]
    end

    subgraph IT["IT — 10.10.30.0/24"]
        C2["Cliente IT<br/>DHCP 10.10.30.100-200"]
    end

    subgraph DMZ["DMZ — 10.10.10.0/24"]
        FW{{"pfSense<br/>10.10.10.1"}}
        SL["SafeLine WAF<br/>10.10.10.15:6778"]
        OD[("Odoo<br/>10.10.10.15:8069<br/>interno")]
        ZB["Zabbix<br/>10.10.10.15:6779"]
    end

    subgraph SOC["SOC — 10.10.40.0/24"]
        WZ["Wazuh SIEM<br/>10.10.40.10:1514"]
    end

    %% ============ FLUJOS ============
    C1 -->|"HTTP :6778"| SL
    C2 -->|"HTTPS :6779"| ZB
    SL  -->|"proxy :8069"| OD
    FW  -.->|"syslog :514"| WZ
    SL  -.->|"eventos"| WZ
    ZB  -.->|"eventos"| WZ
    C1  -.->|"temporal (colaboración)<br/>:6779"| ZB

    %% ============ APLICAR ESTILOS ============
    class C1 red
    class C2 it
    class FW fw
    class SL,OD,ZB dmz
    class WZ soc
    linkStyle default stroke:#888,stroke-width:1.5px
```

## Notas de diseño

- **Odoo (:8069)** solo es accesible desde SafeLine. Nunca se expone directo a RRHH ni a IT.
- **SafeLine (:6778)** es el único punto de entrada a la app web de RRHH.
- **Zabbix (:6779)** solo accesible desde IT. La línea punteada hacia RRHH es la **regla temporal de colaboración** que se abre en vivo durante la demo.
- **Wazuh (SOC)** recibe logs de tres fuentes: pfSense (syslog 514), SafeLine y Zabbix.
- Las flechas **punteadas** representan tráfico de logs o reglas temporales.

## Matriz de puertos

| Origen | Destino | Puerto | Acción | Propósito |
|---|---|---|---|---|
| RRHH | SafeLine | 6778 | Allow | Acceso a Odoo vía WAF |
| IT | Zabbix | 6779 | Allow | Monitoreo |
| SafeLine | Odoo | 8069 | Allow | Proxy interno |
| pfSense | Wazuh | 514/UDP | Allow | Logs del firewall |
| RRHH | Zabbix | 6779 | **Deny** (temporal Allow) | Caso de colaboración |
| * | Odoo:8069 | — | **Deny** | Nunca expuesto directo |
