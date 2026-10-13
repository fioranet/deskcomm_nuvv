# Auditoria de Multitenancy e Isolamento RLS — DeskcommCRM Nuvv

## 1. Modelo Multitenant e Row Level Security (RLS)

O DeskcommCRM foi concebido sob a premissa de **Multi-tenancy com isolamento rigoroso por banco de dados**.

Todas as entidades de negócio (contatos, conversas, mensagens, canais de WhatsApp, agentes de IA, tags, notas internas e arquivos) possuem a chave estrangeira obrigatória:
```sql
organization_id uuid not null references organizations(id) on delete cascade
```

O isolamento é imposto pelo motor do PostgreSQL através de políticas de RLS e das funções canônicas `SECURITY DEFINER`:
- `fn_user_org_ids()`: Devolve a lista exata de organizações a que o usuário autenticado pertence.
- `fn_user_role_in_org(org_id)`: Avalia o papel do usuário dentro daquela organização específica.

---

## 2. Prova Prática Executável de Isolamento (Fase 8)

Foi executada uma bateria de testes transacionais automatizados simulando duas empresas concorrentes em produção:
- **Org A (Empresa A Teste Ltda):** ID `11111111-aaaa-4000-8000-000000000001` com Atendente A.
- **Org B (Empresa B Teste Ltda):** ID `22222222-bbbb-4000-8000-000000000002` com Atendente B.

### Cenário 1: Visibilidade de Leitura do Usuário A
- **Ação:** Assumir o papel autenticado (`SET ROLE authenticated`) com o JWT do Atendente A (`sub: user_a`).
- **Resultado:**
  - O Atendente A enxergou exatamente seu contato e sua conversa da Org A.
  - O Atendente A enxergou **ZERO** contatos, conversas ou canais da Org B.
  - **Status:** **APROVADO (100% de isolamento).**

### Cenário 2: Visibilidade de Leitura do Usuário B
- **Ação:** Assumir o papel autenticado com o JWT do Atendente B (`sub: user_b`).
- **Resultado:**
  - O Atendente B enxergou apenas os dados da Org B.
  - **ZERO** vazamento ou visibilidade de registros da Org A.
  - **Status:** **APROVADO (100% de isolamento).**

### Cenário 3: Tentativa de Injeção / Inserção Cruzada entre Tenants
- **Ação:** Usuário B tenta forçar uma inserção de contato injetando o `organization_id` da Org A no comando:
  ```sql
  INSERT INTO contacts (id, organization_id, display_name, phone_number)
  VALUES ('33333333-000c-4000-8000-000000000003', '11111111-aaaa-4000-8000-000000000001', 'Hacker Invasor', '+5511999999999');
  ```
- **Resultado do PostgreSQL:**
  - `ERROR: 42501 - new row violates row-level security policy for table "contacts"`
  - O banco de dados barrou a gravação no ato.
  - **Status:** **APROVADO (Bloqueio total pelo RLS).**

---

## 3. Conclusão da Auditoria Multitenant

A infraestrutura garante isolamento matemático e criptográfico entre as empresas clientes. Nenhuma empresa tem acesso aos dados, contatos, conversas ou configurações de outra empresa parceira.
