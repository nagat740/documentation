### Installation du serveur wazuh

## Obligation
- Serveur ubuntu
- 4Go ram minimun
- Renommer la machine srv-wazuh

## Mettre a l'heure la machine
- Vérifier que la machien est à l'heure
```bash
timedatectl
```

- Configurer la zone de temps (si besoin)
```bash
timedatectl set-timezone Europe/Paris
```

- Synchronisez le serveur au serveur NTP
modifier le fichier /etc/systemd/timesyncd.conf.
```bash
nano /etc/systemd/timesyncd.conf.
```

- En dessous de la ligne [Time]
définisez la variable (NTP=) à ntp.univ-rennes2.fr

- Activer NTP
```bash
timedatectl set-ntp true
systemctl restart systemd-timesyncd.service
timedatectl timesync-status
```

## Installation de wazuh
- Exécution du script d'installatio du wazuh
```bash
curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```

- Une fois l’installation terminée, la sortie affiche les informations d’identification d’accès et un message confirmant que
l’installation a réussi

# Accéder a l'interface de wazuh
- https://ip-wazuh/

## Commande de secours
- Retrouver le mot de passe admin
```bash
sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt
```

- modifier le mot de passe admin
```bash
cd /usr/share/wazuh-indexer/plugins/opensearch-security/tools
bash wazuh-passwords-tool.sh -u admin -p password
```