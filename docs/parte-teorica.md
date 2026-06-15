# Parte Teórica — Banco de Dados Visual para a ONG Vida Plena

**Disciplina:** Construindo Aplicações Visuais com Integração em Plataformas No-Code e Low-Code
**Ferramenta:** Airtable · **Solução:** gestão de eventos, beneficiários e histórico de participação

---

## 1. Levantamento de requisitos

O levantamento partiu do cenário descrito pela ONG: o controle era feito em planilhas manuais e grupos de WhatsApp, o que gerava **dados duplicados, perda de informação e nenhuma visão consolidada** de quem participou de quê. A partir da dor relatada, traduzi as necessidades em requisitos:

**Requisitos funcionais**
- Cadastrar e visualizar **eventos** planejados (tipo, data, local, vagas, status).
- Registrar **beneficiários** com nome, idade, contato e região.
- Permitir que **um beneficiário se inscreva em vários eventos**, com **histórico visível**.
- **Automatizar notificações** (e-mail de confirmação) e receber inscrições por **formulário**.

**Requisitos não funcionais**
- Operável por equipe **sem conhecimento técnico** (No-Code).
- Dados **organizados, acessíveis e seguros** (LGPD).
- **Baixo custo** e implantação rápida (a ONG será avaliada por um investidor social).

A técnica usada foi a **análise do enunciado/stakeholder** combinada com a modelagem das entidades que aparecem naturalmente no discurso ("eventos", "beneficiários", "participação"), validando cada requisito contra a pergunta: *"isso ajuda a saber quem participou de quê e a manter as pessoas informadas?"*.

## 2. Raciocínio da modelagem

O ponto central é a relação **muitos-para-muitos**: um beneficiário participa de vários eventos e um evento recebe vários beneficiários. Em planilha, isso vira duplicação (a mesma pessoa repetida em cada linha de evento). A boa prática de banco relacional resolve com uma **tabela de junção**.

Por isso o modelo tem **três entidades**: `Beneficiários` e `Eventos` como entidades principais, e `Inscrições` como **entidade associativa** que conecta as duas. Cada inscrição representa "esta pessoa neste evento", carregando os atributos próprios do vínculo (data da inscrição, status e canal de origem). Essa decisão:

- **elimina a duplicação** dos dados pessoais (cada beneficiário existe uma vez);
- transforma o **histórico de participação** em uma consulta natural (as inscrições de uma pessoa, ou de um evento, aparecem vinculadas dos dois lados);
- permite acompanhar o **status** de cada inscrição (confirmada, lista de espera, cancelada), apoiando os indicadores de impacto da ONG.

A modelagem ainda aproveita recursos do Airtable para **reduzir erro e digitação manual**: a **idade** é calculada por fórmula a partir da data de nascimento (sempre atualizada), o **código da inscrição** é gerado automaticamente (autonumber) e a **data da inscrição** assume a data atual por padrão.

## 3. Justificativa da escolha da ferramenta

Entre as opções No-Code/Low-Code pesquisadas, comparei:

| Ferramenta | Perfil | Adequação ao caso |
|---|---|---|
| **Airtable** | Banco relacional visual + automações + formulários + dashboards | **Escolhida** — tudo em uma ferramenta, curva baixa |
| Supabase | Banco SQL (Postgres) com APIs | Poderoso, mas exige perfil técnico |
| Xano | Back-end No-Code | Foco em API, sem planilha visual amigável |
| Make / Zapier | Orquestração de automações | Complementam, mas não são o banco em si |

O **Airtable** foi escolhido porque entrega, numa só plataforma e sem programação: (1) **banco relacional** com relacionamentos N:N nativos (campo *Link to another record*); (2) **interface de planilha** familiar para quem vem do Excel; (3) **formulários** que coletam dados sem expor a base; (4) **automações** (e-mail/webhook) embutidas; (5) **views e Interfaces** (Calendário, Kanban, dashboards); e (6) **plano gratuito** compatível com o porte da ONG. É o melhor equilíbrio entre simplicidade para a equipe e robustez relacional para os dados.

## 4. Estrutura relacional

**Entidades, atributos e relacionamentos:**

- **Beneficiários** — *Nome* (PK lógica), Data de nascimento, Idade (calculada por fórmula), Telefone, E-mail, Região, Data de cadastro, Status.
- **Eventos** — *Nome do evento* (PK lógica), Tipo, Data, Horário, Local, Região, Vagas, Status, Descrição.
- **Inscrições** (junção) — *ID Inscrição* (PK, autonumber), Data da inscrição, Status, Canal, **Beneficiário** (FK → Beneficiários), **Evento** (FK → Eventos).

**Cardinalidade:** `Beneficiários (1) —— (N) Inscrições (N) —— (1) Eventos`. A tabela `Inscrições` decompõe o N:N entre `Beneficiários` e `Eventos` em duas relações 1:N, padrão clássico de modelagem relacional. No Airtable, cada relação é um campo de link bidirecional, então os registros aparecem vinculados nos dois lados automaticamente (cada beneficiário lista suas inscrições; cada evento lista suas inscrições).

## 5. Ética e segurança da informação

A base trata **dados pessoais de população vulnerável**, o que exige cuidado à luz da **LGPD (Lei 13.709/2018)**:

- **Minimização e finalidade:** coletamos apenas dados necessários à operação dos eventos (sem CPF, renda ou dados sensíveis desnecessários); a finalidade é informada no formulário.
- **Consentimento:** o formulário traz aviso de uso dos dados; para menores de idade, há registro da autorização do responsável.
- **Controle de acesso:** a base é privada e editável só por colaboradores autorizados; o público interage apenas pelo **formulário**, sem ver dados de terceiros; o avaliador recebe link **somente leitura**.
- **Integridade e disponibilidade:** o Airtable mantém histórico de revisões e backups, reduzindo o risco de perda que existia nas planilhas soltas.
- **Transparência:** a pessoa pode solicitar correção ou exclusão dos seus dados, atendendo aos direitos do titular.

Em resumo, a solução não só organiza os dados como os trata de forma **responsável e segura**, requisito tão importante quanto a funcionalidade para uma organização que lida com pessoas em situação de vulnerabilidade.
