Excellente question ! **OUI, vous pouvez tout déployer avec Docker** sur Linux. C'est même plus simple et reproductible ! Voici le guide complet step-by-step.

## 🐳 ARCHITECTURE DOCKER COMPLÈTE

Je vais vous donner tout ce dont vous avez besoin :

```python
!pip install matplotlib

import matplotlib.pyplot as plt
import matplotlib.patches as mpatches
from matplotlib.patches import FancyBboxPatch, FancyArrowPatch, Rectangle

fig, ax = plt.subplots(1, 1, figsize=(18, 12))
ax.set_xlim(0, 18)
ax.set_ylim(0, 12)
ax.axis('off')

colors = {
    'docker': '#2496ED',
    'network': '#00B4AB',
    'monitoring': '#FF6B6B',
    'agents': '#4ECDC4',
}

def create_container(ax, x, y, width, height, name, details, color):
    box = FancyBboxPatch(
        (x, y), width, height,
        boxstyle="round,pad=0.08",
        edgecolor='#2C3E50',
        facecolor=color,
        linewidth=2.5,
        alpha=0.85
    )
    ax.add_patch(box)
    
    # Header
    header = Rectangle((x, y + height - 0.5), width, 0.5, 
                       facecolor='#2C3E50', alpha=0.9)
    ax.add_patch(header)
    ax.text(x + width/2, y + height - 0.25, f'🐳 {name}',
            ha='center', va='center', fontsize=10, weight='bold', color='white')
    
    # Details
    detail_y = y + height - 0.9
    for detail in details:
        ax.text(x + 0.1, detail_y, detail, fontsize=7.5, va='top')
        detail_y -= 0.25

def create_arrow(ax, x1, y1, x2, y2, label='', color='black', width=2):
    arrow = FancyArrowPatch(
        (x1, y1), (x2, y2),
        arrowstyle='->,head_width=0.35,head_length=0.7',
        color=color, linewidth=width, mutation_scale=20
    )
    ax.add_patch(arrow)
    if label:
        mid_x, mid_y = (x1 + x2) / 2, (y1 + y2) / 2
        ax.text(mid_x, mid_y + 0.15, label, fontsize=7.5, weight='bold',
                bbox=dict(boxstyle='round,pad=0.3', facecolor='white', 
                         edgecolor=color, linewidth=1.5))

# Title
ax.text(9, 11.2, 'ARCHITECTURE DOCKER - PLATEFORME IDS/HIDS',
        ha='center', fontsize=15, weight='bold',
        bbox=dict(boxstyle='round,pad=0.5', facecolor='#2C3E50', 
                 edgecolor='black', linewidth=3),
        color='white')

# Docker Host
create_network_zone = lambda x, y, w, h, t, c: [
    ax.add_patch(Rectangle((x, y), w, h, facecolor=c, alpha=0.12, 
                          edgecolor='#34495e', linewidth=2.5, linestyle='--')),
    ax.text(x + 0.3, y + h - 0.35, t, fontsize=11, weight='bold',
            bbox=dict(boxstyle='round', facecolor='white', alpha=0.9))
]

create_network_zone(0.5, 0.5, 17, 9.5, '🖥️ DOCKER HOST (Ubuntu 22.04)', '#3498db')

# Docker Network
create_network_zone(1, 1, 7.5, 8, '🔗 Docker Network: ids-network', '#16a085')

# Containers Stack 1: ELK/Wazuh
create_container(ax, 1.5, 6.5, 3, 2.2,
                'elasticsearch',
                ['Image: elasticsearch:8.11.0',
                 'Port: 9200:9200',
                 'Volume: es-data',
                 'Mem: 4GB',
                 'Network: ids-network'],
                colors['docker'])

create_container(ax, 5, 6.5, 3, 2.2,
                'wazuh-manager',
                ['Image: wazuh/wazuh:4.7.0',
                 'Ports: 1514/1515/55000',
                 'Volume: wazuh-data',
                 'Mem: 2GB',
                 'Network: ids-network'],
                colors['docker'])

create_container(ax, 1.5, 3.8, 3, 2.2,
                'kibana',
                ['Image: kibana:8.11.0',
                 'Port: 5601:5601',
                 'Depends: elasticsearch',
                 'Mem: 1GB',
                 'Network: ids-network'],
                colors['docker'])

create_container(ax, 5, 3.8, 3, 2.2,
                'suricata',
                ['Image: jasonish/suricata:latest',
                 'Network: host mode',
                 'Volume: /var/log/suricata',
                 'Interface: eth0',
                 'Network: ids-network'],
                colors['monitoring'])

create_container(ax, 1.5, 1.5, 3, 1.8,
                'filebeat',
                ['Image: elastic/filebeat:8.11.0',
                 'Volume: suricata logs',
                 'Output: elasticsearch',
                 'Network: ids-network'],
                colors['monitoring'])

create_container(ax, 5, 1.5, 3, 1.8,
                'wazuh-dashboard',
                ['Image: wazuh/wazuh-dashboard',
                 'Port: 443:5601',
                 'Depends: wazuh-manager',
                 'Network: ids-network'],
                colors['docker'])

# Machines with Agents
create_network_zone(9.5, 1, 7.5, 8, '🖧 Machines à Monitorer', '#e74c3c')

create_container(ax, 10, 6.5, 3, 2.2,
                'linux-server-1',
                ['Ubuntu 22.04',
                 'Wazuh Agent (container)',
                 'Logs → Manager',
                 'Port: 1514'],
                colors['agents'])

create_container(ax, 13.5, 6.5, 3, 2.2,
                'linux-server-2',
                ['Debian 12',
                 'Wazuh Agent (container)',
                 'Logs → Manager',
                 'Port: 1514'],
                colors['agents'])

create_container(ax, 10, 3.8, 3, 2.2,
                'web-server',
                ['Nginx/Apache',
                 'Wazuh Agent',
                 'Access logs monitored',
                 'Port: 1514'],
                colors['agents'])

create_container(ax, 13.5, 3.8, 3, 2.2,
                'app-server',
                ['Python/Node.js App',
                 'Wazuh Agent',
                 'App logs monitored',
                 'Port: 1514'],
                colors['agents'])

# Flows
# Suricata → Filebeat
create_arrow(ax, 6.5, 3.8, 3, 2.4, 'eve.json', '#FF6B6B', 2.5)

# Filebeat → Elasticsearch
create_arrow(ax, 3, 2.4, 3, 6.5, 'Logs', '#2496ED', 2.5)

# Wazuh Manager → Elasticsearch
create_arrow(ax, 6.5, 7.5, 4.5, 7.5, 'Alerts API', '#9b59b6', 2.5)

# Elasticsearch → Kibana
create_arrow(ax, 3, 6.5, 3, 6, 'Query', '#2496ED', 2)

# Agents → Wazuh Manager
create_arrow(ax, 10, 7.5, 8, 7.5, 'Port 1514', '#00b894', 2.5)
create_arrow(ax, 13.5, 7.5, 8, 7.5, '', '#00b894', 2)
create_arrow(ax, 10, 4.8, 8, 7.2, '', '#00b894', 2)
create_arrow(ax, 13.5, 4.8, 8, 7.2, '', '#00b894', 2)

# Wazuh Manager → Dashboard
create_arrow(ax, 6.5, 4.8, 6.5, 3.3, 'Direct', '#9b59b6', 2)

# Info box
info_text = """
📦 DÉPLOIEMENT DOCKER:
• 1 docker-compose.yml pour tout
• Volumes persistants
• Network bridge ids-network
• Host network pour Suricata
• Agents en containers légers
"""

ax.text(1, 10.2, info_text, fontsize=8, family='monospace',
        bbox=dict(boxstyle='round,pad=0.4', facecolor='lightyellow', 
                 edgecolor='orange', linewidth=2, alpha=0.9),
        verticalalignment='top')

commands_text = """
🚀 COMMANDES:
docker-compose up -d
docker-compose ps
docker-compose logs -f
docker-compose down
"""

ax.text(16.5, 10.2, commands_text, fontsize=8, family='monospace',
        bbox=dict(boxstyle='round,pad=0.4', facecolor='lightblue', 
                 edgecolor='blue', linewidth=2, alpha=0.9),
        verticalalignment='top')

plt.tight_layout()
plt.savefig('architecture_docker.png', dpi=300, bbox_inches='tight', facecolor='white')
print("✅ Diagramme Docker généré: architecture_docker.png")
plt.show()
```

---

## 📋 GUIDE D'INSTALLATION STEP-BY-STEP

### **ÉTAPE 1: Prérequis sur votre machine Linux**

```bash
# Mettre à jour le système
sudo apt update && sudo apt upgrade -y

# Installer Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Ajouter votre user au groupe docker
sudo usermod -aG docker $USER
newgrp docker

# Installer Docker Compose
sudo apt install docker-compose-plugin -y

# Vérifier l'installation
docker --version
docker compose version

# Augmenter les limites système pour Elasticsearch
sudo sysctl -w vm.max_map_count=262144
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

---

### **ÉTAPE 2: Créer la structure du projet**

```bash
# Créer le dossier du projet
mkdir -p ~/ids-platform
cd ~/ids-platform

# Créer la structure
mkdir -p {suricata/rules,suricata/logs,wazuh/data,elasticsearch/data,filebeat/config,configs}
```

---

### **ÉTAPE 3: Créer le fichier docker-compose.yml principal**

Créez ce fichier dans `~/ids-platform/docker-compose.yml` :

```yaml
version: '3.8'

networks:
  ids-network:
    driver: bridge

volumes:
  elasticsearch-data:
  wazuh-data:
  suricata-logs:

services:
  # ===========================================
  # ELASTICSEARCH - Stockage & Indexation
  # ===========================================
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms2g -Xmx2g"
      - xpack.security.enabled=false
      - xpack.security.http.ssl.enabled=false
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data
    networks:
      - ids-network
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:9200 || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5

  # ===========================================
  # KIBANA - Dashboard Elasticsearch
  # ===========================================
  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    container_name: kibana
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
      - xpack.security.enabled=false
    ports:
      - "5601:5601"
    networks:
      - ids-network
    depends_on:
      elasticsearch:
        condition: service_healthy

  # ===========================================
  # WAZUH MANAGER - HIDS Manager
  # ===========================================
  wazuh-manager:
    image: wazuh/wazuh-manager:4.7.0
    container_name: wazuh-manager
    hostname: wazuh-manager
    environment:
      - INDEXER_URL=https://elasticsearch:9200
      - INDEXER_USERNAME=admin
      - INDEXER_PASSWORD=admin
      - FILEBEAT_SSL_VERIFICATION_MODE=none
      - SSL_CERTIFICATE_AUTHORITIES=""
      - SSL_CERTIFICATE=""
      - SSL_KEY=""
    ports:
      - "1514:1514/tcp"  # Agent connection (TCP)
      - "1515:1515/tcp"  # Agent enrollment
      - "55000:55000/tcp" # Wazuh API
    volumes:
      - wazuh-data:/var/ossec/data
      - ./wazuh/logs:/var/ossec/logs
    networks:
      - ids-network

  # ===========================================
  # WAZUH DASHBOARD
  # ===========================================
  wazuh-dashboard:
    image: wazuh/wazuh-dashboard:4.7.0
    container_name: wazuh-dashboard
    environment:
      - OPENSEARCH_HOSTS=http://elasticsearch:9200
      - WAZUH_API_URL=https://wazuh-manager:55000
    ports:
      - "8443:5601"
    networks:
      - ids-network
    depends_on:
      - wazuh-manager
      - elasticsearch

  # ===========================================
  # SURICATA - Network IDS
  # ===========================================
  suricata:
    image: jasonish/suricata:latest
    container_name: suricata
    network_mode: "host"  # Important pour capturer le trafic
    cap_add:
      - NET_ADMIN
      - NET_RAW
      - SYS_NICE
    volumes:
      - suricata-logs:/var/log/suricata
      - ./suricata/rules:/etc/suricata/rules
      - ./suricata/suricata.yaml:/etc/suricata/suricata.yaml
    command: -i eth0 -v  # Remplacer eth0 par votre interface réseau
    restart: unless-stopped

  # ===========================================
  # FILEBEAT - Log Shipper pour Suricata
  # ===========================================
  filebeat:
    image: docker.elastic.co/beats/filebeat:8.11.0
    container_name: filebeat
    user: root
    volumes:
      - ./filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - suricata-logs:/var/log/suricata:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
      - ids-network
    depends_on:
      elasticsearch:
        condition: service_healthy
    command: filebeat -e -strict.perms=false

  # ===========================================
  # WAZUH AGENT - Exemple pour monitorer l'hôte
  # ===========================================
  wazuh-agent:
    image: wazuh/wazuh-agent:4.7.0
    container_name: wazuh-agent-host
    hostname: docker-host
    environment:
      - WAZUH_MANAGER=wazuh-manager
      - WAZUH_AGENT_NAME=docker-host
    volumes:
      - /:/rootfs:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
      - ids-network
    depends_on:
      - wazuh-manager
```

---

### **ÉTAPE 4: Configuration Suricata**

Créez `~/ids-platform/suricata/suricata.yaml` :

```yaml
%YAML 1.1
---
vars:
  address-groups:
    HOME_NET: "[192.168.0.0/16,10.0.0.0/8,172.16.0.0/12]"
    EXTERNAL_NET: "!$HOME_NET"

af-packet:
  - interface: eth0  # Votre interface réseau
    cluster-id: 99
    cluster-type: cluster_flow
    defrag: yes
    use-mmap: yes
    tpacket-v3: yes

outputs:
  - eve-log:
      enabled: yes
      filetype: regular
      filename: /var/log/suricata/eve.json
      types:
        - alert:
            payload: yes
            payload-buffer-size: 4kb
            payload-printable: yes
            packet: yes
        - http:
            extended: yes
        - dns:
            version: 2
        - tls:
            extended: yes
        - files:
            force-magic: no
        - ssh

default-rule-path: /etc/suricata/rules
rule-files:
  - suricata.rules
  - emerging-threats.rules

logging:
  default-log-level: notice
  outputs:
    - console:
        enabled: yes
    - file:
        enabled: yes
        level: info
        filename: /var/log/suricata/suricata.log
```

---

### **ÉTAPE 5: Configuration Filebeat**

Créez `~/ids-platform/filebeat/filebeat.yml` :

```yaml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/suricata/eve.json
    json.keys_under_root: true
    json.add_error_key: true
    fields:
      type: suricata
    fields_under_root: true

output.elasticsearch:
  hosts: ["elasticsearch:9200"]
  index: "suricata-%{+yyyy.MM.dd}"

setup.template.name: "suricata"
setup.template.pattern: "suricata-*"
setup.ilm.enabled: false

logging.level: info
logging.to_files: true
```

---

### **ÉTAPE 6: Télécharger les règles Suricata**

```bash
cd ~/ids-platform/suricata/rules

# Télécharger les règles Emerging Threats
wget https://rules.emergingthreats.net/open/suricata-6.0/emerging.rules.tar.gz
tar -xzf emerging.rules.tar.gz
mv rules/*.rules .
rm -rf rules emerging.rules.tar.gz

# Créer un fichier de règles personnalisées
cat > suricata.rules << 'EOF'
# Custom Suricata Rules
alert icmp any any -> $HOME_NET any (msg:"ICMP Ping Detected"; sid:1000001; rev:1;)
alert tcp any any -> $HOME_NET 22 (msg:"SSH Connection Attempt"; sid:1000002; rev:1;)
alert http any any -> $HOME_NET any (msg:"HTTP Request Detected"; sid:1000003; rev:1;)
EOF
```

---

### **ÉTAPE 7: Démarrer la plateforme**

```bash
cd ~/ids-platform

# Vérifier la configuration
docker compose config

# Démarrer tous les services
docker compose up -d

# Vérifier que tout tourne
docker compose ps

# Voir les logs en temps réel
docker compose logs -f
```

---

### **ÉTAPE 8: Vérification et Tests**

```bash
# 1. Vérifier Elasticsearch
curl http://localhost:9200
curl http://localhost:9200/_cat/indices?v

# 2. Vérifier Wazuh Manager
docker exec -it wazuh-manager /var/ossec/bin/wazuh-control status

# 3. Vérifier Suricata
docker exec -it suricata suricata --build-info

# 4. Accéder aux dashboards
# Kibana: http://localhost:5601
# Wazuh Dashboard: https://localhost:8443
# Username: admin / Password: admin (par défaut)

# 5. Générer du trafic pour tester
ping -c 10 8.8.8.8
curl http://testmynids.org/uid/index.html  # Test NIDS

# 6. Vérifier les alertes dans Elasticsearch
curl "http://localhost:9200/suricata-*/_search?pretty" | head -50
```

---

### **ÉTAPE 9: Déployer des agents Wazuh sur d'autres machines**

Pour monitorer d'autres serveurs Linux :

```bash
# Sur chaque machine à monitorer
docker run -d \
  --name wazuh-agent \
  --hostname $(hostname) \
  -e WAZUH_MANAGER='<IP_DU_WAZUH_MANAGER>' \
  -e WAZUH_AGENT_NAME=$(hostname) \
  -v /:/rootfs:ro \
  -v /var/run/docker.sock:/var/run/docker.sock \
  wazuh/wazuh-agent:4.7.0
```

---

### **ÉTAPE 10: Configuration avancée (optionnel)**

```bash
# Activer la détection de rootkits
docker exec -it wazuh-manager vi /var/ossec/etc/ossec.conf
# Ajouter:
# <rootcheck>
#   <frequency>36000</frequency>
# </rootcheck>

# Redémarrer Wazuh
docker restart wazuh-manager

# Configurer les alertes Email
docker exec -it wazuh-manager vi /var/ossec/etc/ossec.conf
# Configurer la section <global> avec vos paramètres SMTP
```

---

## 🎯 ARCHITECTURE FINALE

Vous aurez :
- ✅ **1 Stack Docker** avec 7 containers
- ✅ **Suricata** analysant le trafic réseau
- ✅ **Wazuh Manager** recevant les logs des agents
- ✅ **Elasticsearch** stockant toutes les données
- ✅ **2 Dashboards** (Kibana + Wazuh) pour visualisation
- ✅ **Filebeat** pour transférer les logs Suricata
- ✅ **Agents Wazuh** sur chaque machine à monitorer

---

## 📊 COMMANDES UTILES

```bash
# Arrêter tout
docker compose down

# Arrêter et supprimer les volumes
docker compose down -v

# Redémarrer un service spécifique
docker compose restart wazuh-manager

# Voir les logs d'un service
docker compose logs -f suricata

# Entrer dans un container
docker exec -it wazuh-manager bash

# Sauvegarder les données
docker run --rm -v ids-platform_elasticsearch-data:/data -v $(pwd):/backup ubuntu tar czf /backup/elasticsearch-backup.tar.gz /data
```

Voulez-vous que je vous aide avec une partie spécifique ou que je crée des scripts d'automatisation supplémentaires ? 🚀
