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





