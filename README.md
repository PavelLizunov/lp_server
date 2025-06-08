📘 LP_SERVER: Unix Setup Guide (Docker + Forge 1.20.1 + RCON)

🛠  Шаг 0. Подготовка клиента
• Удали/выключи моды: ChatHeads, ChatAnimation, TitlebarChanger – иначе краши на Unix.

📦  Шаг 1. Распакуй на сервере
```bash
unzip LP_SERVER.zip -d ~/LP_SERVER
cd ~/LP_SERVER
```

🐳  Шаг 2. Установи Docker + Compose
```bash
# Ubuntu / Debian
sudo apt update && sudo apt install docker.io docker-compose -y
sudo usermod -aG docker $USER && newgrp docker
```

📝  Шаг 3. Создай docker-compose.yml
```bash
cat > docker-compose.yml <<'YML'
services:
  minecraft:
    image: openjdk:17-jdk-slim
    container_name: lp_forge_server
    working_dir: /server
    volumes:
      - .:/server
    ports:
      - "${MC_SERVER_PORT:-25683}:${MC_SERVER_PORT:-25683}/tcp"
      - "24454:24454/udp"                      # Voice Chat
      - "${MC_RCON_PORT:-25575}:${MC_RCON_PORT:-25575}/tcp"
    environment:
      EULA: "TRUE"
    command: /server/run.sh
    restart: unless-stopped
YML
```

🖥  Шаг 4. Генерируем .env из server.properties
```bash
cat > gen-env.sh <<'SH'
#!/bin/bash
PROP=server.properties
[ ! -f $PROP ] && { echo "нет $PROP"; exit 1; }
get(){ grep -E "^$1=" $PROP | cut -d= -f2 ;}
cat <<EOF > .env
MC_SERVER_PORT=$(get server-port  || echo 25565)
MC_RCON_PORT=$(get rcon.port     || echo 25575)
MC_LEVEL_NAME=$(get level-name   || echo world)
EOF
echo ".env создан"; cat .env
SH
chmod +x gen-env.sh && ./gen-env.sh
```
⚙️  Шаг 5. Включи RCON (опц.)
```bash
sed -i '/enable-rcon=/c\enable-rcon=true' server.properties
sed -i '/rcon.password=/c\rcon.password=StrongPass123' server.properties
```

▶️  Шаг 6. Старт
```bash
docker compose --env-file .env up -d
docker logs -f lp_forge_server 
```

📁  Шаг 7. После первого запуска скопируй datapacks
```bash
WORLD="LP_SERVER/${MC_LEVEL_NAME:-world}"
mkdir -p "$WORLD/datapacks"
cp -vn LP_SERVER/datapacks/* "$WORLD/datapacks/"
docker restart lp_forge_server
```

🧩  Шаг 8. Проверка datapacks
# внутри игры или в docker attach
/datapack list

🔑  Шаг 9. RCON (опц.)
```bash
git clone https://github.com/Tiiffi/mcrcon && cd mcrcon && make && sudo install mcrcon /usr/local/bin
mcrcon -H 127.0.0.1 -P $MC_RCON_PORT -p StrongPass123
```
# пример: /say 
Проверка datapacks
# внутри RCON
/datapack list