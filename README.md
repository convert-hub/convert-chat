### Convert Chat 🤖

Plataforma de atendimento via WhatsApp construída a partir de um código-base consolidado, com organização em backend (Express) e frontend (React).

Este repositório usa o fluxo: dev (teste) → Pull Request → main (produção).
Produção nunca é editada manualmente — apenas recebe o que está no main.

🚀 Começando

Estrutura principal:

backend/ — API em Express, integra DB/Redis, filas, jobs etc.

frontend/ — app React (interface do operador/gestor).

scripts/ (opcional) — utilitários de instalação/deploy/backup.

Consulte Implantação em produção
 para publicar com segurança.

📋 Pré-requisitos

Node.js LTS (18.x ou 20.x)

npm (ou yarn/pnpm, se padronizar)

PostgreSQL 13+

Redis 6+

(Opcional) Docker / docker compose

(Opcional) PM2 para orquestrar em produção

🔧 Instalação dos serviços
Redis (via Docker)
# como root (ou com sudo)
docker run --name redis-app \
  -p 6379:6379 \
  --restart always -d \
  -e REDIS_ARGS="--requirepass ${REDIS_PASSWORD}" \
  redis:7 \
  redis-server --requirepass ${REDIS_PASSWORD}

Postgres
# criar DB e usuário (ajuste os nomes/senhas)
sudo -u postgres createdb ${APP_DB_NAME}
sudo -u postgres psql <<SQL
CREATE USER ${APP_DB_USER} WITH ENCRYPTED PASSWORD '${APP_DB_PASS}';
GRANT ALL PRIVILEGES ON DATABASE ${APP_DB_NAME} TO ${APP_DB_USER};
ALTER USER ${APP_DB_USER} CREATEDB;
SQL

🔐 Variáveis de ambiente

Nunca versione o .env. Suba um .env.example (sem segredos).

.env do backend (exemplo)
NODE_ENV=development
PORT=3001
PROXY_PORT=443

# URLs públicas do seu ambiente
BACKEND_URL=https://api.seu-dominio.com
FRONTEND_URL=https://app.seu-dominio.com

# Postgres
DB_DIALECT=postgres
DB_HOST=127.0.0.1
DB_PORT=5432
DB_USER=${APP_DB_USER}
DB_PASS=${APP_DB_PASS}
DB_NAME=${APP_DB_NAME}

# Auth
JWT_SECRET=troque-esta-chave
JWT_REFRESH_SECRET=troque-esta-chave-2

# Redis
REDIS_URI=redis://:${REDIS_PASSWORD}@127.0.0.1:6379
REDIS_OPT_LIMITER_MAX=1
REDIS_OPT_LIMITER_DURATION=3000

# Limites
USER_LIMIT=50
CONNECTIONS_LIMIT=5
CLOSED_SEND_BY_ME=true

# Pagamentos (exemplo, opcional)
GERENCIANET_SANDBOX=false
GERENCIANET_CLIENT_ID=
GERENCIANET_CLIENT_SECRET=
GERENCIANET_PIX_CERT=
GERENCIANET_PIX_KEY=

# E-mail (exemplo)
MAIL_HOST=smtp.gmail.com
MAIL_USER=seu@gmail.com
MAIL_PASS=SuaSenha
MAIL_FROM=seu@gmail.com
MAIL_PORT=465

.env do frontend
REACT_APP_BACKEND_URL=https://api.seu-dominio.com
REACT_APP_HOURS_CLOSE_TICKETS_AUTO=24


Observação: no README antigo havia DB_PASS=${mysql_root_password} e REDIS_URI usando a mesma senha. Aqui padronizamos para APP_DB_PASS (Postgres) e REDIS_PASSWORD (Redis).

🧩 Dependências
# backend
cd backend
npm install --force

# frontend
cd ../frontend
npm install --force

🧪 Rodando localmente (desenvolvimento)
# backend (watch/build podem variar conforme seu package.json)
cd backend
npm run watch   # ou npm run dev
npm start       # se necessário

# frontend
cd ../frontend
npm start

⚙️ Testes

(adicione aqui seus comandos de teste quando disponíveis, ex.: npm test, vitest, jest etc.)

📦 Implantação em produção

Produção puxa o main. Nunca edite “na mão”.
Acesse como usuário de deploy.

su - deploy

Frontend
cd /home/deploy/${APP_NAME}
pm2 stop ${APP_NAME}-frontend || true
git fetch origin --prune
git checkout main
git reset --hard origin/main

cd frontend
npm ci || npm install
rm -rf build
npm run build

pm2 start "npx serve -s build -l 3000" --name ${APP_NAME}-frontend || pm2 restart ${APP_NAME}-frontend
pm2 save


Se você usa Nginx como reverse proxy, aponte para a porta do frontend (ex.: 3000) ou sirva arquivos estáticos via Nginx apontando para frontend/build.

Backend
cd /home/deploy/${APP_NAME}
pm2 stop ${APP_NAME}-backend || true
git fetch origin --prune
git checkout main
git reset --hard origin/main

cd backend
npm ci || npm install
npm update -f || true
# instale tipos extras se o projeto exigir:
# npm install -D @types/fs-extra

rm -rf dist
npm run build

# Migrations/Seeds (Sequelize)
npx sequelize db:migrate
npx sequelize db:seed    # rode apenas se realmente precisar popular

pm2 start "node dist/server.js" --name ${APP_NAME}-backend || pm2 restart ${APP_NAME}-backend
pm2 save


Ajuste o entrypoint conforme seu projeto (ex.: dist/server.js, dist/index.js).

🌿 Fluxo de Git (recomendado)

dev → desenvolvimento (VPS de testes)

main → produção (branch protegido; exige PR)

Rotina:

Trabalhe na VPS de testes (branch dev), git add/commit/push.

Abra PR dev → main no GitHub e faça o merge.

Na produção, sincronize:

cd /home/deploy/${APP_NAME}
git fetch origin --prune
git checkout main
git reset --hard origin/main
# reinicie PM2 conforme acima


(Opcional) Configure GitHub Actions por SSH para publicar automaticamente ao fazer merge no main.

🛠️ Construído com

Express
 — backend

React
 — frontend

NPM
 — pacotes/gerenciador

(Opcional) PM2
 — processos em produção

📌 Versão

1.0.0 (baseline inicial deste repositório)

📄 Licença & Créditos

Se o código-base original possui licença, mantenha os termos e credite a origem.
Atualize esta seção com a licença adotada por este repositório (ex.: MIT) e links necessários.

🤝 Contribuindo

Abra uma issue descrevendo a mudança.

Crie um branch a partir de dev (ex.: feat/nome-da-feature).

Abra PR para dev. Após revisão, integramos em main.

Observações importantes

Padronize nomes das variáveis para evitar confusão (DB x Redis).

Evite rodar seed em produção sem necessidade.

Garanta que .env de produção e teste estão corretos e fora do Git.
