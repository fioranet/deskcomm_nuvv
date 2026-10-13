# Ciclo de Vida e Atualização da Plataforma — DeskcommCRM Nuvv

## 1. Estratégia de Versionamento Git

O ambiente da Nuvv adota a arquitetura de **Fork Controlado**:

- **Repositório Upstream (Original):** `https://github.com/melgarafael/DeskcommCRM`
- **Repositório Origin (Nuvv Oficial):** `https://github.com/fioranet/deskcomm_nuvv`
- **Commit Base Fixado na Implantação:** `ff92e0f98993f4e1f72e737c35e821ba6b618841` (Tag `v1.78.0-177` / correspondente à release `1.79.0`).

Todas as personalizações da Nuvv (documentação, scripts e configurações) residem na branch `main` do repositório da Nuvv sem descaracterizar a árvore upstream.

---

## 2. Procedimento Seguro de Atualização

Nunca execute atualizações cegas em produção sem seguir o procedimento homologado:

### Passo 1: Verificar e Sincronizar o Upstream no Repositório Local
```bash
git fetch upstream
git log --oneline HEAD..upstream/main
```

### Passo 2: Testar e Validar Migrations
Se houver novas migrações em `supabase/migrations/`:
1. Inspecionar o impacto no `supabase/baseline.sql`.
2. Validar que as migrações mantêm a regra de compatibilidade retroativa e idempotência.

### Passo 3: Executar Atualização no VPS via Script Oficial
O script `update.sh` do kit self-host executa backup preventivo antes de qualquer alteração:
```bash
cd /home/nuvv/deskcommcrm
bash hostgator-setup-kit/update.sh
```
O script executa automaticamente:
1. Geração de backup do banco e das sessões do WhatsApp.
2. `git pull` controlado.
3. Aplicação incremental do `supabase/baseline.sql`.
4. `docker compose pull` das novas imagens estáveis.
5. `docker compose up -d` com zero interrupção desnecessária.
6. Execução do `healthcheck.sh` interno.

---

## 3. Procedimento de Rollback

Se uma atualização apresentar comportamento anômalo:
1. Interromper os contêineres:
   ```bash
   docker compose -f docker-compose.prod.yml down
   ```
2. Reverter o código para o commit anterior:
   ```bash
   git reset --hard COMMIT_ANTERIOR
   ```
3. Se o banco sofreu alterações estruturais incompatíveis, restaurar o backup gerado imediatamente antes do update:
   ```bash
   bash hostgator-setup-kit/restore.sh backups/db-PRE-UPDATE.sql.gz
   ```
4. Subir a stack com as imagens da versão anterior:
   ```bash
   docker compose -f docker-compose.prod.yml up -d
   ```
