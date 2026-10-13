# Estrutura de IA, Orçamento e Modelo Comercial Futuro — DeskcommCRM Nuvv

## 1. Arquitetura do Runtime de IA

O DeskcommCRM integra inteligência artificial nativa nos atendimentos через Vercel AI SDK e roteamento dinâmico de provedores.

### Provedores e Modelos Suportados:
- **Anthropic (Claude):** Primário para agentes SDR e atendimento conversacional complexo.
- **OpenAI (GPT-4o, o1, Text-Embedding-3):** Raciocínio estruturado e embeddings para RAG.
- **Google Gemini (Gemini 1.5 Pro / Flash):** Raciocínio multimodal e velocidade de resposta.
- **Modelos Locais / OpenRouter:** Flexibilidade de roteamento por organização.

### Estrutura de Credenciais:
1. **Credenciais da Plataforma Nuvv:** Configuradas a nível de servidor para clientes que contratam IA como serviço gerenciado.
2. **Credenciais Próprias da Organização (BYOK):** A empresa cliente pode inserir sua própria API Key nas configurações do tenant (`ai_credentials`), isolando seu faturamento direto no provedor.

---

## 2. Controle de Orçamento e Medição de Consumo

O DeskcommCRM implementa controle de teto e orçamento nativo por organização através da tabela `ai_budgets`:

- **Métricas Monitoradas:**
  - `current_month_consumed_cents`: Valor acumulado consumido no mês em centavos de dólar, calculado pela função agregadora `fn_gasto_de_ia_do_mes`.
  - `monthly_limit_cents`: Teto limite mensal estipulado para a organização.
  - `alarm_threshold_pct`: Percentual de aviso prévio (ex: 80% do limite).
- **Modos de Aplicação (`enforcement_mode`):**
  - `off`: Orçamento desativado (uso livre).
  - `warn`: Emite avisos visuais no painel sem interromper o serviço.
  - `block`: Ao atingir o teto, a IA para de responder novas mensagens.

---

## 3. Garantia de Continuidade do Atendimento Humano

**Invariante Crucial:** O bloqueio por falta de saldo de IA **NUNCA** suspende ou paralisa o atendimento humano.
Quando o orçamento de IA estoura:
1. O runtime de IA emite `budget_exceeded` e devolve a conversa para a fila de atendimento humano.
2. O cliente é prontamente atendido pelos atendentes humanos da empresa.
3. Notificações de alerta são geradas para o gestor na Inbox e no menu Radar.

---

## 4. Integração Comercial Futura: Créditos Pré-Pagos via SGP

Para viabilizar a venda de créditos de IA com recarga pelo sistema financeiro SGP da Nuvv:
1. **Ponto de Integração Técnico:**
   - O SGP pode interagir via API externa (autenticada por token de serviço Bearer) ou webhook, atualizando a cota `monthly_limit_cents` ou uma tabela dedicada de `org_credits` vinculada ao `organization_id`.
2. **Modelo de Recarga:**
   - O cliente adquire R$ 100,00 de créditos no SGP.
   - O webhook do SGP credita o valor na organização correspondente no DeskcommCRM e reativa automaticamente o `enforcement_mode` caso estivesse bloqueado por limite.
3. **Sem Impacto Operacional:**
   - Nesta fase de implantação, nenhuma chave de IA paga foi vinculada compulsoriamente, preservando a segurança financeira da operação.
