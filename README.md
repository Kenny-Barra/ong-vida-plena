# ONG Vida Plena — Gestão de Eventos e Beneficiários (Airtable)

Banco de dados visual **No-Code** que organiza os eventos, os beneficiários atendidos e o histórico de participação da **ONG Vida Plena**, substituindo planilhas manuais e grupos de WhatsApp por uma base única, relacional e com automações.

> Trabalho da disciplina **"Construindo Aplicações Visuais com Integração em Plataformas No-Code e Low-Code"** — UniFECAF.

---

## Índice

1. [Visão geral](#1-visão-geral)
2. [Links do projeto](#2-links-do-projeto)
3. [O problema e a solução](#3-o-problema-e-a-solução)
4. [Por que Airtable](#4-por-que-airtable)
5. [Modelo de dados](#5-modelo-de-dados)
6. [Campos automáticos](#6-campos-automáticos)
7. [Formulários](#7-formulários)
8. [Automações de e-mail](#8-automações-de-e-mail)
9. [Dashboard (Interface)](#9-dashboard-interface)
10. [Ética e segurança (LGPD)](#10-ética-e-segurança-lgpd)
11. [Entregáveis e evidências](#11-entregáveis-e-evidências)
12. [Organização dos arquivos](#12-organização-dos-arquivos)

---

## 1. Visão geral

| | |
|---|---|
| **Ferramenta** | Airtable (banco visual No-Code) |
| **Entidades** | Beneficiários · Eventos · Inscrições |
| **Relacionamento** | N:N (muitos-para-muitos) via tabela de junção *Inscrições* |
| **Formulários** | 3 (cadastro de beneficiário, cadastro de evento, inscrição em evento) |
| **Automações** | 3 e-mails automáticos (nova inscrição, novo beneficiário, novo evento) |
| **Dashboard** | Interface "Painel Vida Plena" com indicadores e gráficos |
| **Dados de teste** | 11 beneficiários · 6 eventos · 20 inscrições |

---

## 2. Links do projeto

- **Base do projeto (visualização):** https://airtable.com/appovZC8oqmwlf4bt/shrL609n4rMAdxdZz
- **Formulário — Cadastro de Beneficiário:** https://airtable.com/appovZC8oqmwlf4bt/pagWqutX0H3A73zAI/form
- **Formulário — Cadastro de Evento:** https://airtable.com/appovZC8oqmwlf4bt/pag0sODizSTc8kDPx/form
- **Formulário — Inscrição em Evento:** https://airtable.com/appovZC8oqmwlf4bt/pagASAKkzpG0zRiGR/form

> 🔒 Os formulários são **protegidos por senha**. Senha de acesso: **`102030`**

---

## 3. O problema e a solução

**Antes.** A ONG controlava tudo em planilhas soltas e grupos de WhatsApp — gerando dados duplicados, perda de informação, nenhuma visão de quem participou de quais eventos e avisos feitos um a um, na mão.

**Depois.** Uma base única no Airtable onde:
- cada **beneficiário** existe **uma vez** e tem um histórico próprio;
- cada **evento** lista automaticamente quem se inscreveu;
- a tabela **Inscrições** conecta os dois lados (um beneficiário em vários eventos, um evento com vários beneficiários);
- **formulários** recebem cadastros e inscrições sem dar acesso à base;
- **e-mails automáticos** confirmam cadastros, inscrições e avisam sobre novos eventos.

---

## 4. Por que Airtable

O desafio exige solução **No-Code/Low-Code** porque a equipe da ONG não tem conhecimento técnico. Entre as opções pesquisadas (Airtable, Supabase, Xano, Make, Zapier), o **Airtable** foi escolhido por reunir, numa só ferramenta e sem programação: banco **relacional** com N:N nativo, interface de **planilha** familiar, **formulários**, **automações** embutidas, **Interfaces/dashboards** e **plano gratuito** compatível com o porte da ONG.

---

## 5. Modelo de dados

Três tabelas, relacionadas em **N:N** pela tabela de junção **Inscrições**:

```
┌─────────────────┐         ┌─────────────────┐         ┌─────────────────┐
│  BENEFICIÁRIOS  │ 1     N │   INSCRIÇÕES    │ N     1 │     EVENTOS     │
│                 │─────────│  (junção N:N)   │─────────│                 │
│ Nome            │         │ ID Inscrição    │         │ Nome do evento  │
│ Data nascimento │         │ Data inscrição  │         │ Tipo            │
│ Idade (fórmula) │         │ Status          │         │ Data / Horário  │
│ Telefone        │         │ Canal           │         │ Local / Região  │
│ E-mail          │         │ Beneficiário ↔  │         │ Vagas           │
│ Região          │         │ Evento ↔        │         │ Status          │
│ Status          │         │ Observações     │         │ Descrição       │
└─────────────────┘         └─────────────────┘         └─────────────────┘
```

Cada **inscrição** liga **um beneficiário a um evento** — é isso que permite o **histórico de participação** (ex.: "a Ana se inscreveu em 3 eventos").

### Dicionário de dados (campos)

**Beneficiários**

| Campo | Tipo |
|---|---|
| Nome *(primário)* | Single line text |
| Data de nascimento | Date |
| Idade | Fórmula — `DATETIME_DIFF(TODAY(); {Data de nascimento}; 'years')` |
| Telefone | Phone number |
| E-mail | Email |
| Região | Single select |
| Data de cadastro | Data/hora |
| Status | Single select (Ativo · Inativo) |
| Observações | Long text |
| Inscrições | Link → Inscrições |

**Eventos**

| Campo | Tipo |
|---|---|
| Nome do evento *(primário)* | Single line text |
| Tipo | Single select (Inclusão Digital · Capacitação Profissional · Campanha de Saúde) |
| Data · Horário | Date · texto |
| Local · Região | texto · Single select |
| Vagas | Number |
| Status | Single select (Planejado · Inscrições abertas · Realizado · Cancelado) |
| Descrição | Long text |
| Inscrições | Link → Inscrições |

**Inscrições (junção N:N)**

| Campo | Tipo |
|---|---|
| ID Inscrição *(primário)* | Autonumber |
| Data da inscrição | Date (default *hoje*) |
| Status | Single select (Confirmada · Lista de espera · Cancelada) |
| Canal | Single select (Formulário · WhatsApp · Presencial · Importação) |
| Observações | Long text |
| Beneficiário | Link → Beneficiários |
| Evento | Link → Eventos |
| E-mail (benef.) | Lookup (Beneficiário → E-mail) |

---

## 6. Campos automáticos

A base usa recursos do Airtable para preencher campos sozinha — sem digitação manual:

| Campo (tabela) | Tipo | Como funciona |
|---|---|---|
| **Idade** (Beneficiários) | Fórmula | `DATETIME_DIFF(TODAY(); {Data de nascimento}; 'years')` — calculada a partir da data de nascimento, sempre atualizada |
| **ID Inscrição** (Inscrições) | Autonumber | numera cada inscrição automaticamente |
| **Data da inscrição** (Inscrições) | Date (default *hoje*) | novas inscrições já entram com a data atual |
| **E-mail (benef.)** (Inscrições) | Lookup | puxa o e-mail do beneficiário para a automação de e-mail |

---

## 7. Formulários

Três formulários públicos (aba **Forms** do Airtable), protegidos por senha:

1. **Cadastro de Beneficiário** — coleta nome, data de nascimento, telefone, e-mail e região. (A idade é calculada sozinha.)
2. **Cadastro de Evento** — registra novos eventos (tipo, data, local, vagas, etc.).
3. **Inscrição em Evento** — liga um beneficiário a um evento (gera o histórico).

Links na [seção 2](#2-links-do-projeto).

---

## 8. Automações de e-mail

Três automações ativas (aba **Automations**), todas com gatilho *"When a record is created"*:

| Automação | Gatilho | Ação |
|---|---|---|
| **Nova Inscrição** | inscrição criada | e-mail de **confirmação** ao beneficiário |
| **Novo Beneficiário** | beneficiário criado | e-mail de **boas-vindas** |
| **Novo Evento** | evento criado | e-mail de **aviso** à coordenação |

> Observação: no plano atual, o Airtable só entrega e-mail para endereços de colaboradores verificados — por isso os testes foram feitos com um e-mail próprio (evidências em [`evidencias/`](evidencias/)).

---

## 9. Dashboard (Interface)

Interface **"Painel Vida Plena"** (aba *Interfaces*) com:
- indicadores: **Total de Beneficiários** e **Total de Inscrições**;
- gráficos: **Eventos por Tipo** e **Inscrições por região**;
- lista dos **próximos eventos com inscrições abertas**.

---

## 10. Ética e segurança (LGPD)

A base trata **dados pessoais** de população vulnerável, então o projeto considera a **LGPD**:
- **Minimização:** coleta só o necessário (sem CPF/renda).
- **Finalidade e consentimento:** o formulário informa o uso dos dados.
- **Controle de acesso:** base privada; o público interage só pelos **formulários** (protegidos por senha), sem ver dados de terceiros; o avaliador recebe link somente leitura.

Detalhamento na [Parte Teórica](docs/parte-teorica.md).

---

## 11. Entregáveis e evidências

| # | Entregável | Onde está |
|---|---|---|
| 1 | **Parte Teórica** | [`docs/parte-teorica.md`](docs/parte-teorica.md) |
| 2 | **Parte Prática** (base + formulários + automações + dashboard) | Airtable (links na seção 2) + [`evidencias/`](evidencias/) |
| 3 | **Vídeo Pitch** | link do YouTube: `[colar]` |

Os **prints das evidências** ficam na pasta [`evidencias/`](evidencias/).

---

## 12. Organização dos arquivos

```
ong-vida-plena/
├── README.md                       ← este arquivo (visão geral + dicionário de dados)
├── docs/
│   └── parte-teorica.md            ← Parte Teórica (entregável 1)
└── evidencias/                     ← prints que comprovam o sistema funcionando
```

**Tecnologias:** Airtable (banco visual, formulários, automações e Interfaces), No-Code.
