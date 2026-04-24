Voici comment monter ton NAS WD My Cloud EX2 Ultra sous Nobara (qui est basé sur Fedora/RPM).

---

## 1. Installer les outils nécessaires

bash

```bash
sudo dnf install cifs-utils samba-client -y
```

---

## 2. Lister les partages disponibles sur le NAS

bash

```bash
smbclient -L //mycloudex2ultra.local -N
```

Cela t'affiche les noms des partages (ex: `Public`, `TimeMachineBackup`, etc.)

---

## 3. Créer le point de montage

bash

```bash
sudo mkdir -p /mnt/mycloud
```

---

## 4. Montage manuel (test)

**Sans authentification (partage public) :**

bash

```bash
sudo mount -t cifs //mycloudex2ultra.local/Public /mnt/mycloud -o guest,uid=$(id -u),gid=$(id -g),iocharset=utf8
```

**Avec identifiants :**

bash

```bash
sudo mount -t cifs //mycloudex2ultra.local/Public /mnt/mycloud \
  -o username=TON_USER,password=TON_MDP,uid=$(id -u),gid=$(id -g),iocharset=utf8
```

Remplace `Public` par le nom du partage que tu as trouvé à l'étape 2.

---

## 5. Montage automatique au démarrage (fstab)

Crée d'abord un fichier de credentials pour ne pas mettre ton mot de passe en clair :

bash

```bash
sudo nano /etc/samba/mycloud.creds
```

Contenu :

```
username=TON_USER
password=TON_MDP
```

Sécurise le fichier :

bash

```bash
sudo chmod 600 /etc/samba/mycloud.creds
```

Puis édite `/etc/fstab` :

bash

```bash
sudo nano /etc/fstab
```

Ajoute à la fin :

```
//mycloudex2ultra.local/Public  /mnt/mycloud  cifs  credentials=/etc/samba/mycloud.creds,uid=1000,gid=1000,iocharset=utf8,_netdev  0  0
```

L'option `_netdev` est importante — elle indique d'attendre que le réseau soit actif avant de monter.

Teste avec :

bash

```bash
sudo mount -a
```

---

## Notes

- Si tu as une erreur de résolution `.local`, essaie avec l'IP directe du NAS à la place de `mycloudex2ultra.local`
- Si erreur de version SMB, ajoute `vers=2.0` (ou `vers=3.0`) dans les options
- `uid=1000` correspond à ton utilisateur principal — vérifie avec `id -u`