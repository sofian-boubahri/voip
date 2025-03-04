# 📞 Installation et Configuration d'un Serveur VoIP avec Asterisk

Ce document fournit un guide détaillé pour l’installation, la configuration et la sécurisation d’un serveur VoIP basé sur Asterisk.

---

## 🔹 Introduction

Asterisk est un logiciel open-source permettant de mettre en place un serveur VoIP. Ce tutoriel vous guidera à travers l'installation et la configuration d'Asterisk sous Linux.

---

## 🚀 Installation d'Asterisk

### 1. Installation des dépendances
```sh
sudo apt update && sudo apt upgrade -y
sudo apt install build-essential libxml2-dev libncurses5-dev \
linux-headers-$(uname -r) libsqlite3-dev libssl-dev \
libedit-dev uuid-dev libjansson-dev pkg-config
```

### 2. Téléchargement et installation
```sh
mkdir -p /usr/src/asterisk
cd /usr/src/asterisk
wget https://downloads.asterisk.org/pub/telephony/asterisk/asterisk-22-current.tar.gz
tar -xvzf asterisk-22-current.tar.gz
cd asterisk-22.2.0/
```

### 3. Compilation et installation
```sh
sudo ./configure --with-jansson-bundled
sudo make menuselect
sudo make -j$(nproc)
sudo make install
sudo make samples
sudo make config
```

### 4. Démarrage d'Asterisk
```sh
sudo systemctl enable asterisk
sudo systemctl start asterisk
sudo asterisk -rvvvvv
```

---

## 👥 Configuration des utilisateurs

Modifier `pjsip.conf` :
```sh
cd /etc/asterisk
sudo nano pjsip.conf
```

Ajouter :
```ini
[general]
language=fr
allowguest=no
auth_type=userpass

[6001]
type=endpoint
aors=6001
auth=auth6001
context=internal
disallow=all
allow=ulaw

[auth6001]
type=auth
auth_type=userpass
username=6001
password=supersecret

[6001_aor]
type=aor
max_contacts=1
```

Redémarrer Asterisk :
```sh
sudo systemctl restart asterisk
```

---

## 🔒 Sécurisation des appels par TLS

### 1. Génération des certificats
```sh
sudo mkdir -p /etc/asterisk/keys
cd /etc/asterisk/keys
openssl req -x509 -newkey rsa:2048 -keyout private.key -out certificate.pem -days 365 -nodes -subj "/CN=asterisk"
```

### 2. Configuration TLS dans `pjsip.conf`
```ini
[transport-tls]
type=transport
protocol=tls
bind=0.0.0.0:5061
cert_file=/etc/asterisk/keys/certificate.pem
priv_key_file=/etc/asterisk/keys/private.key
method=tlsv1_2
```

Redémarrer Asterisk :
```sh
sudo systemctl restart asterisk
```

---

## 🖥️ Configuration sur MicroSIP

Configurer un compte SIP avec :
- **Serveur SIP** : IP du serveur
- **Nom d’utilisateur** : `6001`
- **Mot de passe** : `supersecret`
- **Transport** : `TLS`

Une fois connecté, vous devriez voir un statut **"Enregistré"**.

---

## ✅ Test et validation

1. Vérifier la connexion (`pjsip show endpoints`).
2. Tester un appel entre utilisateurs.
3. Vérifier l'audio et la stabilité de l'appel.
4. Tester le redémarrage du serveur et la reconnexion.

---

## 🤖 Automatisation et déploiement

Créer un script `script.sh` pour automatiser les appels :
```sh
nano script.sh
```

Coller ce contenu :
```sh
#!/bin/bash

CSV_FILE="contact.csv"
CALLS_DIR="/var/spool/asterisk/outgoing/"
CALLER_ID="9000"

if [[ ! -f "$CSV_FILE" ]]; then
    echo "❌ Erreur : le fichier $CSV_FILE n'existe pas."
    exit 1
fi

tail -n +2 "$CSV_FILE" | while IFS=, read -r NAME NUMBER; do
    if [[ -z "$NAME" || -z "$NUMBER" ]]; then
        continue
    fi
    CALL_FILE="/tmp/call_$NUMBER.call"
    cat <<EOF > "$CALL_FILE"
Channel: PJSIP/$NUMBER
CallerID: "Prospection" <$CALLER_ID>
MaxRetries: 2
RetryTime: 60
WaitTime: 30
Context: auto_calls
Extension: s
Priority: 1
EOF
    chmod 777 "$CALL_FILE"
    mv "$CALL_FILE" "$CALLS_DIR/"
    echo "✅ Appel généré pour $NAME ($NUMBER)"
done
```

Donner les permissions et exécuter :
```sh
chmod +x script.sh
./script.sh
```

---

## 📝 Conclusion

Ce guide vous a permis d'installer et de sécuriser un serveur VoIP avec Asterisk. Vous pouvez maintenant tester et optimiser votre infrastructure.

Bonne configuration ! 🚀

