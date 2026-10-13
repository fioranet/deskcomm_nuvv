# Pendências e Recomendações de Evolução — DeskcommCRM Nuvv

## 1. Pendências Operacionais e Itens Dependentes de Autorização/Credenciais

1. **Ativação de Chaves Reais de Inteligência Artificial:**
   - **Status:** Estrutura arquitetural e guards de teto homologados.
   - **Dependência:** Fornecimento das credenciais reais de produção (OpenAI, Anthropic ou Google) pela equipe Nuvv ou inserção direta pelas empresas clientes no painel do tenant.
2. **Conexão dos Primeiros Números Oficiais de WhatsApp:**
   - **Status:** Container WAHA NOWEB ativo e saudável.
   - **Dependência:** Escaneamento de QR Code com os aparelhos celulares dos primeiros clientes/departamentos.
3. **Configuração de Servidor SMTP / Resend para Convites por E-mail:**
   - **Status:** Supabase Auth funcional em modo direto.
   - **Dependência:** Configuração de credenciais SMTP ou API Key do Resend no `.env` para envio automatizado de e-mails transacionais de convite e redefinição de senha.
4. **Troca de Senha Administrativa Inicial:**
   - **Ação Recomendada:** O administrador geral da Nuvv deve efetuar login em `https://crm.nuvv.com.br` com as credenciais iniciais registradas em `.runtime/admin-credentials`, alterar sua senha e habilitar MFA via aplicativo autenticador.

---

## 2. Recomendações para Evolução Comercial e Produto

1. **Divisão por Departamentos (Comercial / Suporte / Financeiro):**
   - **Diagnóstico Atual:** O DeskcommCRM nativo gerencia atendentes e filas de roteamento por **Canal** (`channel_routing_policies`) e **Tags**, não possuindo uma entidade `departments` isolada no banco de dados.
   - **Recomendação Nuvv:** Utilizar canais dedicados por departamento (ex: número WhatsApp Suporte vs número WhatsApp Comercial) ou filas automatizadas por tags, associando políticas de roteamento distintas a cada grupo de atendentes.
2. **Integração de Recarga Financeira Pré-Paga com SGP:**
   - Desenvolver endpoint seguro ou webhook para consumir eventos de liquidação financeira do SGP e injetar créditos automaticamente no saldo do cliente.
3. **Backup Externo Offsite:**
   - Configurar o envio automático dos arquivos compactados em `/home/nuvv/deskcommcrm/backups/` para um bucket S3/Wasabi externo via ferramenta `rclone` ou script auxiliar diário.
