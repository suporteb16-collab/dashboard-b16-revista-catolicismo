# Dashboard — Revista Catolicismo (B16)

Dashboard de performance Meta Ads × Kiwify com dados em tempo real.

**Stack:** Google Sheets → Cloudflare Worker → GitHub Pages

---

## Arquivos do repositório

```
dashboard-b16-revista-catolicismo/
├── index.html   ← dashboard (subir no GitHub)
├── worker.js    ← código do Worker (colar no Cloudflare — NÃO subir no GitHub)
└── README.md
```

---

## ⚠ Problema: Worker com erro DNS (NXDOMAIN)

O Worker existe mas nunca foi deployado com código. Siga estes passos:

### Passo 1 — Abrir o Worker no Cloudflare

1. Acesse [dash.cloudflare.com](https://dash.cloudflare.com)
2. Menu esquerdo → **Workers & Pages**
3. Clique no worker `dashboard-api`
4. Clique em **Edit Code** (botão azul no topo direito)

### Passo 2 — Colar o código

1. Selecione todo o código existente (Ctrl+A) e delete
2. Cole o conteúdo completo do arquivo `worker.js` deste repositório
3. Clique em **Deploy** (botão azul)
4. Aguarde a mensagem de sucesso

### Passo 3 — Configurar as Secrets

Ainda no painel do Worker → aba **Settings** → **Variables**:

Clique em **Add variable** para cada uma abaixo, marcando **Encrypt**:

| Nome da variável | Valor |
|---|---|
| `GOOGLE_SERVICE_ACCOUNT_EMAIL` | e-mail da service account (ex: `dashboard-reader@projeto.iam.gserviceaccount.com`) |
| `GOOGLE_PRIVATE_KEY` | campo `private_key` do JSON da service account — cole completo incluindo `-----BEGIN PRIVATE KEY-----` e as quebras de linha `\n` |
| `SPREADSHEET_ID` | `1T6QihhlPDOUQm7B4Dv_VAT9i519JT-DVE1hphJTiSf0` |

Clique em **Save and Deploy** após adicionar as variáveis.

### Passo 4 — Testar o Worker

Abra no navegador (substituindo pelo seu domínio real):

```
https://dashboard-api.soft-sunset-5004.workers.dev/?sheet=kiwify
https://dashboard-api.soft-sunset-5004.workers.dev/?sheet=metaads
```

Deve retornar um array JSON. Se retornar erro, verifique:
- Se a planilha está compartilhada com o e-mail da service account
- Se a `GOOGLE_PRIVATE_KEY` foi colada corretamente (com `\n` preservados)

### Passo 5 — Subir o index.html no GitHub

1. No repositório → **Add file → Upload files**
2. Arraste o `index.html`
3. Commit

### Passo 6 — Ativar GitHub Pages

1. Repositório → **Settings → Pages**
2. Source: **Deploy from a branch** → Branch: `main` → pasta: `/ (root)`
3. Clique em **Save**
4. URL gerada: `https://suporteb16-collab.github.io/dashboard-b16-revista-catolicismo`

---

## Estrutura das abas da planilha

### Aba `kiwify`
| Coluna | Descrição |
|---|---|
| `order_ref` | ID do pedido |
| `order_status` | Status — filtrar por `paid` |
| `Faturamento` | Valor da venda |
| `created_at` | Data (ISO 8601 ou YYYY-MM-DD) |
| `TrackingParameters_utm_content` | Criativo |
| `TrackingParameters_utm_medium` | Conjunto de anúncios |
| `TrackingParameters_utm_term` | Posicionamento |
| `TrackingParameters_utm_campaign` | Campanha |
| `Ad Name` | Nome do anúncio (opcional, para cruzamento) |
| `Ad Set Name` | Nome do conjunto (opcional) |

### Aba `metaads`
| Coluna | Descrição |
|---|---|
| `Date` | Data (YYYY-MM-DD) |
| `Spend` | Valor gasto |
| `Impressions` | Impressões |
| `Reach` | Alcance |
| `Link Clicks` | Cliques no link |
| `Campaign Name` | Nome da campanha (usado para `[Q]`/`[F]`) |
| `Ad Name` | Nome do anúncio |
| `Ad Set Name` | Nome do conjunto |
| `CPM` | Custo por mil impressões |
| `CTR` | Click-through rate |
| `Frequency` | Frequência média |

---

## Segmentação de público

```
Campaign Name contém [Q] → Público Quente
Campaign Name contém [F] → Público Frio
demais                    → Outros
```

---

## Métricas calculadas

| Métrica | Fórmula |
|---|---|
| Vendas | COUNT onde order_status = paid |
| Faturamento | SUM(Faturamento) onde paid |
| Investimento | SUM(Spend) |
| ROAS | Faturamento ÷ Investimento |
| CAC | Investimento ÷ Vendas |
| CPM | Média de CPM (Meta Ads) |
| CTR | Média de CTR (Meta Ads) |
| Taxa de Carregamento | Page Views ÷ Cliques no Link |
| Taxa Checkout/Page | Checkouts ÷ Page Views |
| Taxa Venda/Checkout | Vendas ÷ Checkouts |

---

## Restringir CORS (recomendado após deploy)

No `worker.js`, troque `'Access-Control-Allow-Origin': '*'` por:

```js
'Access-Control-Allow-Origin': 'https://suporteb16-collab.github.io'
```
