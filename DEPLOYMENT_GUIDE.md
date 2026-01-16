# 🚀 Guia de Deploy - Sistema de Gestão de Compras Ziran v2.0

## 📋 Arquitetura

| Componente | Tecnologia | Porta |
|------------|-----------|-------|
| **Backend** | FastAPI + SQLAlchemy | 8000 |
| **Frontend** | Vue 3 + Vite + TailwindCSS | 80 |
| **Database** | PostgreSQL 15 | 5432 |
| **Proxy** | Nginx (no container frontend) | 80 |

---

## 🔧 Deploy Rápido (Docker)

### **1. Criar arquivo `.env`**

```bash
# Na raiz do projeto, copie o exemplo:
cp .env.production.example .env

# Edite o arquivo .env:
nano .env   # ou use seu editor preferido
```

### **2. Configurar variáveis obrigatórias**

```env
# OBRIGATÓRIO - Gere com: python -c "import secrets; print(secrets.token_hex(32))"
SECRET_KEY=sua_chave_segura_aqui_64_caracteres_hex

# OBRIGATÓRIO - Senha do banco de dados
POSTGRES_PASSWORD=sua_senha_forte_do_banco

# Opcional - ajuste conforme necessário
CORS_ORIGINS=http://localhost,http://seu-dominio.com
```

### **3. Iniciar os containers**

```bash
# Build e start
docker-compose up -d --build

# Verificar status
docker-compose ps

# Ver logs
docker-compose logs -f backend
```

### **4. Acessar o sistema**

- **Frontend:** http://localhost
- **API Docs:** http://localhost:8000/api/docs
- **Health Check:** http://localhost:8000/health

### **5. Login inicial**

```
Usuário: admin
Senha: admin123 (ALTERE IMEDIATAMENTE)
```

---

## 🔐 Segurança em Produção

### **Checklist Obrigatório**

- [ ] SECRET_KEY gerada com `secrets.token_hex(32)`
- [ ] POSTGRES_PASSWORD forte (mín. 16 caracteres)
- [ ] Arquivo `.env` NÃO commitado no Git
- [ ] DEBUG=false
- [ ] ENVIRONMENT=production
- [ ] CORS_ORIGINS apenas domínios permitidos
- [ ] Senha do admin alterada no primeiro acesso

### **Gerar SECRET_KEY Segura**

```bash
# Windows PowerShell
python -c "import secrets; print(secrets.token_hex(32))"

# Linux/Mac
openssl rand -hex 32
```

### **Exemplo de Senha Forte**

```bash
# Gerar senha aleatória para banco
python -c "import secrets; print(secrets.token_urlsafe(24))"
```

---

## 📁 Estrutura de Arquivos

```
Sistema-de-compras/
├── .env                    # ⚠️ Variáveis sensíveis (NÃO COMMITAR)
├── .env.production.example # Template de configuração
├── docker-compose.yml      # Orquestração dos containers
├── backend/
│   ├── Dockerfile
│   ├── app/
│   ├── alembic/
│   └── requirements.txt
└── frontend/
    ├── Dockerfile
    ├── nginx.conf
    └── src/
```

---

## 🛠️ Comandos Úteis

### **Docker**

```bash
# Parar todos os containers
docker-compose down

# Parar e remover volumes (APAGA DADOS!)
docker-compose down -v

# Rebuild apenas backend
docker-compose up -d --build backend

# Entrar no container
docker exec -it compras_backend bash

# Ver logs em tempo real
docker-compose logs -f
```

### **Banco de Dados**

```bash
# Backup do banco
docker exec compras_db pg_dump -U compras_user compras_ziran > backup.sql

# Restaurar backup
docker exec -i compras_db psql -U compras_user compras_ziran < backup.sql

# Acessar psql
docker exec -it compras_db psql -U compras_user -d compras_ziran
```

### **Migrations (Alembic)**

```bash
# Dentro do container backend
docker exec -it compras_backend bash
alembic upgrade head
alembic revision --autogenerate -m "descricao"
```

---

## 🌐 Deploy com HTTPS (Produção Real)

### **Opção 1: Reverse Proxy (Nginx/Traefik)**

```nginx
# /etc/nginx/sites-available/compras
server {
    listen 443 ssl;
    server_name compras.seudominio.com;
    
    ssl_certificate /etc/letsencrypt/live/compras.seudominio.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/compras.seudominio.com/privkey.pem;
    
    location / {
        proxy_pass http://localhost:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    location /api {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### **Opção 2: Cloudflare Tunnel**

```bash
cloudflared tunnel --url http://localhost:80
```

---

## 🚨 Troubleshooting

### **Container não inicia**

```bash
# Verificar logs
docker-compose logs backend

# Erros comuns:
# - "SECRET_KEY não definida" → Criar arquivo .env
# - "POSTGRES_PASSWORD não definida" → Adicionar senha no .env
# - "Connection refused" → Aguardar DB healthcheck
```

### **Erro de CORS**

```bash
# Verificar CORS_ORIGINS no .env
CORS_ORIGINS=http://localhost,http://seu-ip:80
```

### **Migrations não aplicadas**

```bash
docker exec -it compras_backend alembic upgrade head
```

### **Reset completo (desenvolvimento)**

```bash
docker-compose down -v
docker-compose up -d --build
```

---

## 📊 Monitoramento

### **Health Checks**

```bash
# Backend
curl http://localhost:8000/health

# Database (via backend)
curl http://localhost:8000/api/v1/dashboard/resumo
```

### **Logs Estruturados**

Os logs do backend incluem:
- Timestamp
- Método HTTP
- Path
- Status code
- Duração (ms)
- IP do cliente

---

## ✅ Checklist GO-LIVE

### **Pré-Deploy**

- [ ] `.env` criado com todas variáveis
- [ ] SECRET_KEY segura (64 chars hex)
- [ ] POSTGRES_PASSWORD forte
- [ ] Migrations aplicadas
- [ ] Testes funcionais OK

### **Pós-Deploy**

- [ ] Alterar senha do admin
- [ ] Verificar health check
- [ ] Testar login/logout
- [ ] Testar criação de solicitação
- [ ] Testar fluxo de aprovação
- [ ] Configurar backup automático

---

## 📞 Suporte

Em caso de problemas:
1. Verificar logs: `docker-compose logs -f`
2. Verificar `.env` está configurado
3. Verificar containers: `docker-compose ps`
4. Reiniciar: `docker-compose restart`
