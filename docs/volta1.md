Para automatizar a criação das pastas e arquivos diretamente no GitHub e localmente, podemos seguir este fluxo:

1️⃣ Criar um repositório no GitHub (se ainda não existir).
2️⃣ Gerar a estrutura de diretórios e arquivos localmente via script.
3️⃣ Fazer commit e push para o GitHub automaticamente.
4️⃣ Clonar o repositório e executar o docker-compose.yml.


---

1️⃣ Criar o Repositório no GitHub

Se o repositório ainda não existe, crie via API do GitHub:

🔹 Substitua SEU_GITHUB_TOKEN pelo seu token pessoal e BetMasterAI pelo nome do repositório.

curl -H "Authorization: token SEU_GITHUB_TOKEN" \
     -H "Accept: application/vnd.github.v3+json" \
     https://api.github.com/user/repos \
     -d '{"name":"BetMasterAI","private":false}'

✅ Isso cria um repositório público chamado BetMasterAI.


---

2️⃣ Criar a Estrutura de Pastas e Arquivos Localmente

Crie um script para gerar os diretórios e arquivos:

#!/bin/bash

# Criando diretórios
mkdir -p BetMasterAI/{backend/{app,event_bus,tests},frontend/{public,src/components}}

# Criando arquivos backend
echo 'from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "FastAPI está rodando!"}' > BetMasterAI/backend/app/main.py

echo 'from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers="kafka:9092")

def send_message(topic, message):
    producer.send(topic, message.encode("utf-8"))
    producer.flush()

send_message("test_topic", "Mensagem de teste do producer!")' > BetMasterAI/backend/event_bus/kafka_producer.py

echo 'from kafka import KafkaConsumer

consumer = KafkaConsumer("test_topic", bootstrap_servers="kafka:9092")

for message in consumer:
    print(f"Mensagem recebida: {message.value.decode("utf-8")}")' > BetMasterAI/backend/event_bus/kafka_consumer.py

# Criando arquivos frontend
echo '<!DOCTYPE html>
<html>
<head>
    <title>BetMasterAI</title>
    <link rel="stylesheet" type="text/css" href="styles.css">
</head>
<body>
    <h1>Bem-vindo ao BetMasterAI</h1>
</body>
</html>' > BetMasterAI/frontend/public/index.html

echo 'body { font-family: Arial, sans-serif; }' > BetMasterAI/frontend/public/styles.css

# Criando Docker-Compose
echo 'version: "3.8"
services:
  fastapi:
    build: ./backend
    ports:
      - "8000:8000"

  zookeeper:
    image: wurstmeister/zookeeper
    ports:
      - "2181:2181"

  kafka:
    image: wurstmeister/kafka
    ports:
      - "9092:9092"
    environment:
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181' > BetMasterAI/docker-compose.yml

# Criando README
echo '# BetMasterAI
Automação de infraestrutura com Kafka e FastAPI.' > BetMasterAI/README.md

echo "✅ Estrutura criada com sucesso!"

📌 Salve esse script como setup.sh e execute:

chmod +x setup.sh && ./setup.sh


---

3️⃣ Fazer Commit e Enviar para o GitHub

cd BetMasterAI
git init
git remote add origin https://github.com/seu-usuario/BetMasterAI.git
git add .
git commit -m "Automação da criação da infraestrutura"
git branch -M main
git push -u origin main

✅ Agora o repositório está atualizado no GitHub!


---

4️⃣ Clonar e Subir os Containers

git clone https://github.com/seu-usuario/BetMasterAI.git
cd BetMasterAI
docker-compose up -d

✅ Infraestrutura pronta para testes e implantação automática! 🚀

