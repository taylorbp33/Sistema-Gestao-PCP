# Sistema de Gestão PCP — Produção, Precificação e Custos (Google Sheets + Apps Script)

Um web app construído em **Google Apps Script** usando **Google Sheets como banco de dados**, voltado para operações de produção (especialmente alimentos) que precisam de:

- **PCP (Planejamento e Controle da Produção)** com **Ordens de Produção (OP)** e impressão de comandas/checklists
- **Fichas Técnicas (BOM)** com subprodutos e explosão recursiva
- **Cálculo de custos (MP/Emb, MO e Máquina/Recurso)**, **CTM**, **impostos** e **rentabilidade**
- **Precificação** (tabela por cliente/setor + precificação rápida)
- **Gestão de Clientes e Pedidos**, com histórico e status

> Objetivo do projeto: transformar uma planilha em um “mini-ERP” de produção e precificação, com UX moderna, regras de negócio e relatórios prontos para imprimir.

---

## Por que esse projeto importa (contexto de negócio)
Na prática, muita operação pequena/média vive com:
- custos “no feeling”
- fichas técnicas despadronizadas
- retrabalho na separação de ingredientes
- dificuldade em manter margem por canal (varejo/atacado/marketplace)
- PCP informal (sem OP, sem checklist, sem rastreio por lote)

Este sistema centraliza tudo isso em uma única aplicação, com:
- dados estruturados em abas (Sheets)
- regras no backend (Apps Script)
- interface web com módulos, permissões e relatórios

---

## Principais funcionalidades (o que o app entrega)

### 1) PCP: Ordens de Produção (OP)
- **Geração de OP** por produto (ficha técnica), com:
  - produto (ficha), **quantidade**, **lote**, **data de produção** e validade
  - status: **Planeada → Em Produção → Concluída**
- **Histórico de OPs com filtros** (produto, lote, período)
- **Impressão da OP (comanda/checklist)**:
  - lista pronta para o chão de fábrica: **ingredientes + embalagens + subprodutos**
  - layout pensado para impressão (paisagem) e execução operacional

**O diferencial técnico aqui:** a OP usa a lógica de estrutura de produto (BOM) para gerar uma lista de montagem consistente e repetível.

---

### 2) Fichas Técnicas (BOM) com subprodutos (recursivo)
- Cadastro completo de ficha:
  - ingredientes
  - **subprodutos** (receitas dentro de receitas)
  - embalagem
  - recursos de produção (máquina / mão de obra)
- Cálculo de custo **recursivo**, com proteção contra ciclos (dependências circulares)
- “Disponível para venda” para controlar o que aparece em precificação/simulador/tabela
- Impressão do conjunto de fichas (“livro”)

**O que isso demonstra para recrutadores:**
- modelagem de dados + regra de negócio
- recursão e consistência de cálculo em árvore (produto → subproduto → insumos)

---

### 3) Custos: MP/Emb, MO e Máquina/Recurso
- Ingredientes com custo real por unidade:
  - preço embalagem / quantidade + frete por unidade
- Recursos de produção com:
  - produtividade (un/h) e custo hora
  - classificação do custo em **mão de obra** vs **máquina**
- Recalcular custos de todas as fichas (consistência global)

---

### 4) CTM + Tributação + Rentabilidade (Simulações)
- CTM configurável por categoria (percentual + ativo)
- Tributação configurável (com permissão de edição restrita)
- **Simulador de Pedido** com:
  - múltiplos itens
  - detalhamento de impostos/CTM por item e consolidado
  - cálculo de lucro líquido e margens
- Impressão do relatório de simulação (financeiro + operacional)

---

### 5) Precificação (Tabela e modo rápido)
- **Margens por Setor** (markup por canal)
- **Tabela de preços por cliente/setor** (com impressão)
- **Overrides administrativos** de repasse fixo por produto/setor
- Precificação rápida para tomada de decisão

---

### 6) Comercial: Clientes e Pedidos
- Clientes com:
  - setor e markup sugerido
  - CTMs ativos por cliente (personalização do custo variável)
- Pedidos:
  - CRUD com itens armazenados em JSON
  - histórico com filtros por cliente/período
  - status (ex.: pendente/em produção/entregue/cancelado)

---

## Arquitetura (como eu estruturei)
**Frontend (webapp.html)**
- Single Page App simples (router por abas)
- Módulos separados por renderizadores (`renderPedidos`, `renderClientes`, `renderProducao`, etc.)
- Impressões com HTML + CSS dedicado (relatórios e OP)

**Backend (code.gs)**
- APIs `api_*` consumidas pelo frontend via `google.script.run`
- Regras de negócio:
  - cálculo recursivo de custos
  - simulação de pedido (impostos + CTM)
  - filtros e normalização de datas
  - permissões por usuário/perfil

**Persistência (Google Sheets)**
- Abas como tabelas (“database”):
  - `Ingredientes`, `FichasTecnicas`, `RecursosProducao`, `Clientes`, `Pedidos`, `OrdensProducao`, etc.
- Itens complexos armazenados em colunas JSON (ex.: `Itens_JSON`, `Recursos (JSON)`)

---

## Lógica “core” (o que eu considero o coração do sistema)

### Explosão de ficha (BOM) + subprodutos
- O app trata o produto como uma árvore:
  - produto final → subprodutos → ingredientes/embalagens
- O cálculo:
  - multiplica quantidades pelo fator (quantidade planejada / rendimento)
  - agrega ingredientes por origem (rastreamento “de qual ficha veio”)
  - evita ciclos e inconsistências

### Custeio por recursos (MO x Máquina)
- O custo não é “tempo fixo”: é calculado por produtividade (un/h) e custo hora
- Classifica custos por tipo de recurso (mão de obra vs máquina)

### Simulação financeira (pedido)
- Consolida receita, CMV, CTM e impostos
- Retorna:
  - visão gerencial (lucro líquido/margens)
  - visão operacional (lista de separação + checklist)

---

## Competências demonstradas (para recrutadores)
- **Full-stack com Apps Script**: UI web + API + persistência em Sheets
- **Modelagem de dados pragmática** (tabelas + JSON onde faz sentido)
- **Algoritmos de negócio**: recursão, agregação, consistência de cálculo e validações
- **Controle de acesso**: permissões por usuário/perfil + módulos habilitados dinamicamente
- **UX orientada à operação**: telas “do dia a dia” + impressão pronta para uso
- **Manutenibilidade**: APIs coesas (`api_*`), helpers de parse/normalização, filtros por data/lote

---

## Tech stack
- Google Apps Script (JavaScript)
- Google Sheets (persistência)
- HTML/CSS/JS
- TailwindCSS (CDN), Font Awesome, Chart.js

---

## Como executar (deploy)
1. Crie uma Planilha Google (será o “banco”).
2. Extensões → Apps Script
3. Crie/cole os arquivos:
   - `code.gs`
   - `index.html`
   - `webapp.html`
4. No Sheets, execute o setup pelo menu do app:
   - **⚙️ App Admin → Verificar/Criar Abas**
5. Publique:
   - Implantar → Nova implantação → Aplicativo da Web

---

## Abas (tabelas) criadas/geridas
- Segurança/Controle: `Users`, `Perfis`, `AuditLog`
- Produção: `Ingredientes`, `FichasTecnicas`, `RecursosProducao`, `OrdensProducao`
- Estratégia: `CustosFixos`, `CTM_Categorias`, `Margens_Setor`, `Tributacao_Config`
- Comercial: `Clientes`, `Pedidos`
- Precificação: `TabelaPrecos_Overrides`
- App: `App_Config` (logo e preferências)

---

## Próximos passos (evolução natural)
- Estoque completo + baixa automática ao concluir OP (MP/embalagens/subprodutos)
- Hash de senha e melhorias de autenticação (segurança)
- Vincular pedidos → geração automática de OP (PCP integrado ao comercial)
- Relatórios gerenciais adicionais (curva ABC, margem por cliente/produto)

---

## Screenshots / Demo
<img width="2866" height="1346" alt="image" src="https://github.com/user-attachments/assets/97228d6b-3a1e-4fcf-b694-ba0a8686f272" />
<img width="994" height="646" alt="image" src="https://github.com/user-attachments/assets/813cf55b-f1fe-4774-92ae-cda16f045359" />
<img width="1006" height="447" alt="image" src="https://github.com/user-attachments/assets/4b8ff1ca-83b2-4853-a196-cc590cc65987" />
<img width="1006" height="642" alt="image" src="https://github.com/user-attachments/assets/89f7b78a-1fb8-4f30-a1a3-1edc59790320" />
<img width="1151" height="502" alt="image" src="https://github.com/user-attachments/assets/7cf4c222-e5db-4119-9c9f-491c31326273" />
<img width="987" height="621" alt="image" src="https://github.com/user-attachments/assets/329fbd44-2356-4303-8cb4-ab03ce60395c" />
<img width="1031" height="540" alt="image" src="https://github.com/user-attachments/assets/795a2730-908a-4f29-8a07-994944c6c466" />
<img width="1002" height="621" alt="image" src="https://github.com/user-attachments/assets/36ca3b60-702d-4343-82a2-409edcc83d17" />
<img width="1030" height="592" alt="image" src="https://github.com/user-attachments/assets/03af28b9-2e31-4c6f-8a14-53d0cd8c870f" />
<img width="1378" height="603" alt="image" src="https://github.com/user-attachments/assets/c2da3052-5113-49c4-99dc-8743c89cabe4" />
<img width="447" height="641" alt="image" src="https://github.com/user-attachments/assets/c3fb3fe0-de7a-45ae-9fb5-f0301f06d2b3" />
<img width="422" height="567" alt="image" src="https://github.com/user-attachments/assets/aa9c1699-bda6-4b90-947d-2c8352e0b697" />
 <img width="417" height="469" alt="image" src="https://github.com/user-attachments/assets/06cc50e1-ae01-4a4d-92e1-9bd67d11034f" />
 <img width="437" height="371" alt="image" src="https://github.com/user-attachments/assets/ed58444a-ed21-46df-8e40-54da30d5648c" />
<img width="413" height="641" alt="image" src="https://github.com/user-attachments/assets/c176537e-6973-483a-a4e4-cf8f7444bf5d" />
<img width="974" height="437" alt="image" src="https://github.com/user-attachments/assets/69cd5b9f-92bf-413e-934d-0e6bc7f18e64" />

---

