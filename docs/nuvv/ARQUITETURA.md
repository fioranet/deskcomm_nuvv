# Arquitetura — DeskcommCRM Nuvv Multiatendimento

## 1. Visão Geral da Solução

O **DeskcommCRM Nuvv** é uma plataforma multitenant de multiatendimento corporativo e CRM integrada a canais de mensageria (WhatsApp via WAHA e Meta Cloud API), banco de dados relacional isolado via Row Level Security (RLS), autenticação JWT e runtime de inteligência artificial.

A infraestrutura foi implantada seguindo o modelo canônico **Single-Server Self-Hosted** do DeskcommCRM, garantindo controle total sobre os dados, conformidade nativa com a LGPD e soberania de infraestrutura.

---

## 2. Topologia do Servidor de Produção

- **Endereço IP:** `200.6.48.43`
- **Domínio (FQDN):** `https://crm.nuvv.com.br`
- **Sistema Operacional:** Debian GNU/Linux 12 (bookworm) x86_64
- **Recursos Alocados:** 10 vCPUs, 8.0 GiB RAM, 4.0 GiB Swap, 98 GB SSD
- **Rede e Gateway:** Configurado com gateway persistente `200.6.48.1` na interface `ens18`
- **Firewall:** UFW ativo em modo restritivo (portas permitidas: 22/SSH, 80/HTTP, 443/HTTPS)

---

## 3. Arquitetura de Containers Docker

A pilha de produção opera com isolamento em duas redes Docker bridge (`internal` e `traefik_proxy` / `caddy`), composta por 18 contêineres:

```
                                  [ INTERNET ]
                                        │
                         (Portas 80/443 TCP - HTTPS TLS)
                                        ▼
                      ┌───────────────────────────────────┐
                      │    Caddy Reverse Proxy (HTTPS)    │
                      └─────────────────┬─────────────────┘
                                        │ (Proxy Interno)
         ┌──────────────────────────────┴──────────────────────────────┐
         ▼                                                             ▼
┌──────────────────┐                                         ┌───────────────────┐
│ Next.js 16 (App) │                                         │   Supabase Auth   │
│ Port 3000        │                                         │   (GoTrue /auth)  │
└────────┬─────────┘                                         └─────────┬─────────┘
         │                                                             │
         ├──────────────────┬──────────────────┐                       │
         ▼                  ▼                  ▼                       │
┌──────────────────┐ ┌──────────────┐ ┌──────────────────┐             │
│  Deskcomm Worker │ │ Deskcomm Sched│ │    WAHA NOWEB    │             │
│  (Engine SDR)    │ │ (Cron jobs)  │ │ (Port 3000 int)  │             │
└────────┬─────────┘ └──────┬───────┘ └────────┬─────────┘             │
         │                  │                  │                       │
         └──────────────────┼──────────────────┘                       │
                            ▼                                          │
       ┌────────────────────────────────────────────────────────┐      │
       │                   PostgreSQL 17 DB                     │◄─────┘
       │          (Supabase Baseline + RLS Multitenant)         │
       └────────────────────────────┬───────────────────────────┘
                                    │
           ┌────────────────────────┴────────────────────────┐
           ▼                                                 ▼
┌─────────────────────┐                           ┌─────────────────────┐
│  Redis + SRH Cache  │                           │  Supabase Storage   │
└─────────────────────┘                           └─────────────────────┘
```

### Componentes Principais:
1. **Frontend & API Gateway (Next.js 16 App Router):**
   - UI de atendimento, Inbox unificada, Kanban comercial, supervisão e administração.
   - Rotas REST sob `/api/v1/` com validação Zod e auditoria assíncrona.
   - Imagem: `ghcr.io/melgarafael/deskcommcrm:stable` (v1.79.0).
2. **Workers em Segundo Plano:**
   - `deskcomm-worker`: Drena eventos da tabela `event_log` e orquestra execuções de automação e IA.
   - `deskcomm-scheduler`: Dispara crons periódicos de rotação, rotinas de agenda e auditorias.
3. **Canal WhatsApp (WAHA Core NOWEB):**
   - Engine Chromium-less de alta performance e baixo consumo de memória.
   - Autenticação via `X-Api-Key` com hash SHA-512.
   - Webhook interno apontando diretamente para `http://app:3000/api/v1/webhooks/waha`.
4. **Camada de Dados (Supabase Self-Hosted v0.8.1):**
   - PostgreSQL 17 com extensões `pgvector`, `uuid-ossp`, `pg_stat_statements`.
   - Autenticação JWT via GoTrue / Supabase Auth com suporte a MFA TOTP.
   - Storage com backend em sistema de arquivos local (`STORAGE_BACKEND=file`).
   - PostgREST e Realtime para assinaturas em tempo real via WebSocket.
5. **Caddy Web Server:**
   - Emissão e renovação automática de certificados SSL/TLS Let's Encrypt para `crm.nuvv.com.br`.
   - Compressão zstd/gzip e headers modernos de segurança HTTP.
