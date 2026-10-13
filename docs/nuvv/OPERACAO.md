# Operação e Gestão — DeskcommCRM Nuvv

## 1. Acesso Administrativo Inicial

- **URL de Produção:** [https://crm.nuvv.com.br](https://crm.nuvv.com.br)
- **Painel Administrativo da Plataforma:** [https://crm.nuvv.com.br/admin](https://crm.nuvv.com.br/admin)
- **Painel Operacional do Tenant:** [https://crm.nuvv.com.br/app](https://crm.nuvv.com.br/app)
- **Credenciais do Proprietário:**
  - Usuário: `admin@crm.nuvv.com.br`
  - Senha Inicial: Registrada com segurança em `/home/nuvv/deskcommcrm/.runtime/admin-credentials` no VPS.
  - Recomenda-se ativar MFA TOTP logo no primeiro acesso em **Configurações › Segurança**.

---

## 2. Onboarding e Gestão de Clientes (Tenants)

A Nuvv opera o DeskcommCRM no modelo de plataforma central. Para cadastrar um novo cliente corporativo:

1. Acesse o menu **Administração › Organizações** (`/admin/organizations`).
2. Clique em **Nova Organização**.
3. Preencha:
   - Razão Social / Nome de Exibição da empresa cliente.
   - Slug identificador único (ex: `cliente-acme`).
   - Fuso horário (padrão: `America/Sao_Paulo`).
4. Convide o gestor do cliente com papel `manager` ou `admin`.
5. O cliente receberá o convite por e-mail ou link direto para definir sua senha de acesso.

---

## 3. Atendimento e Operação da Caixa de Entrada (Inbox)

O DeskcommCRM separa rigidamente a **Caixa de Entrada Operacional** do **Funil Comercial (Kanban)**:

- **Inbox (/app/inbox):**
  - **Fila:** Conversas abertas aguardando claim ou roteamento automático.
  - **Minhas:** Conversas atribuídas diretamente ao atendente logado.
  - **Equipe:** Conversas em andamento com outros atendentes (visibilidade para supervisores e administradores).
  - **Finalizadas:** Histórico completo de conversas resolvidas com busca por termos, tags e contatos.
- **Notas Internas:** Mensagens amarelas visíveis apenas para a equipe interna, não enviadas ao cliente final.
- **Transferência:** É possível transferir conversas diretamente para outro atendente ou devolver para a fila geral.
- **Roteamento por Capacidade:** O sistema calcula a disponibilidade e o teto máximo de conversas simultâneas por atendente (`attendant_availability`), distribuindo novos contatos de forma balanceada.

---

## 4. Conexão de Canais WhatsApp

Para conectar o número de um cliente:
1. No menu do tenant, acesse **Canais › Conexões** (`/app/channels`).
2. Clique em **Nova Conexão (WhatsApp)**.
3. Escolha o provider **WAHA**.
4. O sistema solicitará a geração do QR Code através da API do WAHA (`http://waha:3000/api/sessions`).
5. O cliente aponta o aplicativo do WhatsApp do celular e escaneia o código.
6. A sessão migra imediatamente para o status `WORKING`, habilitando o tráfego bidirecional de mensagens e mídias.
