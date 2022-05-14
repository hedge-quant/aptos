# Installation Aptos Validator + Fullnode

Il vous faut de préference un serveur VPS avec un SSD minimum 350 Go

On démarre avec déclaration des variables (copier et coller tout dans le terminal)

```bash
echo "export WORKSPACE=testnet" >> $HOME/.bash_profile
echo "export PUBLIC_IP=$(curl -s ifconfig.me)" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

Mis à jour des packages

```bash
sudo apt update && sudo apt upgrade -y
````

Installation des dépendances

```bash
sudo apt-get install jq unzip -y
````

Installation du Docker

```bash
sudo apt-get install ca-certificates curl gnupg lsb-release -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io -y
```

Installation du Docker Compose

```bash
mkdir -p ~/.docker/cli-plugins/
curl -SL https://github.com/docker/compose/releases/download/v2.2.3/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
chmod +x ~/.docker/cli-plugins/docker-compose
sudo chown $USER /var/run/docker.sock
```

Télécharger Aptos CLI

```bash
wget -qO aptos-cli.zip https://github.com/aptos-labs/aptos-core/releases/download/aptos-cli-v0.1.1/aptos-cli-0.1.1-Ubuntu-x86_64.zip
unzip -o aptos-cli.zip -d /usr/local/bin
chmod +x /usr/local/bin/aptos
rm aptos-cli.zip
```

## Création du Validateur

Créez le dossier Workspace

```bash
mkdir ~/$WORKSPACE && cd ~/$WORKSPACE
```

Télécharger les fichiers config

```bash
wget -qO docker-compose.yaml https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/aptos-node/docker-compose.yaml
wget -qO validator.yaml https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/aptos-node/validator.yaml
wget -qO fullnode.yaml https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/aptos-node/fullnode.yaml
```

On va générer les trois clés: private-keys.yaml, validator-identity.yaml, validator-full-node-identity.yaml

```bash
aptos genesis generate-keys --output-dir ~/$WORKSPACE
```

Configuration du validateur, mettez votre username + (même IP pour les validator + fullnode)

```bash
aptos genesis set-validator-configuration \
  --keys-dir ~/$WORKSPACE --local-repository-dir ~/$WORKSPACE \
  --username aptosbot \
  --validator-host $PUBLIC_IP:6180 \
  --full-node-host $PUBLIC_IP:6182
```

Génération des clés racines

```bash
mkdir keys
aptos key generate --output-file keys/root
```

Création du fichier layot (même username au lieu de aptosbot)

```bash
tee layout.yaml > /dev/null <<EOF
---
root_key: "0x5243ca72b0766d9e9cbf2debf6153443b01a1e0e6d086c7ea206eaf6f8043956"
users:
  - aptosbot
chain_id: 23
EOF
```

Téléchargez le dossier framework

```bash
wget -qO framework.zip https://github.com/aptos-labs/aptos-core/releases/download/aptos-framework-v0.1.0/framework.zip
unzip -o framework.zip
rm framework.zip
```

On compile le genesis 

```bash
aptos genesis generate-genesis --local-repository-dir ~/$WORKSPACE --output-dir ~/$WORKSPACE
```

Et on lance le Docker Compose

```bash
docker compose up -d
```

Pour toutes les questions: TG: @nickkauffmann

Nick

