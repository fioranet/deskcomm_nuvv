# Segurança, Autenticação e LGPD — DeskcommCRM Nuvv

## 1. Princípios de Segurança Implementados

O DeskcommCRM Nuvv foi projetado e implantado com segurança em profundidade:

1. **Superfície de Exposição Mínima:**
   - Apenas o Caddy expõe as portas 80 e 443 à Internet.
   - Postgres, Redis, SRH, GoTrue, Storage e WAHA operam em redes internas do Docker.
   - O firewall UFW bloqueia qualquer pacote direto às portas de infraestrutura.
2. **Criptografia em Trânsito:**
   - TLS 1.2 e TLS 1.3 obrigatórios com certificados automáticos Let's Encrypt.
   - HSTS e cabeçalhos modernos configurados (`X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`).
3. **Criptografia em Repouso e Segredos:**
   - Tokens de canais e webhooks criptografados no banco (`bytea` criptografado via chaves do Vault).
   - Senhas de banco e chaves de API com entropia de 256 bits.
   - Acesso ao WAHA protegido com hash `sha512` no header `X-Api-Key`.

---

## 2. Autenticação e RBAC (Controle Baseado em Papéis)

O sistema utiliza Supabase Auth com tokens JWT assinados via `HS256`/`RS256` e cookies `SameSite=Strict`.

### Hierarquia de Papéis (RBAC):
- **Platform Admin:** Administrador global da infraestrutura Nuvv (`/admin`).
- **Admin:** Proprietário/Diretor da empresa cliente. Acesso total às configurações, canais, financeiro e usuários do seu tenant.
- **Manager:** Supervisor de atendimento. Gerencia filas, kanbans, atendentes, campanhas e métricas.
- **Agent:** Atendente operacional. Acesso restrito à Inbox, contatos e conversas atribuídas.
- **Viewer:** Visualizador somente leitura para relatórios e auditoria.

### Política de MFA (Autenticação em Dois Fatores):
- Suporte nativo a TOTP (Google Authenticator, Microsoft Authenticator, 1Password).
- Obrigatoriedade configurável em duas esferas: nível de plataforma e nível de organização cliente.

---

## 3. Conformidade com a LGPD

1. **Isolamento de Dados por Tenant:**
   - RLS ativo em 100% das tabelas que contêm dados de clientes.
   - Um cliente jamais enxerga ou acessa registros de outro cliente sob nenhuma circunstância.
2. **Trilha de Auditoria Universal (`audit_log`):**
   - Todas as mutações e acessos a dados sensíveis registram ator, IP, `organization_id`, timestamp e payload anterior/posterior.
3. **Direito ao Esquecimento e Anonimização:**
   - O sistema dispõe de rotinas automatizadas para expurgo e anonimização de dados de contato e mídias mediante solicitação do titular.
