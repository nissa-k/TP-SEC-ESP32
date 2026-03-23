# TP-SEC-ESP32 — Durcissement Linux pour infrastructure ESP32

> **Atelier Sécurité Linux Serveur**  
> Durcir un serveur Ubuntu dédié à l'administration d'une infrastructure IoT ESP32
> Karadag Nissa

---

## Objectif

Rendre l'infrastructure ESP32 plus sûre par défaut en appliquant les bonnes pratiques Linux serveur :

1. Système à jour (patches de sécurité)
2. Accès contrôlés (gestion utilisateurs, SSH durci)
3. Pare-feu restrictif (UFW + WireGuard)
4. Traçabilité complète (journaux SSH, auditd)

---

## Structure du TP

```
TP-SEC-ESP32/
├── README.md               ← Ce fichier
├── compte_rendu_ESP32.docx ← Rapport complet avec captures
└── sections/
    ├── 1_mises_a_jour.md
    ├── 2_gestion_utilisateurs.md
    ├── 3_configuration_ssh.md
    ├── 4_pare_feu_ufw.md
    └── 5_logs_audit.md
```

---

## Commandes utilisées

### 1 · Mise à jour du système

```bash
sudo apt update
sudo apt full-upgrade -y
sudo apt autoremove -y
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure unattended-upgrades   # Activer les MAJ auto de sécurité
```

### 2 · Gestion des utilisateurs

```bash
sudo adduser admin                          # Créer un compte admin dédié
sudo usermod -aG sudo admin                 # Donner les droits sudo
groups admin                                # Vérification
```

> **Politique de mot de passe recommandée** (cybermalveillance.gouv.fr) :  
> minimum 12 caractères, majuscules + minuscules + chiffres + caractères spéciaux

### 3 · Durcissement SSH (`/etc/ssh/sshd_config`)

```bash
sudo nano /etc/ssh/sshd_config
```

Paramètres à décommenter / modifier :

```
LoginGraceTime 30
PermitRootLogin no
StrictModes yes
MaxAuthTries 3
PasswordAuthentication yes
PermitEmptyPasswords no
X11Forwarding no
UsePAM yes
```

```bash
sudo sshd -t                                # Vérification syntaxique
sudo systemctl restart ssh
sudo systemctl status ssh
```

### 4 · Configuration Pare-feu UFW

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 51820/udp                    # WireGuard VPN
sudo ufw allow from 10.10.10.0/24 to any port 22   # SSH via VPN seulement
sudo ufw enable
sudo ufw status
```

### 5 · Vérification des logs SSH

```bash
sudo journalctl -u ssh                      # Logs du service SSH
sudo cat /var/log/auth.log                  # Authentifications système
```

### 6 · Installation et configuration de auditd

```bash
sudo apt install auditd audispd-plugins -y
sudo systemctl enable auditd
sudo systemctl start auditd
sudo systemctl status auditd
sudo systemctl is-enabled auditd

sudo auditctl -l                            # Lister les règles actives
sudo auditctl -w /etc/passwd -p wa -k passwd_changes   # Surveiller /etc/passwd
sudo ausearch -k passwd_changes             # Rechercher les événements
```

### 7 · Test de validation

```bash
sudo vipw                                   # Ouvrir /etc/passwd de façon sécurisée
sudo ausearch -k passwd_changes             # Vérifier la traçabilité
whoami
groups
```

---

## Architecture réseau

```
[Client Windows]
     │
     │  WireGuard VPN (port 51820/udp)
     │  Tunnel : 10.10.10.2 → 10.10.10.1
     ▼
[Serveur Ubuntu ESP32]
  IP VPN : 10.10.10.1/24
  SSH    : port 22 (autorisé UNIQUEMENT depuis 10.10.10.0/24)
  MQTT   : port 8883/tcp (Anywhere)
```

---

## Règles UFW appliquées

| Port       | Protocole | Action | Source          |
|------------|-----------|--------|-----------------|
| 8883       | tcp       | ALLOW  | Anywhere        |
| 51820      | udp       | ALLOW  | Anywhere        |
| 22         | —         | ALLOW  | 10.10.10.0/24   |
| 22         | tcp       | DENY   | Anywhere        |

---

## Résultats obtenus

| Mesure                          | Statut |
|---------------------------------|--------|
| Système mis à jour              |        |
| MAJ automatiques activées       |        |
| Compte root SSH désactivé       |        |
| Utilisateur admin sudo créé     |        |
| SSH durci (MaxAuthTries, etc.)  |        |
| Pare-feu UFW actif              |        |
| SSH restreint au VPN            |        |
| Journaux SSH vérifiés           |        |
| auditd installé et actif        |        |
| Règle audit /etc/passwd         |        |
| Test traçabilité validé         |        |

---

## Références

- [cybermalveillance.gouv.fr](https://www.cybermalveillance.gouv.fr/) — Politique de mots de passe robustes
- [Ubuntu Server Guide — UFW](https://ubuntu.com/server/docs/security-firewall)
- [OpenSSH Manual — sshd_config](https://man.openbsd.org/sshd_config)
- [Linux Audit Documentation](https://github.com/linux-audit/audit-documentation)
- [WireGuard](https://www.wireguard.com/)

---

## Informations

| Champ        | Valeur                          |
|--------------|---------------------------------|
| Cours        | Sécurité Linux Serveur          |
| École        | Numeryx University              |
| VM           | Ubuntu (esp32-VirtualBox)       |
| Utilisateur  | esp32 / admin                   |
| Date         | Décembre 2025                   |
