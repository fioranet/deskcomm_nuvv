# Política de Backup e Recuperação de Desastres — DeskcommCRM Nuvv

## 1. Escopo do Backup

O procedimento oficial de backup do DeskcommCRM Nuvv captura três dimensões essenciais para garantir a recuperação integral da plataforma:

1. **Dump do Banco de Dados Relacional (`db-*.sql.gz`):**
   - Dump completo do PostgreSQL 17 contendo todos os dados, schemas (`public`, `auth`, `storage`), RLS, funções e triggers.
   - Gerado via container `postgres:17-alpine` e compactado com integridade CRC checada (`gzip -t`).
2. **Snapshot de Sessões do WhatsApp (`waha-*.tgz`):**
   - Compactação dos dados do volume do WAHA contendo credenciais de pareamento do WhatsApp.
3. **Mídias e Anexos de Clientes (`storage-*.tgz`):**
   - Snapshot do diretório do Supabase Storage contendo fotos, áudios e documentos trocados nos atendimentos.

---

## 2. Automação Diária e Retenção

O backup automático foi configurado no cron do sistema operacional:
- **Arquivo de Cron:** `/etc/cron.d/deskcomm-backup`
- **Frequência:** Diariamente às **03:00 da madrugada** (horário do servidor).
- **Destino Local:** `/home/nuvv/deskcommcrm/backups/` com permissões estritas `0700` (`chmod 700`).
- **Política de Retenção:** O script mantém automaticamente os **14 backups mais recentes** de cada tipo e remove os mais antigos.

---

## 3. Evidência de Teste de Restauração em Ambiente Isolado

Para validar a confiabilidade do backup, foi executado um teste real de restauração:
1. Um backup fresco foi gerado: `db-20261013-112224.sql.gz` (340 KB compactado).
2. Um banco de dados isolado `deskcomm_restore_test` foi criado temporariamente no Postgres.
3. O dump foi descompactado e aplicado via `psql`.
4. **Resultado da Auditoria:** **186 tabelas** foram restauradas e validadas no schema `public` com 100% de sucesso.
5. O banco de teste foi removido após a prova.

---

## 4. Procedimento de Restauração em Caso de Desastre (Disaster Recovery)

Em caso de falha catastrófica de hardware ou migração de VPS:

1. Subir novo VPS com o ambiente preparado (`install-single-server.sh`).
2. Copiar os arquivos de backup para o diretório `/home/nuvv/deskcommcrm/backups/`.
3. Executar o script de restauração:
   ```bash
   cd /home/nuvv/deskcommcrm
   bash hostgator-setup-kit/restore.sh backups/db-AAAAMMDD-HHMMSS.sql.gz
   ```
4. Restaurar os anexos do storage e as sessões do WhatsApp:
   ```bash
   tar -xzf backups/storage-AAAAMMDD-HHMMSS.tgz -C .runtime/supabase/volumes/storage/
   docker run --rm -v deskcommcrm_waha_data:/data -v /home/nuvv/deskcommcrm/backups:/backup alpine:3.20 tar -xzf /backup/waha-AAAAMMDD-HHMMSS.tgz -C /data
   ```
5. Reiniciar os serviços Docker:
   ```bash
   docker compose -f docker-compose.prod.yml restart
   ```
