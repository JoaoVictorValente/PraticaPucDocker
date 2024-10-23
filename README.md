# PraticaPucDocker

O repositório tem como objetivo demonstrar o uso das tecnologias de conteinerização, utilizando docker compose, buscando demonstrar as boas práticas e suas funcionalidades.

# Autor: 
João Victor Pereira Valente

# Matrícula 
206479 

# Instruções de inicialização 

1- Clonar o repositório em sua máquina local, lembrando que é necessário possuir o docker e docker compose instalados em sua máquina.

```shell
git clone https://github.com/JoaoVictorValente/PraticaPucDocker.git
```

2- Entrar dentro do diretório do projeto.

```shell
cd PraticaPucDocker
```

3- Startar os containers 

```shell
docker compose up -d
```

# Acessar a aplicação 

A aplicação estará acessível no navegador em localhost na porta 80 : http://localhost

# O Jogo

A dinâmica do jogo, é criar uma nova partida no "Maker" inserindo frases, guarde o id do game. Na parte "breaker" insira o id gerado, o próximo passo é tentar advinhar a frase.

# Arquitetura 

# DOCKER-COMPOSE.YML

Foi criado um docker-compose.yml na raiz do projeto, o mesmo tem algumas configurações importantes.

Estrutura e Descrição
Version:

version: '3.8': Define a versão do Docker Compose que está sendo usada. Essa versão determina as funcionalidades disponíveis e a sintaxe suportada.
Services:

A seção services descreve os containers que serão executados. Cada serviço representa um container que será gerenciado pelo Docker Compose.
frontend:

build: Define como construir a imagem do container a partir de um Dockerfile localizado no diretório ./frontend-app. A variável NODE_ENV é definida como production.

container_name: Nome do container será frontendapp.

ports: O serviço expõe a porta 80 do container para a porta 80 do host.

networks: Conectado à rede app_net.

restart: Reinicia o container a menos que ele seja manualmente parado.

depends_on: Garante que os serviços backendapp1 e backendapp2 sejam iniciados antes do frontend.

healthcheck: Define um teste para verificar se o container está funcionando corretamente, fazendo uma requisição curl à porta 80.

backendapp1 e backendapp2:

Ambos os serviços têm configuração semelhante:

build: Constrói a imagem a partir de um Dockerfile localizado em ./backend-app.

expose: Expõe a porta 5000 internamente na rede Docker.

networks: Conectado às redes app_net e db_net.

restart: Reinicia automaticamente a menos que parado manualmente.

environment: Define variáveis de ambiente para configurar o ambiente Flask e a conexão com o banco de dados.

depends_on: Certifica-se de que o postgres seja iniciado antes.

healthcheck: Verifica a saúde do serviço ao tentar acessar a rota /health na porta 5000.

postgres:
image: Utiliza a imagem postgres:17.0-alpine.

container_name: Nome do container será postgresdb.

restart: Sempre reinicia o container em caso de falha.

ports: Expõe a porta 5432 do container para a porta 5432 do host.

networks: Conectado à rede db_net.

environment: Define a senha para o banco de dados.

volumes: Persistência de dados usando o volume postgres-data.

healthcheck: Verifica se o banco de dados está pronto utilizando o comando pg_isready.

Networks:
Define duas redes:
app_net: Para comunicação entre o frontend e os backends.

db_net: Para comunicação entre os backends e o banco de dados.

Volumes:
postgres-data: Volume nomeado para armazenar os dados do banco de dados PostgreSQL de forma persistente.

# Dockerfile Backend

FROM python:3.9-alpine

Baseia a imagem no Python 3.9 utilizando a versão alpine, que é uma versão mínima e leve do sistema operacional, ideal para reduzir o tamanho da imagem final.
WORKDIR /app

Define o diretório de trabalho dentro do container como /app. Todos os comandos subsequentes serão executados a partir deste diretório.
COPY requirements.txt .

Copia o arquivo requirements.txt do host para o diretório /app dentro do container. Este arquivo geralmente contém as dependências necessárias para o projeto Python.
RUN pip install --no-cache-dir -r requirements.txt

Instala as dependências listadas em requirements.txt usando o pip, sem armazenar cache, para manter a imagem leve.
COPY . .

Copia todos os arquivos e diretórios do contexto atual (onde está o Dockerfile) para o diretório /app dentro do container.
ENV PYTHONUNBUFFERED=1

Define uma variável de ambiente para o Python, que desativa o buffer de saída (stdout e stderr). Isso faz com que a saída seja mostrada em tempo real, útil para registros de logs.
RUN apk update && apk upgrade && apk add --no-cache curl wget

Executa a atualização do sistema e instala os pacotes curl e wget, usando o gerenciador de pacotes apk do Alpine Linux. O --no-cache impede que pacotes intermediários sejam armazenados, mantendo a imagem mais enxuta.
EXPOSE 5000

Informa que o container irá usar a porta 5000. Isso não abre a porta diretamente, mas indica que a aplicação espera que conexões sejam feitas nessa porta.
CMD ["sh", "./start-backend.sh"]

Define o comando padrão que será executado quando o container for iniciado. Neste caso, ele irá executar o script start-backend.sh, que provavelmente inicializa o backend.

# Dockerfile frontend

FROM node:20-alpine AS build

Inicia a fase de build usando a imagem node:20-alpine. Essa é uma imagem leve que contém Node.js e é usada para construir a aplicação React. O uso de AS build permite nomear esta etapa para referência posterior.
WORKDIR /app

Define o diretório de trabalho como /app dentro do container. Todos os comandos subsequentes serão executados nesse diretório.
COPY package*.json ./

Copia os arquivos package.json e package-lock.json para o diretório de trabalho dentro do container. Isso é feito antes de copiar o código completo para permitir que o Docker cache as dependências e evite a reinstalação desnecessária.
RUN npm ci --only=production && npx update-browserslist-db@latest

Instala as dependências necessárias em modo de produção usando npm ci, que garante uma instalação consistente com o package-lock.json.
Executa o comando npx update-browserslist-db@latest para atualizar a lista de navegadores suportados, garantindo que o código gerado seja compatível.
COPY . .

Copia todo o código do projeto para o diretório de trabalho /app dentro do container.
ARG REACT_APP_BACKEND_URL=http://localhost

Define uma variável de build chamada REACT_APP_BACKEND_URL com um valor padrão. Esse valor pode ser sobrescrito durante o build.
ENV REACT_APP_BACKEND_URL=$REACT_APP_BACKEND_URL

Define a variável de ambiente REACT_APP_BACKEND_URL para uso no processo de build do React, permitindo que a URL do backend seja configurada dinamicamente.
RUN npm run build

Executa o script de build do React, que gera os arquivos otimizados para produção no diretório build. Essa é a versão final da aplicação que será servida.
FROM nginx:1.27-alpine

Inicia uma nova etapa de build, agora usando uma imagem nginx:1.27-alpine. Esta é uma imagem menor e focada em servir conteúdo estático, ideal para produção.
COPY --from=build /app/build /usr/share/nginx/html

Copia os arquivos estáticos gerados na fase anterior para o diretório onde o NGINX serve o conteúdo, /usr/share/nginx/html. Essa abordagem de multi-stage build resulta em uma imagem final mais leve, pois apenas os arquivos necessários são incluídos.
COPY nginx.conf /etc/nginx/nginx.conf

Copia um arquivo de configuração personalizado do NGINX para sobrescrever o padrão, permitindo ajustes no comportamento do servidor.
EXPOSE 80

Declara que o container estará disponível na porta 80, a porta padrão para servir conteúdo HTTP.
CMD ["nginx", "-g", "daemon off;"]

Define o comando padrão para iniciar o NGINX no container, garantindo que ele rode em primeiro plano (daemon off) e continue ativo.


