---
config:
  theme: neo
---
flowchart TB
 subgraph FIREWALLZONE["🔥 Firewall & Gateway"]
        PFSENSE{"pfSense<br>Firewall"}
  end
 subgraph LANZONE["🏢 LAN - RRHH<br>10.10.20.0/24"]
        RHPC["💻 Lubuntu Client<br>DHCP"]
        RHLABEL["GW: 10.10.20.1<br>DHCP Enabled"]
  end
 subgraph OPT1ZONE["💻 OPT1 - IT<br>10.10.30.0/24"]
        ITPC["💻 Lubuntu Client<br>DHCP"]
        ITLABEL["GW: 10.10.30.1<br>DHCP Enabled"]
  end
 subgraph OPT2ZONE["📦 OPT2 - DMZ<br>10.10.10.0/24"]
        UBUNTUSERVER["🐧 Ubuntu Server<br>10.10.10.15"]
        SAFELINE["🛡️ SafeLine WAF<br>:6778"]
        ODOO["📊 Odoo ERP<br>:8069<br>Internal"]
        GRAFANA["📈 Grafana<br>:6779"]
        PROMETHEUS["📉 Prometheus<br>+ Node Exporter"]
        DMZLABEL["GW: 10.10.10.1<br>Static IP"]
  end
 subgraph OPT3ZONE["🔒 OPT3 - SOC<br>10.10.40.0/24"]
        WAZUH["🛡️ Wazuh SIEM<br>10.10.40.10"]
        SOCLABEL["GW: 10.10.40.1<br>Static IP<br>Agent Hub"]
  end
    WAN["🌐 Internet<br>WAN"] -- WAN Connection --> PFSENSE
    PFSENSE -- VLAN/Segment --> RHPC & ITPC & UBUNTUSERVER & WAZUH
    UBUNTUSERVER --> SAFELINE & GRAFANA
    SAFELINE -- Reverse Proxy<br>Port 6778 --> ODOO
    GRAFANA -- Data Pull --> PROMETHEUS
    RHPC -- ✅ Puerto 6778<br>SafeLine Only --> SAFELINE
    RHPC -- 🔄 Temporal<br>Puerto 6779 --> GRAFANA
    ITPC -- ✅ Puerto 6779<br>Grafana Only --> GRAFANA
    RHPC -- Logs/Events --> WAZUH
    ITPC -- Logs/Events --> WAZUH
    UBUNTUSERVER -- Logs/Events<br>Docker --> WAZUH
    PFSENSE -- Firewall Logs --> WAZUH
