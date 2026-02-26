# Contexto do Projeto — API Conta Simples Developer Portal

> **Para quê serve este arquivo:** Cole o conteúdo deste documento no início de um chat com IA (Cursor, Claude, ChatGPT, etc.) para dar contexto completo do projeto antes de pedir qualquer tarefa.

---

## 1. O que é este projeto

Developer Portal da **API Conta Simples**, construído com **Mintlify**. É um site de documentação pública para parceiros e desenvolvedores que integram com a API da Conta Simples (fintech brasileira de cartões corporativos).

O portal está em **português brasileiro (pt-BR)**.

---

## 2. Stack e plataforma

| Item | Detalhe |
|------|---------|
| Plataforma de docs | [Mintlify](https://mintlify.com) |
| Formato das páginas | **MDX** (Markdown + JSX components do Mintlify) |
| Configuração central | `docs.json` (formato novo do Mintlify — **NÃO usar `mint.json`**) |
| Spec da API | `openapi/public-openapi.json` (OpenAPI 3.0.3) |
| Linguagem dos exemplos | **TypeScript** |
| URLs da API | Sandbox: `https://api-sandbox.contasimples.com` · Produção: `https://api.contasimples.com` |
| Dev local | `npx mintlify@latest dev` → `http://localhost:3000` |
| Deploy | Push para branch principal → Mintlify Cloud faz deploy automático |

**Importante:** Este projeto **NÃO tem `mint.json`**. Toda configuração fica em `docs.json`. Se criar `mint.json`, ele pode conflitar e sobrescrever configurações do `docs.json`.

---

## 3. Estrutura de arquivos

```
docs/
├── docs.json                          # Config central (navegação, tema, OpenAPI, contextual menu)
├── index.mdx                          # Home do portal
├── openapi/
│   └── public-openapi.json            # OpenAPI 3.0.3 — usado pelo Mintlify (NÃO EDITAR o contrato técnico)
│
├── comece-aqui/                       # Tab: Guides → grupo "Comece Aqui"
│   ├── quickstart.mdx                 #   Quickstart (5 min)
│   ├── autenticacao.mdx               #   OAuth 2.0 Client Credentials
│   └── ambientes.mdx                  #   Sandbox vs Produção
│
├── guias/                             # Tab: Guides → grupo "Integração"
│   ├── fluxo-integracao.mdx           #   Passo a passo do Sandbox à Produção
│   └── boas-praticas.mdx              #   Paginação, retry, erros, anexos
│
├── referencia/                        # Tab: Guides → grupo "Referência"
│   └── dicionario-dados.mdx           #   Entidades, campos, enumerações
│
├── api-reference/                     # Tab: API Reference
│   ├── introducao.mdx                 #   Landing page da API Reference
│   ├── autenticacao.mdx               #   POST /oauth/v1/access-token (gerado do OpenAPI)
│   ├── extrato-cartao.mdx             #   POST /statements/v1/credit-card (gerado do OpenAPI)
   │   └── download-anexo.mdx             #   GET /attachments/v1/content/{attachmentId} (gerado do OpenAPI)
│
├── operacao/                          # Páginas operacionais
│   ├── erros-comuns.mdx               #   Diagnóstico de erros por código HTTP
│   ├── suporte.mdx                    #   Como abrir chamados
│   └── changelog.mdx                  #   Histórico de releases
│
├── images/                            # Imagens estáticas
├── logo/                              # Logos light/dark
├── snippets/                          # Snippets reutilizáveis
├── favicon.svg
├── AGENTS.txt                         # Instruções genéricas Mintlify para IA
├── CONTRIBUTING.md
├── README.md
└── LICENSE
```

---

## 4. Arquitetura de informação (regras rígidas)

### Navegação: 3 tabs no topo

```
Guides          │  API Reference         │  Changelog
────────────────┼────────────────────────┼──────────────
Home            │  Visão Geral           │  Changelog
Comece Aqui     │  Autenticação          │
Integração      │  Cartões & Transações  │
Referência      │  Anexos                │
```

### Separação de conteúdo — regra fundamental

| Seção | Contém | NÃO contém |
|-------|--------|------------|
| **Guides** | Contexto, fluxos de negócio, boas práticas, conceitos, dicionário de dados | Endpoints, parâmetros, schemas, métodos HTTP |
| **API Reference** | Endpoints, parâmetros, schemas, playground interativo, códigos HTTP | Fluxos de negócio, explicações conceituais |
| **Changelog** | Histórico de mudanças da API | — |

### Cross-linking

- Guides **apontam para** API Reference quando mencionam um endpoint
- API Reference **aponta para** Guides para contexto e boas práticas
- Nunca duplicar conteúdo entre seções

---

## 5. API — endpoints disponíveis

A API tem **3 endpoints** (definidos no OpenAPI spec):

### POST /oauth/v1/access-token
- Obtém token de acesso (JWT) via OAuth 2.0 Client Credentials
- Header: `Authorization: Basic {base64(api_key:api_secret)}`
- Content-Type: `application/x-www-form-urlencoded`
- Body: `grant_type=client_credentials`
- Retorna: `{ access_token, token_type: "Bearer", expires_in: 3600 }`
- Token válido por 1 hora
- Credenciais (API Key + API Secret) gerenciadas via Internet Banking: https://ib.contasimples.com/integracoes/api/credenciais
- Acesso às credenciais depende do perfil de usuário/permissões no IB
- Não requer autenticação (é o endpoint que gera o token)

### POST /statements/v1/credit-card
- Consulta extrato de transações de cartão por período
- Body: `{ startDate, endDate, limit, nextPageStartKey? }`
- Período máximo: 62 dias
- Limit: 5–100
- Retorna: `{ transactions: CardTransaction[], nextPageStartKey? }`
- Paginação baseada em cursor (`nextPageStartKey`)
- Header: `Authorization: Bearer {TOKEN}`

### GET /attachments/v1/content/{attachmentId}
- Download binário de um anexo (comprovante, nota fiscal)
- Retorna: arquivo binário (PNG, JPEG ou PDF)
- O `attachmentId` vem do array `attachments` dentro de cada transação
- Header: `Authorization: Bearer {TOKEN}`

### Entidades principais
- **CardTransaction** — transação com campos: id, operation (CASH_IN/CASH_OUT), transactionDate, status (PENDING/PROCESSED/CANCELED), type (22 valores possíveis), merchant, amountBrl, card, category, costCenter, attachments
- **Card** — id, maskedNumber, responsibleName, responsibleEmail, type (VIRTUAL/PHYSICAL)
- **Category** — id, name
- **CostCenter** — id, name
- **Attachment** — id, name, _links (HATEOAS)

---

## 6. docs.json — configuração essencial

### Contextual menu (menu "Copy page" em cada página)
```json
"contextual": {
  "options": ["copy", "view", "chatgpt", "claude", "perplexity", "mcp", "cursor", "vscode"]
}
```
**NUNCA remover ou reduzir estas opções.** O MCP é especialmente importante para este portal.

### OpenAPI
```json
"openapi": ["/openapi/public-openapi.json"]
```

### Páginas de endpoint (MDX que referenciam o OpenAPI spec)
```yaml
# extrato-cartao.mdx
---
title: "Extrato de Cartão"
openapi: "POST /statements/v1/credit-card"
---
```
O Mintlify gera automaticamente o playground e documentação técnica a partir do spec OpenAPI 3.0.3. O spec precisa ser OpenAPI 3.x para o Mintlify renderizar corretamente (Swagger 2.0 não renderiza).

---

## 7. Convenções e padrões

### Escrita
- Português brasileiro (pt-BR)
- Voz ativa, segunda pessoa ("você")
- Uma ideia por frase
- Sentence case nos headings
- **Negrito** para elementos de UI
- `Code` para nomes de arquivo, comandos, campos, endpoints

### Componentes Mintlify usados
- `<Steps>`, `<Step>` — jornadas passo a passo
- `<CardGroup>`, `<Card>` — links de navegação visual
- `<AccordionGroup>`, `<Accordion>` — conteúdo colapsável
- `<Tabs>`, `<Tab>` — conteúdo em abas
- `<Info>`, `<Warning>`, `<Tip>`, `<Note>` — callouts
- Blocos mermaid — diagramas (sem `<Frame>`, renderizados diretamente)
- Tabelas markdown padrão

### Links internos
- Sempre usar caminhos relativos à raiz: `/comece-aqui/quickstart`, `/api-reference/extrato-cartao`
- Âncoras com `#`: `/guias/boas-praticas#paginação`

---

## 8. Restrições — o que NÃO fazer

1. **NÃO criar `mint.json`** — usar apenas `docs.json`
2. **NÃO editar o contrato técnico do `openapi/public-openapi.json`** — apenas metadados (tags, summary, description) podem ser ajustados; endpoints, parâmetros e schemas devem refletir a API real
3. **NÃO inventar endpoints, parâmetros ou schemas** que não existam no spec
4. **NÃO duplicar conteúdo** entre Guides e API Reference
5. **NÃO criar páginas de guia por recurso** (ex: guias/cartoes.mdx, guias/transacoes.mdx) — isso já foi tentado e removido por causar duplicação com API Reference
6. **NÃO remover opções do `contextual.options`** no docs.json — especialmente o MCP
7. **NÃO usar inglês** no conteúdo das páginas (o conteúdo é pt-BR, mas os nomes das tabs "Guides", "API Reference", "Changelog" ficam em inglês)

---

## 9. TODOs conhecidos no projeto

Existem alguns placeholders `TODO` nas páginas que precisam ser preenchidos quando informações reais estiverem disponíveis:

- `ambientes.mdx` — Dados de teste disponíveis no Sandbox
- `suporte.mdx` — Canais de suporte específicos (e-mail, telefone)
- `suporte.mdx` — SLAs de resposta
- `changelog.mdx` — Versão 1.0.0 quando API for para produção

---

## 10. Como rodar localmente

```bash
cd /Users/weiberlangarcia/Code/docs
npx mintlify@latest dev        # rodar em http://localhost:3000
npx mintlify@latest broken-links  # verificar links quebrados
```

---

## 11. Jornada do desenvolvedor (fluxo projetado)

```
Home → Quickstart (5 min)
  → Credenciais via IB (ib.contasimples.com/integracoes/api/credenciais)
  → Autenticação (OAuth 2.0 com API Key + API Secret)
  → Ambientes (Sandbox → Produção)
  → Fluxo de Integração (passo a passo completo)
  → Boas Práticas (paginação, retry, erros)
  → API Reference (endpoints, schemas, playground)
  → Dicionário de Dados (entidades e enumerações)
  → Go-live (checklist de produção)
```
