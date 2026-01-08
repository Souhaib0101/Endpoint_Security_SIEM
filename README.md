# Sécurité des endpoints et supervision SIEM : étude de cas multi-OS (Linux & Windows)




# Plateforme SIEM/EDR avec Wazuh sur AWS

Déploiement d'une solution de surveillance de sécurité centralisée combinant SIEM (Security Information and Event Management) et EDR (Endpoint Detection and Response) pour environnements Linux et Windows dans le cloud AWS.

## Vue d'ensemble

Ce projet implémente une architecture complète de détection et réponse aux incidents utilisant Wazuh comme plateforme centrale. L'infrastructure permet la surveillance en temps réel, la corrélation d'événements et l'analyse des menaces sur des endpoints hétérogènes.

### Objectifs

- Centraliser la collecte et l'analyse d'événements de sécurité
- Détecter les comportements suspects et tentatives d'intrusion
- Surveiller l'intégrité des fichiers critiques
- Fournir une visibilité unifiée sur l'infrastructure

## Architecture

### Composants déployés

**Serveur Wazuh** (Ubuntu 22.04 - t3.large)

- Wazuh Manager : collecte et corrélation des événements
- Wazuh Indexer : stockage et indexation des données
- Wazuh Dashboard : interface de visualisation

**Endpoints surveillés**

- **Linux-Client** : Ubuntu 22.04 (t2.micro) - surveillance SSH, FIM, sudo
- **Windows-Client** : Windows Server 2022 (t2.micro) - surveillance RDP, Sysmon, gestion des identités

### Flux de communication

```
Agents (Linux/Windows)
    ↓ TCP 1514 (événements)
    ↓ TCP 1515 (enregistrement)
Wazuh Manager
    ↓
Wazuh Indexer → Wazuh Dashboard
```

## Architecture Wazuh all-in-one 

![[schema.png]]


## Déploiement rapide

### Installation du serveur Wazuh

```bash
# Connexion SSH au serveur
ssh -i wazuh-key.pem ubuntu@<IP_publique>

# Mise à jour système
sudo apt update && sudo apt -y upgrade

# Installation all-in-one
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash wazuh-install.sh -a
```

**Important** : Sauvegarder les identifiants affichés en fin d'installation.

### Installation agent Linux

```bash
# Sur le client Linux
sudo apt install apt-transport-https curl gnupg

curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo apt-key add -
echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list

sudo apt update
sudo WAZUH_MANAGER="<IP_privée_serveur>" apt install wazuh-agent
sudo systemctl start wazuh-agent
```

### Installation agent Windows

1. Télécharger l'installeur MSI depuis [packages.wazuh.com](https://packages.wazuh.com/)
2. Exécuter l'installeur via l'interface graphique
3. Configurer l'adresse IP du manager
4. Démarrer le service :

```powershell
NET START WazuhSvc
```

### Installation Sysmon (Windows)

```powershell
# Télécharger Sysmon depuis Microsoft Sysinternals
.\Sysmon64.exe -accepteula -i

# Configurer l'agent Wazuh
# Éditer C:\Program Files (x86)\ossec-agent\ossec.conf
# Ajouter la section Sysmon dans <localfile>
# Redémarrer l'agent
```

## Configuration sécurité

### Security Groups AWS

**Wazuh-Server-SG**

- SSH (22) : IP administrateur uniquement
- HTTPS (443) : IP administrateur uniquement
- TCP 1514, 1515 : agents Wazuh

**Linux-Client-SG**

- SSH (22) : IP administrateur uniquement
- Sortant vers serveur : 1514, 1515

**Windows-Client-SG**

- RDP (3389) : IP administrateur uniquement
- Sortant vers serveur : 1514, 1515

## Accès au Dashboard

1. Naviguer vers `https://<IP_publique_serveur>`
2. Accepter le certificat auto-signé
3. Se connecter avec les identifiants admin

**Sections principales :**

- **Overview** : vue globale des alertes et événements
- **Security Events** : détails des incidents détectés
- **Integrity Monitoring** : modifications de fichiers
- **Agents** : gestion et statut des endpoints

## Scénarios de validation

### Test 1 : Échecs d'authentification SSH

```bash
# Depuis une machine externe
ssh utilisateur_incorrect@<IP_Linux_Client>
```

**Résultat attendu** : Alertes de tentatives SSH échouées

### Test 2 : Échecs RDP

Tentatives RDP avec mot de passe incorrect vers Windows-Client

**Résultat attendu** : Alertes d'authentification Windows

### Test 3 : Élévation de privilèges

```powershell
# Sur Windows-Client
New-LocalUser -Name "testuser" -NoPassword
Add-LocalGroupMember -Group "Administrators" -Member "testuser"
```

**Résultat attendu** : Alertes de création d'utilisateur et ajout au groupe Administrators

### Test 4 : Modification fichiers sensibles

```bash
# Sur Linux-Client
sudo echo "test" | sudo tee -a /etc/passwd
```

**Résultat attendu** : Alertes FIM sur /etc/passwd

## Fonctionnalités clés

-  Détection d'intrusions en temps réel
- Surveillance de l'intégrité des fichiers (FIM)
- Corrélation d'événements multi-plateformes
- Analyse comportementale avec Sysmon
- Alertes configurables par niveau de sévérité
- Génération de rapports PDF
- Recherche et filtrage avancés

## Maintenance

### Vérifier l'état des services

**Sur le serveur :**

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

**Sur les agents :**

```bash
# Linux
sudo systemctl status wazuh-agent

# Windows (PowerShell)
Get-Service WazuhSvc
```




---

**Auteur** : Souhaib MONTAKI
Étudiant ingénieur – Génie du Logiciel et des Systèmes Informatiques Distribués
ENSET Mohammedia
**Date** : Janvier 2026
