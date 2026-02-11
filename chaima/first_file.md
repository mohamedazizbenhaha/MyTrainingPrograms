Je vais vous aider à créer un schéma d'architecture détaillé pour votre plateforme IDS/HIDS. Commençons par décomposer tous les éléments.

## 📋 ÉLÉMENTS DE L'ARCHITECTURE

### **1. Couche de Surveillance (Assets Monitored)**
- **Serveurs Linux** (Ubuntu/Debian/CentOS) - avec agents Wazuh
- **Serveurs Windows** - avec agents Wazuh  
- **Postes de travail** - avec agents Wazuh
- **Équipements réseau** - routeurs, switches, firewalls

### **2. Couche de Détection**
- **Suricata (NIDS)** - Détection d'intrusion réseau
  - Analyse du trafic en temps réel
  - Application des règles de détection
  - Génération d'alertes réseau
  
- **Wazuh Agents (HIDS)** - Détection d'intrusion hôte
  - Surveillance des logs système
  - Détection d'anomalies
  - Vérification d'intégrité de fichiers (FIM)
  - Détection de rootkits

### **3. Couche de Collecte & Agrégation**
- **Filebeat** - Collecte et transfert des logs Suricata
- **Packetbeat** (optionnel) - Analyse des paquets réseau
- **Wazuh Manager** - Serveur central de gestion Wazuh

### **4. Couche de Traitement**
- **Logstash** (optionnel) - Enrichissement et transformation des logs
- **Wazuh Manager** - Corrélation et analyse des événements

### **5. Couche de Stockage**
- **Elasticsearch/OpenSearch** - Indexation et stockage des données

### **6. Couche de Visualisation**
- **Kibana/OpenSearch Dashboards** - Visualisation des données
- **Wazuh Dashboard** - Interface intégrée de gestion

### **7. Couche de Gestion**
- **SOC Analysts** - Équipe de surveillance
- **SIEM Console** - Console centralisée

---

## 🔄 INTERACTIONS DÉTAILLÉES

### **Flux 1: Détection Réseau (Suricata)**
```
Trafic Réseau → Port Mirror/TAP → Suricata
   ↓
Analyse avec règles (Emerging Threats, custom)
   ↓
Génération d'alertes (eve.json)
   ↓
Filebeat collecte les logs
   ↓
[Optionnel: Logstash pour enrichissement]
   ↓
Elasticsearch/OpenSearch (indexation)
   ↓
Kibana/Wazuh Dashboard (visualisation)
```

### **Flux 2: Détection Hôte (Wazuh)**
```
Hôtes (Linux/Windows) avec Wazuh Agent
   ↓
Collecte des logs système, événements, FIM, rootkit detection
   ↓
Envoi sécurisé au Wazuh Manager (port 1514/1515)
   ↓
Wazuh Manager - Décodage, analyse, corrélation
   ↓
Application des règles et policies
   ↓
Génération d'alertes
   ↓
Elasticsearch/OpenSearch via API
   ↓
Wazuh Dashboard (visualisation intégrée)
```

### **Flux 3: Corrélation et Alerte**
```
Alertes Suricata + Alertes Wazuh
   ↓
Elasticsearch (stockage centralisé)
   ↓
Corrélation temporelle et contextuelle
   ↓
Dashboard unifié (Kibana + Wazuh)
   ↓
Notifications (Email, Slack, SIEM externe)
   ↓
SOC Analysts - Investigation et réponse
```

---

## 📊 WORKFLOW DÉTAILLÉ

### **Phase 1: Collection (Data Collection)**
1. **Réseau**: Suricata capture le trafic via port mirroring
2. **Hôtes**: Agents Wazuh collectent logs, événements, métriques
3. **Timestamp**: Synchronisation temporelle (NTP crucial)

### **Phase 2: Détection (Detection & Analysis)**
1. **Suricata**: Matching des signatures avec le trafic
2. **Wazuh**: Application des règles sur les logs hôtes
3. **Anomaly Detection**: Détection de comportements anormaux

### **Phase 3: Génération d'Alertes (Alert Generation)**
1. Création d'alertes avec niveaux de sévérité
2. Enrichissement avec métadonnées (IP, user, process)
3. Classification par MITRE ATT&CK

### **Phase 4: Agrégation (Aggregation)**
1. Filebeat agrège les logs Suricata
2. Wazuh Manager agrège les logs des agents
3. Centralisation dans le pipeline

### **Phase 5: Corrélation (Correlation)**
1. Corrélation multi-sources dans Elasticsearch
2. Identification de chaînes d'attaque
3. Réduction des faux positifs

### **Phase 6: Stockage (Storage)**
1. Indexation dans Elasticsearch
2. Rétention selon politique (hot/warm/cold)
3. Backup et archivage

### **Phase 7: Visualisation (Visualization)**
1. Dashboards temps réel
2. Rapports de conformité
3. Investigations forensiques

### **Phase 8: Réponse (Response)**
1. Alerting vers équipes SOC
2. Playbooks de réponse automatisée
3. Blocage actif (Active Response Wazuh)

---

## 🐍 CODE PYTHON POUR GÉNÉRER LE SCHÉMA

Voici le code Python complet pour générer votre schéma d'architecture:

```python
# Installation des dépendances (à exécuter une fois dans votre cellule)
!pip install diagrams graphviz

# Code principal
from diagrams import Diagram, Cluster, Edge
from diagrams.onprem.security import Wazuh
from diagrams.onprem.monitoring import Grafana, Prometheus
from diagrams.elastic.elasticsearch import Elasticsearch, Kibana, Logstash
from diagrams.onprem.network import Internet, Nginx
from diagrams.onprem.compute import Server
from diagrams.generic.network import Router, Switch, Firewall
from diagrams.generic.os import Windows, LinuxGeneral
from diagrams.generic.storage import Storage
from diagrams.programming.language import Python
from diagrams.onprem.client import Users

# Configuration du diagramme
graph_attr = {
    "fontsize": "14",
    "bgcolor": "white",
    "pad": "0.5",
    "splines": "ortho",
    "nodesep": "0.8",
    "ranksep": "1.0"
}

node_attr = {
    "fontsize": "12",
    "style": "rounded",
}

edge_attr = {
    "fontsize": "10",
}

with Diagram(
    "Plateforme IDS/HIDS - Architecture Détaillée",
    filename="ids_hids_architecture",
    show=False,
    direction="TB",
    graph_attr=graph_attr,
    node_attr=node_attr,
    edge_attr=edge_attr,
    outformat="png"
):
    
    # Équipe SOC
    soc_team = Users("SOC Analysts")
    
    # ============================================
    # COUCHE DE VISUALISATION
    # ============================================
    with Cluster("Couche Visualisation & Gestion"):
        wazuh_dashboard = Grafana("Wazuh Dashboard")
        kibana = Kibana("Kibana/OpenSearch\nDashboards")
        dashboards = [wazuh_dashboard, kibana]
    
    # ============================================
    # COUCHE DE STOCKAGE & TRAITEMENT
    # ============================================
    with Cluster("Couche Stockage & SIEM"):
        with Cluster("Indexation & Stockage"):
            elasticsearch = Elasticsearch("Elasticsearch/\nOpenSearch")
        
        with Cluster("Corrélation & Management"):
            wazuh_manager = Wazuh("Wazuh Manager\n(Correlation Engine)")
            logstash = Logstash("Logstash\n(Optionnel)")
    
    # ============================================
    # COUCHE DE COLLECTE
    # ============================================
    with Cluster("Couche Collecte & Agrégation"):
        filebeat = Server("Filebeat\n(Log Shipper)")
        packetbeat = Server("Packetbeat\n(Network Analyzer)")
    
    # ============================================
    # COUCHE DE DÉTECTION
    # ============================================
    with Cluster("Couche Détection"):
        with Cluster("Network IDS"):
            suricata = Firewall("Suricata NIDS\n(Network Traffic)")
        
        with Cluster("Host IDS"):
            agent_linux = LinuxGeneral("Wazuh Agent\n(Linux)")
            agent_windows = Windows("Wazuh Agent\n(Windows)")
            agents = [agent_linux, agent_windows]
    
    # ============================================
    # COUCHE RÉSEAU & ASSETS
    # ============================================
    with Cluster("Infrastructure Monitorée"):
        with Cluster("Réseau"):
            network_traffic = Internet("Trafic Réseau\n(Internet/LAN)")
            tap_mirror = Router("Port Mirror/TAP")
        
        with Cluster("Assets (Hosts)"):
            linux_servers = Server("Serveurs Linux\nUbuntu/Debian/CentOS")
            windows_servers = Server("Serveurs Windows\nWindows Server")
            workstations = Server("Postes de Travail")
            monitored_hosts = [linux_servers, windows_servers, workstations]
    
    # ============================================
    # FLUX DE DONNÉES - DÉTECTION RÉSEAU
    # ============================================
    network_traffic >> Edge(label="Mirror Traffic", color="blue", style="bold") >> tap_mirror
    tap_mirror >> Edge(label="Span Port", color="blue") >> suricata
    suricata >> Edge(label="eve.json\n(Alerts & Logs)", color="red") >> filebeat
    filebeat >> Edge(label="JSON Logs", color="orange") >> logstash
    logstash >> Edge(label="Enriched Data", color="orange") >> elasticsearch
    
    # Alternative: Direct to ES
    filebeat >> Edge(label="Direct", color="orange", style="dashed") >> elasticsearch
    
    # ============================================
    # FLUX DE DONNÉES - DÉTECTION HÔTE
    # ============================================
    for host in monitored_hosts:
        host >> Edge(label="System Logs\nFIM, Rootkit", color="green") >> agents
    
    for agent in agents:
        agent >> Edge(label="Port 1514/1515\n(Encrypted)", color="green", style="bold") >> wazuh_manager
    
    wazuh_manager >> Edge(label="Analyzed Alerts\n(API)", color="purple") >> elasticsearch
    
    # ============================================
    # FLUX DE VISUALISATION
    # ============================================
    elasticsearch >> Edge(label="Query", color="black") >> dashboards
    wazuh_manager >> Edge(label="Direct Integration", color="purple", style="dashed") >> wazuh_dashboard
    
    # ============================================
    # FLUX SOC
    # ============================================
    for dashboard in dashboards:
        dashboard >> Edge(label="Monitor & Investigate", color="brown") >> soc_team
    
    # ============================================
    # FLUX DE CORRÉLATION
    # ============================================
    elasticsearch >> Edge(label="Correlation\nQueries", color="darkblue", style="dotted") >> wazuh_manager
    
    # Packetbeat optionnel
    network_traffic >> Edge(label="Packet Capture", color="blue", style="dashed") >> packetbeat
    packetbeat >> Edge(label="Network Metrics", color="orange", style="dashed") >> elasticsearch

print("✅ Diagramme généré avec succès : ids_hids_architecture.png")
print("\n📊 Le schéma montre:")
print("   • Flux de détection réseau (Suricata → Filebeat → ES)")
print("   • Flux de détection hôte (Agents → Wazuh Manager → ES)")
print("   • Couche de visualisation (Kibana + Wazuh Dashboard)")
print("   • Corrélation centralisée dans Elasticsearch")
print("   • Interactions avec l'équipe SOC")
```

---

## 🎨 VERSION ALTERNATIVE SIMPLIFIÉE

Si vous voulez une version plus simple avec matplotlib:

```python
!pip install matplotlib networkx

import matplotlib.pyplot as plt
import matplotlib.patches as mpatches
from matplotlib.patches import FancyBboxPatch, FancyArrowPatch
import numpy as np

fig, ax = plt.subplots(1, 1, figsize=(16, 12))
ax.set_xlim(0, 10)
ax.set_ylim(0, 12)
ax.axis('off')

# Définition des couleurs
colors = {
    'network': '#E3F2FD',
    'detection': '#FFF3E0',
    'collection': '#F3E5F5',
    'storage': '#E8F5E9',
    'visualization': '#FCE4EC',
    'arrow': '#546E7A'
}

# Fonction pour créer des boîtes
def create_box(ax, x, y, width, height, label, color, fontsize=10):
    box = FancyBboxPatch(
        (x, y), width, height,
        boxstyle="round,pad=0.1",
        edgecolor='black',
        facecolor=color,
        linewidth=2
    )
    ax.add_patch(box)
    ax.text(x + width/2, y + height/2, label,
            ha='center', va='center',
            fontsize=fontsize, weight='bold',
            wrap=True)

# Fonction pour créer des flèches
def create_arrow(ax, x1, y1, x2, y2, label='', color='black'):
    arrow = FancyArrowPatch(
        (x1, y1), (x2, y2),
        arrowstyle='->,head_width=0.4,head_length=0.8',
        color=color,
        linewidth=2,
        linestyle='-'
    )
    ax.add_patch(arrow)
    if label:
        mid_x, mid_y = (x1 + x2) / 2, (y1 + y2) / 2
        ax.text(mid_x + 0.2, mid_y, label,
                fontsize=8, style='italic',
                bbox=dict(boxstyle='round', facecolor='white', alpha=0.8))

# TITRE
ax.text(5, 11.5, 'Architecture IDS/HIDS - Plateforme de Détection d\'Intrusion',
        ha='center', fontsize=16, weight='bold')

# ========== COUCHE 1: RÉSEAU & ASSETS ==========
ax.text(0.5, 10.5, 'COUCHE 1: Infrastructure Monitorée', fontsize=12, weight='bold')
create_box(ax, 0.5, 9, 2, 1, 'Trafic\nRéseau', colors['network'])
create_box(ax, 3, 9, 1.5, 1, 'Serveurs\nLinux', colors['network'])
create_box(ax, 5, 9, 1.5, 1, 'Serveurs\nWindows', colors['network'])
create_box(ax, 7, 9, 1.5, 1, 'Postes de\nTravail', colors['network'])

# ========== COUCHE 2: DÉTECTION ==========
ax.text(0.5, 7.8, 'COUCHE 2: Détection', fontsize=12, weight='bold')
create_box(ax, 0.5, 6.5, 2.5, 1, 'Suricata\nNIDS', colors['detection'])
create_box(ax, 4, 6.5, 2, 1, 'Wazuh Agent\n(Linux)', colors['detection'])
create_box(ax, 6.5, 6.5, 2, 1, 'Wazuh Agent\n(Windows)', colors['detection'])

# Flèches Couche 1 → Couche 2
create_arrow(ax, 1.5, 9, 1.5, 7.5, 'Mirror', colors['arrow'])
create_arrow(ax, 3.75, 9, 5, 7.5, 'Logs', colors['arrow'])
create_arrow(ax, 5.75, 9, 5, 7.5, 'Logs', colors['arrow'])
create_arrow(ax, 7.75, 9, 7.5, 7.5, 'Logs', colors['arrow'])

# ========== COUCHE 3: COLLECTE ==========
ax.text(0.5, 5.3, 'COUCHE 3: Collecte & Agrégation', fontsize=12, weight='bold')
create_box(ax, 0.5, 4, 2, 1, 'Filebeat', colors['collection'])
create_box(ax, 4, 4, 4, 1, 'Wazuh Manager\n(Correlation)', colors['collection'])

# Flèches Couche 2 → Couche 3
create_arrow(ax, 1.75, 6.5, 1.5, 5, 'eve.json', colors['arrow'])
create_arrow(ax, 5, 6.5, 6, 5, 'Port 1514', colors['arrow'])
create_arrow(ax, 7.5, 6.5, 6, 5, 'Port 1514', colors['arrow'])

# ========== COUCHE 4: STOCKAGE ==========
ax.text(0.5, 2.8, 'COUCHE 4: Stockage & SIEM', fontsize=12, weight='bold')
create_box(ax, 2, 1.5, 3, 1, 'Elasticsearch/\nOpenSearch', colors['storage'])

# Flèches Couche 3 → Couche 4
create_arrow(ax, 1.5, 4, 3.5, 2.5, 'Logs', colors['arrow'])
create_arrow(ax, 6, 4, 3.5, 2.5, 'Alerts API', colors['arrow'])

# ========== COUCHE 5: VISUALISATION ==========
ax.text(0.5, 0.3, 'COUCHE 5: Visualisation', fontsize=12, weight='bold')
create_box(ax, 1, -1, 2, 0.8, 'Kibana\nDashboard', colors['visualization'], 9)
create_box(ax, 4, -1, 2, 0.8, 'Wazuh\nDashboard', colors['visualization'], 9)
create_box(ax, 7, -1, 2, 0.8, 'SOC\nAnalysts', colors['visualization'], 9)

# Flèches Couche 4 → Couche 5
create_arrow(ax, 3.5, 1.5, 2, -0.2, 'Query', colors['arrow'])
create_arrow(ax, 3.5, 1.5, 5, -0.2, 'Query', colors['arrow'])
create_arrow(ax, 3, -0.6, 8, -0.6, 'Monitor', colors['arrow'])
create_arrow(ax, 5, -0.6, 8, -0.6, 'Monitor', colors['arrow'])

# Légende
legend_elements = [
    mpatches.Patch(color=colors['network'], label='Infrastructure'),
    mpatches.Patch(color=colors['detection'], label='Détection'),
    mpatches.Patch(color=colors['collection'], label='Collecte'),
    mpatches.Patch(color=colors['storage'], label='Stockage'),
    mpatches.Patch(color=colors['visualization'], label='Visualisation')
]
ax.legend(handles=legend_elements, loc='upper right', fontsize=10)

plt.tight_layout()
plt.savefig('architecture_ids_hids_simple.png', dpi=300, bbox_inches='tight')
print("✅ Diagramme simplifié généré : architecture_ids_hids_simple.png")
plt.show()
```

---

## 📝 NOTES IMPORTANTES

**Ports et Protocoles:**
- Wazuh Agent → Manager: 1514 (events), 1515 (enrollment)
- Elasticsearch: 9200 (API), 9300 (cluster)
- Kibana: 5601
- Wazuh Dashboard: 443

**Sécurité:**
- Chiffrement TLS pour toutes les communications
- Authentification des agents Wazuh
- API keys pour Elasticsearch
- RBAC sur les dashboards

**Performance:**
- Suricata: Multithreading recommandé
- Elasticsearch: Cluster 3+ nodes en production
- Rétention: 30-90 jours selon volumétrie
