# Instalação e Provisionamento — DeskcommCRM Nuvv

## 1. Pré-Requisitos e Ambiente

- **Servidor:** VPS dedicado Debian 12 (bookworm).
- **Pacotes Base:** `docker.io`, `docker-compose-plugin`, `ufw`, `curl`, `jq`, `openssl`, `git`, `systemd-timesyncd`.
- **Portas Externas:** Exclusivamente `22` (SSH), `80` (HTTP) e `443` (HTTPS). Todas as demais portas de bancos, Redis, WAHA e Supabase são estritamente internas.

---

## 2. Passo a Passo de Execução

### 2.1 Preparação de Rede e Segurança no Servidor
```bash
# Garantir gateway padrão correto no /etc/network/interfaces
# gateway 200.6.48.1

# Instalação do Docker e utilitários
apt update && apt install -y docker-ce docker-compose-plugin ufw git curl jq openssl

# Configuração de Firewall restritivo
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp
ufw allow 80/tcp
ufw allow 443/tcp
ufw --force enable
```

### 2.2 Clonagem e Inicialização do Repositório
```bash
git clone https://github.com/fioranet/deskcomm_nuvv.git /home/nuvv/deskcommcrm
cd /home/nuvv/deskcommcrm
```

### 2.3 Execução do Instalador Single-Server Oficial
```bash
bash hostgator-setup-kit/install-single-server.sh --domain crm.nuvv.com.br
```
O script executa de forma determinística:
1. Geração de segredos de alta entropia (`JWT_SECRET`, `ANON_KEY`, `SERVICE_ROLE_KEY`, `WAHA_API_KEY_SHA512`, senhas de banco).
2. Provisionamento da stack local Supabase `v0.8.1` em `.runtime/supabase`.
3. Aplicação do schema completo via `supabase/baseline.sql` através de container isolado `postgres:17-alpine`.
4. Inicialização de Caddy, Next.js, Workers, Scheduler, Redis e WAHA.
5. Emissão do certificado TLS Let's Encrypt para `crm.nuvv.com.br`.
6. Criação do usuário administrador da plataforma (`admin@crm.nuvv.com.br`) e geração de credenciais em `.runtime/admin-credentials`.

---

## 3. Verificação de Saúde Pós-Instalação

Para conferir o status de saúde da pilha após a inicialização:
```bash
# Validar containers
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Testar health endpoint interno da aplicação
curl -s https://crm.nuvv.com.br/api/v1/health | jq .
```
Resposta esperada:
```json
{
  "status": "ok",
  "checks": {
    "supabase": "ok",
    "redis": "ok",
    "waha": "ok"
  }
}
```
