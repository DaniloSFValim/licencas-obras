
# 🚀 GUIA COMPLETO: SISTEMA DE LICENÇAS PARA OBRAS

## ÍNDICE
1. Pré-requisitos
2. Instalação Backend
3. Instalação Frontend
4. Configuração do Banco de Dados
5. Testes Iniciais
6. Deploy em Produção
7. Troubleshooting

---

## 1️⃣ PRÉ-REQUISITOS

### Tecnologias Necessárias
- **Node.js 18+** - [Download](https://nodejs.org/)
- **PostgreSQL 12+** - [Download](https://www.postgresql.org/download/)
- **Git** - [Download](https://git-scm.com/)
- **Postman** (opcional, para testar API) - [Download](https://www.postman.com/)

### Verificar Instalação
```bash
node --version
npm --version
psql --version
```

---

## 2️⃣ INSTALAÇÃO BACKEND

### Passo 1: Clonar/Preparar Repositório
```bash
cd backend
npm install
```

### Passo 2: Configurar Variáveis de Ambiente
```bash
cp .env.example .env
```

Editar `.env`:
```
DB_HOST=localhost
DB_PORT=5432
DB_NAME=licencas_obras
DB_USER=postgres
DB_PASSWORD=SUA_SENHA_AQUI
PORT=5000
JWT_SECRET=chave_super_secreta_longa_aleatoriaXXXXXXXX
JWT_EXPIRY=7d
NODE_ENV=development
```

### Passo 3: Criar Banco de Dados
```bash
# Abrir PostgreSQL
psql -U postgres

# No prompt do PostgreSQL:
CREATE DATABASE licencas_obras;
\c licencas_obras
CREATE EXTENSION postgis;
CREATE EXTENSION postgis_topology;
\q
```

### Passo 4: Executar Schema SQL
```bash
psql -U postgres -d licencas_obras -f schema.sql
```

**Verá:**
```
CREATE TABLE
CREATE INDEX
...
INSERT 0 1  ✅ (Admin criado)
```

### Passo 5: Iniciar Servidor Backend
```bash
npm run dev
```

**Esperado:**
```
==================================================
✅ Servidor iniciado na porta 5000
📍 http://localhost:5000
==================================================

✅ Conectado ao banco de dados: 2024-04-15...
📅 Iniciando cron jobs...
   - Verificação de expiração: Todos os dias às 2:00 AM
✅ Cron jobs iniciados com sucesso
```

### Teste Rápido Backend
```bash
curl http://localhost:5000/health
```

**Resposta esperada:**
```json
{
  "sucesso": true,
  "mensagem": "Sistema funcionando corretamente",
  "timestamp": "2024-04-15T..."
}
```

---

## 3️⃣ INSTALAÇÃO FRONTEND

### Passo 1: Instalar Dependências
```bash
cd frontend
npm install
```

### Passo 2: Configurar Variáveis de Ambiente
```bash
cp .env.example .env.local
```

Editar `.env.local`:
```
VITE_API_BASE_URL=http://localhost:5000/api
VITE_MAP_CENTER_LAT=-22.8835
VITE_MAP_CENTER_LNG=-43.1072
VITE_MAP_ZOOM=12
```

### Passo 3: Iniciar Servidor Frontend
```bash
npm run dev
```

**Esperado:**
```
VITE v5.0.0  building for development
➜  Local:   http://localhost:5173/
➜  Press h to show help
```

### Teste Rápido Frontend
Abra navegador: `http://localhost:5173`

---

## 4️⃣ CONFIGURAÇÃO DO BANCO DE DADOS

### Importar Shapefile de Niterói (Opcional)

Se você tiver o arquivo `.shp`, converta para GeoJSON:

```bash
# Instalar ogr2ogr (Windows/Mac/Linux)
# Windows: https://trac.osgeo.org/osgeo4w/
# Mac: brew install gdal
# Linux: apt-get install gdal-bin

ogr2ogr -f GeoJSON Limite_Municipal.geojson Limite_Municipal.shp
```

Depois insira no banco:
```sql
INSERT INTO limite_municipal (nome, geometria) 
SELECT 'Niterói', ST_GeomFromGeoJSON('{"type":"Feature","geometry":...}');
```

### Verificar Dados no Banco

```bash
# Conectar ao banco
psql -U postgres -d licencas_obras

# Ver tabelas
\dt

# Ver usuário admin
SELECT id, nome, email, role FROM usuarios;

# Ver empresas (vazio no início)
SELECT * FROM empresas;

# Ver licenças (vazio no início)
SELECT * FROM licencas;
```

---

## 5️⃣ TESTES INICIAIS

### 5.1 Teste de Login

**Credentials Padrão:**
- Email: `admin@niteroi.rj.gov.br`
- Senha: `password123`

```bash
# Via Postman ou curl
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@niteroi.rj.gov.br",
    "senha": "password123"
  }'
```

**Resposta esperada:**
```json
{
  "sucesso": true,
  "mensagem": "Login realizado com sucesso",
  "dados": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "usuario": {
      "id": 1,
      "nome": "Administrador",
      "email": "admin@niteroi.rj.gov.br",
      "role": "admin"
    }
  }
}
```

### 5.2 Teste Via Interface Web

1. Abra `http://localhost:5173`
2. Clique em "Entrar"
3. Use email/senha padrão
4. Veja o Dashboard

### 5.3 Criar uma Empresa (Teste)

**Via Frontend:**
1. Dashboard → Empresas
2. Botão "Nova Empresa"
3. Preencha:
   - Nome: "Construtora ABC"
   - CNPJ: "12345678000190"
   - Endereço: "Rua das Flores, 123"
4. Salvar

**Via API:**
```bash
TOKEN="seu_token_aqui"

curl -X POST http://localhost:5000/api/empresas \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "nome": "Construtora ABC",
    "cnpj": "12345678000190",
    "telefone": "2133334444",
    "email": "contato@construtora.com.br",
    "endereco": "Rua das Flores, 123",
    "bairro": "Icaraí"
  }'
```

### 5.4 Emitir uma Licença

**Dados necessários:**
- Empresa (criada acima)
- Latitude: -22.8835
- Longitude: -43.1072
- Datas válidas
- Responsável da obra

---

## 6️⃣ DEPLOY EM PRODUÇÃO

### Opção 1: Render.com (Recomendado - Free Tier)

#### Backend no Render

1. Criar conta em [render.com](https://render.com)
2. Conectar GitHub
3. Novo "Web Service"
4. Selecionar repositório
5. Configurar:
   - Build Command: `npm install`
   - Start Command: `npm run dev`
   - Environment Variables:
     ```
     DB_HOST=seu_db_host
     DB_PORT=5432
     DB_NAME=licencas_obras
     DB_USER=postgres
     DB_PASSWORD=xxxx
     JWT_SECRET=xxxx
     ```
6. Deploy

#### Frontend no Vercel

1. Criar conta em [vercel.com](https://vercel.com)
2. Importar repositório
3. Framework: Vite
4. Build Command: `npm run build`
5. Output Directory: `dist`
6. Environment:
   ```
   VITE_API_BASE_URL=https://seu-backend.onrender.com/api
   ```
7. Deploy

### Opção 2: Servidor Próprio (VPS)

```bash
# SSH no servidor
ssh root@seu.ip.com

# Instalar Node e PostgreSQL
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs postgresql postgresql-contrib

# Clonar projeto
git clone seu-repo.git
cd seu-repo/backend
npm install

# Configurar .env com dados de produção
nano .env

# Iniciar com PM2
npm install -g pm2
pm2 start server.js --name licencas-backend

# Frontend
cd ../frontend
npm run build
# Servir com Nginx/Apache
```

### Opção 3: Docker (Melhor Prática)

Crie `Dockerfile` na raiz do backend:

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --omit=dev
COPY . .
EXPOSE 5000
CMD ["npm", "start"]
```

Build e run:
```bash
docker build -t licencas-backend .
docker run -p 5000:5000 --env-file .env licencas-backend
```

---

## 7️⃣ TROUBLESHOOTING

### ❌ "Conexão ao banco de dados recusada"

**Solução:**
```bash
# Verificar se PostgreSQL está rodando
sudo service postgresql status

# Ou verificar credenciais no .env
psql -U postgres -h localhost
```

### ❌ "CORS error ao chamar API"

**Solução:** Verificar `app.js`:
```javascript
app.use(cors({
  origin: 'http://localhost:3000', // Adicionar seu URL
  credentials: true
}));
```

### ❌ "Token expirado"

**Frontend automático:** Componente `useAuth` tenta renovar automaticamente
**Manual:** Fazer login novamente

### ❌ "Port 5000 já em uso"

```bash
# Encontrar e matar processo
lsof -i :5000
kill -9 <PID>

# Ou mudar porta em .env
PORT=5001
```

### ❌ "Mapa não carrega"

**Solução:** Verificar:
1. Leaflet CSS no `index.html`
2. Ícones do Leaflet em `MapaPage.jsx`
3. Dados GeoJSON sendo retornados pela API

### ❌ "Banco vazio após schema.sql"

```bash
# Verificar se schema rodou
psql -U postgres -d licencas_obras -c "\dt"

# Se vazio, rodar manualmente
psql -U postgres -d licencas_obras < schema.sql
```

---

## 📋 CHECKLIST FINAL

- [ ] PostgreSQL instalado e rodando
- [ ] Banco `licencas_obras` criado
- [ ] Schema SQL executado
- [ ] Backend iniciado na porta 5000
- [ ] Frontend iniciado na porta 5173
- [ ] Login funciona
- [ ] Pode criar empresa
- [ ] Pode emitir licença
- [ ] Mapa carrega
- [ ] Cron jobs iniciados

---

## 🎯 PRÓXIMOS PASSOS

1. **Adicionar dados reais** de Niterói (shapefile, empresas, etc)
2. **Configurar email** para notificações
3. **Integrar com sistema de pagamento** (taxa de licença)
4. **API para denúncias** (email, WhatsApp, formulário)
5. **Dashboard admin** com relatórios
6. **Mobile app** (React Native)

---

## 📞 SUPORTE

- **Logs Backend:** `console do npm run dev`
- **Logs Frontend:** DevTools (F12)
- **Banco de dados:** pgAdmin (recomendado)

---

**Parabéns! Sistema de Licenças está pronto para uso! 🎉**
