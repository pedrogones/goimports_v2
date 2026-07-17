# GO Imports V2 — Bootstrap OpenCart 4 + PHP 8.3

Documento mestre para montar o projeto **do zero** no repositório [pedrogones/goimports_v2](https://github.com/pedrogones/goimports_v2.git), reaproveitando o tema **`template_v2`** e **toda a funcionalidade** da loja atual (hoje OpenCart **3.0.3.1** / OpenCart Brasil **1.3.0**).

**Princípio:** o que era legado (`basico` + OCMOD + apps Loja5/Codemarket) **não some** — vira implementação **nativa / portada** no OpenCart 4, com `template_v2` como **único** tema de vitrine.

---

## 1. Identidade do projeto

| Item | Valor |
|------|--------|
| Repositório | `https://github.com/pedrogones/goimports_v2.git` |
| GitHub user | `pedrogones` |
| Git author name | `pedrogones` |
| Git author email | `pedrotargin@gmail.com` |
| Loja alvo | GO Imports |
| Tema de vitrine | `template_v2` (único; sem `basico` em produção) |
| Backend | OpenCart **4.x** (última estável 4.x no início do projeto) |
| Runtime | PHP **8.3** |
| Banco | MySQL 8.x / MariaDB 10.6+ |
| Prefixo de tabelas | `oc_` (manter, salvo decisão explícita) |
| Idioma padrão | `pt-br` |
| Timezone | `America/Sao_Paulo` |

### 1.1 Configurar Git **neste repositório** (não global)

```bash
cd /caminho/goimports_v2
git init   # se ainda vazio
git remote add origin https://github.com/pedrogones/goimports_v2.git

# Somente neste repo:
git config user.name "pedrogones"
git config user.email "pedrotargin@gmail.com"

git checkout -b main
```

Confirmar:

```bash
git config user.name
git config user.email
# pedrogones
# pedrotargin@gmail.com
```

---

## 2. Objetivo e escopo

### 2.1 Objetivo

Subir uma loja **OpenCart 4 + PHP 8.3** limpa, com:

1. Tema **`template_v2`** (markup, CSS, JS e bindings já evoluídos no backend atual).
2. **Paridade funcional** com a loja atual (pagamentos, fretes, marketing, fraud, captcha, campos custom, CEP, home modules, etc.).
3. Código **sem depender** do tema `basico` nem de OCMOD frágeis como fonte da verdade (OCMOD só como ponte temporária, se inevitável).

### 2.2 Fonte de verdade (repo atual / legado)

Projeto de referência: `goimports-backend` (este monorepo OpenCart 3).

| Camada | Origem no legado | Destino no V2 |
|--------|------------------|---------------|
| Tema | `catalog/view/theme/template_v2/` | Portar 1:1 e adaptar a OC4 (Twig/paths/events) |
| Extensão admin do tema | `admin/controller/extension/theme/template_v2.php` | Recriar no padrão OC4 |
| Controllers custom | `catalog/controller/...` (CEP, PDP, checkout, módulos) | Reescrever / adaptar para OC4 |
| Pagamentos / fretes BR | Yapay, PagHiper, Cielo, Melhor Envio, Correios, etc. | Portar ou substituir por versão OC4 oficial |
| Dados | Dump MySQL prod/staging | Migrar schema 3→4 + seed tema |

### 2.3 Fora de escopo (fase 0)

- Manter `journal2` / `basico` como tema ativo.
- PHP &lt; 8.3.
- OpenCart 3.x em produção V2.
- Dependência permanente de VQMod (hoje já desabilitado).

---

## 3. Stack alvo

### 3.1 Runtime

| Componente | Versão |
|------------|--------|
| PHP | **8.3** (FPM ou Apache `mod_php`) |
| Extensões PHP | `gd`, `intl`, `mysqli`/`pdo_mysql`, `zip`, `curl`, `mbstring`, `xml`, `json`, `openssl` |
| MySQL | 8.0+ (ou MariaDB equivalente) |
| Web server | Apache 2.4 ou Nginx + PHP-FPM |
| Composer | 2.x |

### 3.2 OpenCart

- Base: **OpenCart 4.x** (core oficial +, se necessário, pacote Brasil compatível com OC4).
- Estrutura OC4 difere da 3.x (namespaces, DI, events, admin routes, storage). Toda portabilidade deve seguir a API OC4, **não** copiar controllers 3.x cegamente.

### 3.3 Docker (recomendado no dia 1)

Alvo:

- Imagem base: `php:8.3-apache-bookworm` (ou `php:8.3-fpm` + nginx).
- Serviço `web` + MySQL (container ou host).
- Volumes: `system/storage/{cache,logs,session,upload}`, `image/cache`.
- Templates de `config.php` / `admin/config.php` via env (`DB_*`, `HTTP_SERVER`, `HTTPS_SERVER`).
- Entrypoint: `composer install`, permissões storage, gerar configs.

Referência do legado (atualizar PHP 8.1 → **8.3**): `Dockerfile`, `docker-compose.yml`, `Makefile`, `bin/*`, `.env.example`.

---

## 4. Tema `template_v2` — o que deve existir no dia 1

### 4.1 Árvore mínima

```
catalog/view/theme/template_v2/
├── template/
│   ├── common/          # header, footer, home, menu, maintenance, success, trust_bar, newsletter…
│   ├── product/         # category, product, search, special, manufacturer, cards
│   ├── checkout/        # cart, checkout, addresses, methods, confirm
│   ├── account/         # login, register, orders, address, wishlist…
│   ├── information/     # contact, sitemap…
│   ├── extension/       # captcha, module (slideshow, featured, latest, banners…)
│   └── error/
├── stylesheet/
│   ├── main.css
│   └── pikaday.css
├── javascript/
│   ├── main.js
│   ├── pikaday.js
│   ├── oc-datetimepicker.js
│   ├── init*.js
│   └── components/
└── image/
```

### 4.2 Superfícies obrigatórias (paridade)

| Área | Deve funcionar |
|------|----------------|
| Home | `content_top` / `content_bottom`, slideshow/hero, carrosséis de produto, promo banners, trust bar, newsletter |
| Header/menu | Logo, busca, conta, wishlist, carrinho (badge), menu categorias (`top=1` + idioma) |
| Listagem | Categoria, busca, promoções, fabricante — **omitir produto sem imagem no disco** |
| PDP | Galeria, opções, preço/pix, Trustvox, badges, Go Express / WhatsApp, `entregue_por` |
| Carrinho / checkout | Fluxo completo + captcha Google v2 checkbox |
| Conta | Cadastro (datepicker), login, pedidos, endereços, wishlist, returns |
| Contato / informações | Layout v2 + captcha |
| Manutenção | Template v2 (não cair só no legado) |

### 4.3 Extensão admin do tema

- Registrar tema `template_v2` no admin OC4.
- Settings: status, directory, image sizes (product, category, related, cart, wishlist…).
- Ativação: `config_theme = template_v2` (ou equivalente OC4).

### 4.4 Regras de portabilidade do tema

1. **Zero fallback silencioso para `basico`** em produção V2.
2. Remover dependências de CSS/JS do `basico` no `header.twig` (jQuery/Bootstrap/Slick: decidir stack única; preferir o que o `main.js` do v2 já usa + jQuery só se o checkout OC ainda exigir).
3. Manter IDs/classes que o checkout AJAX do OpenCart precisa (`#collapse-*`, `cart.add`, etc.).
4. Datepicker: **Pikaday local** (não CDN), como no backend atual.
5. Captcha: template `extension/captcha/google` com `data-size="normal"` (checkbox “Não sou um robô”).

---

## 5. Paridade funcional — checklist completo

Tudo abaixo existia / estava em uso no legado. No V2: **manter comportamento**; se a extensão não tiver build OC4, **reimplementar** ou **substituir por equivalente oficial**, sem perder UX.

### 5.1 Pagamentos

| Integração | Notas |
|------------|--------|
| Yapay (boleto / cartão / pix) | `yapayb`, `yapayc`, `yapayp` + fee cartão no listing |
| PagHiper (boleto / pix) | Incl. OCMOD legado → portar |
| Cielo API Pro (Loja5) | crédito / boleto / débito / pix / qrcode / tef |
| PagSeguro (transparente) | boleto / crédito / débito |
| PagueVeloz | boleto / crédito + CPF/RG/número endereço |
| BoletoBancario / segunda via | manter fluxo de link de pagamento |

### 5.2 Frete / logística

| Integração | Notas |
|------------|--------|
| Melhor Envio (Codemarket) | `code_melhorenvio` + Code API |
| Correios / Correios Pro | online/offline |
| Frete grátis Pro | regras Loja5 |
| SIGEP + Pré-postagem | admin/logística |
| Frenet / Faixa CEP | se ainda usados em prod |
| Consulta frete no PDP | CEP + ViaCEP (`common/postcode`) |

### 5.3 Marketing / CRM / reviews

| Integração | Notas |
|------------|--------|
| Trustvox | widget PDP + checkout; store id configurável (sem hardcode no tema) |
| SmartHint | sync produto/pedido; **PHP 8 safe** (IDs de categoria escalares) |
| Active Campaign | sync pedidos / conexão API |
| Facebook Business / Pixel | catálogo + eventos |
| Google Analytics e-commerce | |
| OneSignal | workers + init no header v2 (hoje só no basico) |

### 5.4 Fraud / captcha / SEO

| Item | Notas |
|------|--------|
| Google reCAPTCHA **v2 checkbox** | `captcha_google_key` / `captcha_google_secret`; localhost pode usar chaves de teste |
| ClearSale API | fraud extension |
| SEO URL amigável | portar comportamento OC3 → OC4 |
| Google Shopping / feeds | Codemarket Google Shopping + feeds nativos |

### 5.5 Checkout UX

- Avaliar **um** fluxo canônico no OC4:
  - Checkout nativo OC4 + twig v2, **ou**
  - OnePageCheckout / d_quickcheckout **se** houver versão OC4 estável.
- Não manter dois stacks ativos sem dono.

### 5.6 Campos e regras de negócio custom

| Custom | Onde |
|--------|------|
| `entregue_por` | PDP / product |
| `express` (Go Express) | PDP / listing |
| Flag “novo” (date_added) | cards |
| Omitir listagem sem arquivo de imagem | `ModelToolImage::resizeListing` / equivalente OC4 |
| Situações de pedido | `config_order_status_id`, `config_processing_status`, `config_complete_status`, `config_fraud_status_id` |
| Idioma | descrições em `language_id` do `pt-br` (categorias, produtos, banners) |
| Store | `product_to_store` / `category_to_store` no `store_id` ativo |

### 5.7 Admin / ops

| Item | Notas |
|------|--------|
| Export/Import produtos | Tool V3.x → versão OC4 |
| Export clientes/pedidos | |
| Banner sortable / banner timer | se ainda usados na home |
| Bairro obrigatório | forms account/checkout |
| Etiquetas / declaração conteúdo Correios | |
| Relatório pedidos por estado | |
| Modo manutenção | template v2 + bypass só para admin logado |

---

## 6. Plano de execução (fases)

### Fase 0 — Repositório e baseline

1. Clonar / inicializar [goimports_v2](https://github.com/pedrogones/goimports_v2.git).
2. Setar `user.name` / `user.email` **locais** (seção 1.1).
3. Commit inicial: este documento + `.gitignore` + README curto.
4. Instalar OpenCart **4.x** limpo (sem dump antigo ainda).
5. Docker PHP **8.3** + MySQL; healthcheck HTTP 200 na home.

**Critério de saída:** admin e catalog sobem em PHP 8.3 sem fatal.

### Fase 1 — Tema `template_v2`

1. Copiar `catalog/view/theme/template_v2/` do legado.
2. Adaptar paths/events/Twig ao OC4 (loader, theme engine, asset URLs).
3. Registrar extensão de tema no admin.
4. Ativar `template_v2` como tema da loja.
5. Home/header/footer/listagem/PDP/conta/checkout renderizam sem cair no default quebrado.

**Critério de saída:** navegar vitrine completa com dados demo OC4.

### Fase 2 — Dados e schema

1. Dump prod/staging → ambiente V2.
2. Rodar migração oficial 3→4 (ou ferramenta de upgrade OC) + scripts de colunas custom (`entregue_por`, `express`, tabelas SmartHint/Facebook, etc.).
3. Validar:
   - `oc_language` pt-br
   - `product_description` / `category_description` / `banner_image` no `language_id` ativo
   - `product_to_store` / `category_to_store`
   - order statuses + settings de processamento/completo
4. Sync imagens: `image/catalog/` (script tipo `sync-catalog-images.sh`).

**Critério de saída:** busca/categoria/home retornam produtos reais com imagem.

### Fase 3 — Integrações críticas (checkout)

Ordem sugerida:

1. Captcha Google + config admin  
2. Frete (Melhor Envio **ou** Correios — o que estiver vivo em prod) + CEP  
3. Pagamento principal (Yapay/Cielo/PagHiper — o de maior volume)  
4. Demais gateways  
5. ClearSale (se usado no fluxo)

**Critério de saída:** pedido teste ponta a ponta (PIX/boleto/cartão conforme prod).

### Fase 4 — Marketing e ops

1. Trustvox, SmartHint, Active Campaign, Facebook, Analytics, OneSignal  
2. Feeds / Google Shopping  
3. Export/import, etiquetas, pré-postagem, SIGEP  
4. Remover OCMODs mortos; o que sobrar documentar

**Critério de saída:** paridade checklist §5 marcada.

### Fase 5 — Hardening

1. Remover tema `basico` do deploy (ou manter só como archive fora do runtime).
2. PHP 8.3 strict: eliminar `TypeError` conhecidos (SmartHint, etc.).
3. Performance (cache OC4, image cache, CDN se houver).
4. Segurança (HTTPS, secrets só em env, sem keys no git).
5. Runbook de deploy (cPanel/VPS) + backup.

---

## 7. Estrutura sugerida do repositório V2

```
goimports_v2/
├── README.md
├── docs/
│   └── GOIMPORTS-V2-BOOTSTRAP.md   ← este arquivo
├── .gitignore
├── .env.example
├── Dockerfile                     # php:8.3-…
├── docker-compose.yml
├── Makefile
├── bin/
│   ├── activate-theme-v2.sh
│   ├── configure-recaptcha.sh
│   ├── sync-catalog-images.sh
│   ├── mysql-local.sh
│   └── opencart-health.sh
├── catalog/
│   └── view/theme/template_v2/
├── admin/
├── system/
├── image/
│   ├── catalog/                   # versionar se necessário ao deploy
│   └── cache/                     # gitignore
└── ...                            # core OpenCart 4
```

### 7.1 `.gitignore` mínimo

Ignorar: `config.php`, `admin/config.php`, `.env`, `system/storage/{cache,logs,session,modification,upload}`, `image/cache/`, `vendor/` genérico.

**Não ignorar** (se o deploy depender de git pull):

- `catalog/view/theme/template_v2/`
- `system/library/code/vendor/` (CodeMarket) — se a lib continuar no projeto
- opcionalmente `image/catalog/` (já feito no legado)

---

## 8. Configurações de loja (seed obrigatório)

Após install / import:

```sql
-- Tema
UPDATE oc_setting SET value = 'template_v2' WHERE `key` = 'config_theme' AND store_id = 0;
-- (ajustar keys OC4 se o naming mudar)

-- Idioma
UPDATE oc_setting SET value = 'pt-br' WHERE `key` IN ('config_language', 'config_admin_language') AND store_id = 0;

-- Captcha Google v2
UPDATE oc_setting SET value = 'google' WHERE `key` = 'config_captcha' AND store_id = 0;
-- captcha_google_key / captcha_google_secret / captcha_google_status = 1

-- Pedidos (exemplo padrão; validar IDs reais)
-- config_order_status_id = 1
-- config_processing_status = ["2","15"]  (serialized/json conforme OC4)
-- config_complete_status   = ["3","5"]
-- config_fraud_status_id   = 8

-- Manutenção OFF em staging público
UPDATE oc_setting SET value = '0' WHERE `key` = 'config_maintenance' AND store_id = 0;
```

**Atenção OC4:** nomes de settings/events podem mudar; validar no admin após install e atualizar este bloco.

### 8.1 reCAPTCHA (prod de referência)

Site key em produção atual (`goimports.com.br`):

`6Lf8c5QpAAAAAK7ovFloRUh1DTpwQ2PWNbwjBMj-`

- Tipo: **reCAPTCHA v2 checkbox**
- Secret: só no admin / `oc_setting` / `.env` — **nunca** no git
- Domínios autorizados no Google Admin: prod + staging + `localhost`

Configurável no admin: **Extensões → Captcha → Google**.

---

## 9. Comandos de referência (Makefile alvo)

```bash
make docker-up          # sobe PHP 8.3 + app
make config-docker      # gera config.php a partir do .env
make theme-v2           # ativa template_v2 + permissões admin
make configure-recaptcha
make sync-images        # image/catalog produtos
make sync-banners
make db-import FILE=dump.sql
make health             # opencart-health.sh
```

Ajustar scripts do legado (`bin/`) para paths e settings do OC4.

---

## 10. Migração de código — regras

1. **Não** copiar `system/storage/modification/` do OC3.
2. **Não** assumir que OCMOD XML 3.x aplica em OC4.
3. Customizações preferir:
   - extensão OC4 (`extension/...`), ou
   - event listeners OC4, ou
   - override de tema.
4. Todo `TypeError` PHP 8 (string como array, null, etc.) corrigir na portagem.
5. Listagens: se arquivo de imagem não existe em `DIR_IMAGE`, **não listar** o produto (comportamento já definido no legado).
6. Idioma: produto/categoria/banner sem `*_description` no `language_id` ativo = **invisível** — seed/copy obrigatório.

---

## 11. Critérios de aceite (go-live V2)

- [ ] PHP 8.3 em prod/staging sem notices fatais
- [ ] OpenCart 4 admin + catalog
- [ ] Tema **somente** `template_v2`
- [ ] Home com banners reais + módulos de produto
- [ ] Menu categorias + footer categorias
- [ ] Listagem/PDP/carrinho/checkout/conta
- [ ] Captcha v2 checkbox validando
- [ ] Frete cotando (ao menos um carrier produtivo)
- [ ] Pagamento teste aprovado (ao menos um método produtivo)
- [ ] Pedido altera status e baixa estoque conforme `processing`/`complete`
- [ ] Trustvox / analytics / pixel conforme prod
- [ ] Manutenção OFF; admin bypass ok
- [ ] Git: `pedrogones` / `pedrotargin@gmail.com` nos commits do repo
- [ ] Secrets fora do git; `.env.example` documentado

---

## 12. Riscos e decisões abertas

| Risco | Mitigação |
|-------|-----------|
| Extensão BR sem versão OC4 | Contatar vendor (Codemarket/Loja5/Yapay) **antes** da Fase 3; senão reescrever |
| Diff grande Twig OC3→OC4 | Portar tema por superfície (home → listing → PDP → checkout) |
| Dump 3.x incompatível | Upgrade path oficial + scripts de colunas custom |
| OCMOD “NOT FOUND” massivo | Esperado ao mudar core/tema; não é go-live blocker se a feature foi reimplementada |
| Duplo checkout (OPC + d_qc) | Escolher **um** no kickoff da Fase 3 |

---

## 13. Primeiros commits sugeridos no `goimports_v2`

1. `docs: bootstrap OpenCart 4 + PHP 8.3 + template_v2`
2. `chore: scaffold OC4 + Docker PHP 8.3`
3. `feat: porta tema template_v2`
4. `feat: ativa tema e seeds de idioma/captcha`
5. `feat: porta frete/pagamento críticos`
6. `feat: porta marketing (Trustvox, SmartHint, …)`
7. `chore: remove dependências do tema basico`

---

## 14. Contatos / refs rápidas

- Repo V2: https://github.com/pedrogones/goimports_v2.git  
- Autor Git: `pedrogones` &lt;pedrotargin@gmail.com&gt;  
- Tema origem: `goimports-backend/catalog/view/theme/template_v2/`  
- Docs tema legado: `README-TEMPLATE-V2.md` (atualizar após OC4)  
- OpenCart 4 docs: documentação oficial no release escolhido  

---

**Resumo:** este projeto nasce **zerado** em OpenCart 4 / PHP 8.3, com `template_v2` como loja, e trata o legado como **especificação de paridade** — não como runtime. Tudo que a GO Imports faz hoje deve existir de forma **nova**, estável e sem depender do tema `basico`.
