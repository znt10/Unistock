<div align="center">

# UniStock

**Controle de estoque para redes com várias lojas.**

Cada unidade enxerga o próprio estoque. O gerente enxerga todas.
Quando um produto cruza o mínimo, o responsável recebe um aviso antes de faltar.

[![Produção](https://img.shields.io/badge/produ%C3%A7%C3%A3o-unistock.online-2ea44f?style=flat-square)](https://unistock.online/login)
[![Backend](https://img.shields.io/badge/backend-Django%20%2B%20DRF-092E20?style=flat-square&logo=django)](https://github.com/znt10/Unistock_Back)
[![Frontend](https://img.shields.io/badge/frontend-Next.js%2016-000000?style=flat-square&logo=nextdotjs)](https://github.com/znt10/Unistock_Front)
[![WhatsApp Bot](https://img.shields.io/badge/whatsapp-bot%20integrado-25D366?style=flat-square&logo=whatsapp)](#bot-de-whatsapp)
[![License](https://img.shields.io/badge/licen%C3%A7a-MIT-blue?style=flat-square)](#licença)

[**Acessar o sistema**](https://unistock.online/login) &middot;
[Backend](https://github.com/znt10/Unistock_Back) &middot;
[Frontend](https://github.com/znt10/Unistock_Front) &middot;
[Documentação](#documentação)

</div>

---

## Em resumo

Uma rede de lojas raramente sabe, em tempo real, quanto tem de cada produto em
cada unidade. A contagem vira planilha, a planilha atrasa, e o item acaba antes
de alguém perceber.

O UniStock fecha esse ciclo:

| | |
|---|---|
| 📦 **Estoque por unidade** | cada loja tem o próprio, com quantidade mínima definida por produto |
| 🔔 **Aviso antes de faltar** | cruzou o mínimo, o responsável recebe email — uma vez por episódio, sem spam |
| 🔁 **Pedido de reposição** | o responsável pede pelo sistema; o gerente acompanha e muda o status |
| 🧾 **PDV integrado** | a venda dá baixa no estoque na hora |
| 💬 **Bot de WhatsApp** | consulta catálogo, abre pedido e dá baixa manual no estoque, direto do zap da loja |
| 🏭 **Fábrica e etiquetas** | a fábrica recebe os pedidos das lojas e imprime uma etiqueta com QR por caixa |
| 📱 **Leitura das caixas** | a loja lê o QR no celular: chegou, aberta, acabou — o estoque anda sozinho, com desfazer |
| 👤 **Acesso por perfil** | Admin, Gerente e Responsável enxergam coisas diferentes — e isso vale na API, não só na tela |

**Stack:** Django + DRF + MySQL + Celery no backend, Next.js 16 + TypeScript no
frontend, Evolution API para o bot de WhatsApp, tudo em Docker.

---

## O que é o UniStock

O UniStock é um sistema web de gerenciamento de estoque pensado para **redes**,
não para uma loja só. A diferença importa: o problema não é saber quanto tem, é
saber quanto tem **onde**, e conseguir mover produto entre unidades sem telefone
e caderno.

O sistema centraliza produtos, quantidades e pedidos entre as unidades da mesma
rede. O gerente acompanha o estoque de cada loja; o responsável de unidade cuida
da sua e pede reposição direto pelo sistema — ou pelo WhatsApp.

### O que é "estoque" aqui

Estoque é a quantidade disponível de **um produto em uma loja específica** — não
um número global da rede. Cada registro guarda:

- quantidade atual e quantidade mínima;
- estado do produto: normal, congelado ou resfriado;
- histórico de movimentação.

Quando a quantidade cai abaixo do mínimo, o sistema notifica os responsáveis. O
aviso sai **uma vez por episódio**: enquanto o item continuar baixo, ele não
repete. Isso é de propósito — alerta que repete todo dia vira ruído e para de
ser lido.

### Perfis de acesso

| Perfil | Alcance |
|---|---|
| **Admin** | tudo, mais o Django Admin |
| **Gerente** | todas as lojas: cadastra unidade e usuário, vê o relatório geral |
| **Responsável** | apenas a própria loja |

O recorte do responsável é aplicado no backend, na consulta ao banco. Um
responsável que chame a API diretamente continua vendo só a loja dele — a
restrição não depende da interface.

### Funcionalidades

- Controle de estoque por loja, com alerta de quantidade mínima
- Cadastro e categorização de produtos (salgados, esfihas, fogazzas, mercado…)
- Pedidos de reposição entre unidades, com acompanhamento de status
- PDV integrado ao estoque
- Histórico de movimentações
- Relatório de pedidos em PDF, por período
- Notificações por email, enviadas de forma assíncrona (Celery + Redis)
- Bot de WhatsApp: consulta de catálogo, abertura de pedido e baixa manual de
  estoque — restrito ao número já cadastrado de cada loja
- Várias empresas no mesmo sistema, cada uma com o próprio catálogo e lojas
- Fábrica própria: fila de pedidos das lojas, registro de produção e etiqueta
  térmica 60×40 com QR por caixa
- Leitura das caixas na loja pelo celular (o site instala como aplicativo):
  cada leitura do QR avança a caixa — chegou, aberta, acabou — e move o estoque
  da fábrica para a loja, com desfazer e a lista do que ainda está a caminho

---

## Arquitetura

```text
                        ┌─────────────────────┐
                        │  Frontend (Next.js)  │
                        │  cookies HTTP-only    │
                        └──────────┬───────────┘
                                   │ /backend/* (proxy same-origin)
                                   ▼
                        ┌─────────────────────┐
                        │  Backend (Django +    │
                        │  DRF, JWT em cookie)   │
                        └───┬─────────┬────────┘
                            │         │
                 ┌──────────┘         └──────────┐
                 ▼                               ▼
        ┌─────────────────┐             ┌──────────────────┐
        │  MySQL 8          │             │  Celery + Redis    │
        │  dados de todas    │             │  email assíncrono,  │
        │  as lojas          │             │  digest diário       │
        └─────────────────┘             └──────────────────┘
                            ▲
                            │ BOT_SERVICE_TOKEN (nao usa JWT)
                            │
                  ┌──────────────────────┐
                  │  Evolution API          │
                  │  (self-hosted, Baileys, │
                  │  nao-oficial)            │
                  └───────────┬────────────┘
                              │
                              ▼
                        WhatsApp da loja
```

O bot de WhatsApp resolve a loja que está agindo **pelo número que mandou a
mensagem**, contra o `telefone_whatsapp` já cadastrado — não existe parâmetro
para uma loja mexer no estoque de outra.

---

## Bot de WhatsApp

O canal de WhatsApp roda sobre a [Evolution API](https://github.com/EvolutionAPI/evolution-api),
uma API REST não-oficial baseada em [Baileys](https://github.com/WhiskeySockets/Baileys)
(WhatsApp Web), auto-hospedada via Docker — sem depender da API oficial paga da
Meta.

| Rota (autenticada por `BOT_SERVICE_TOKEN`) | O que faz |
|---|---|
| `GET /api/v1/bot/catalogo/` | lista os produtos disponíveis |
| `POST /api/v1/bot/pedido/` | abre um pedido pra loja de quem mandou a mensagem |
| `POST /api/v1/bot/pedido/<numero>/confirmar/` | confirma um pedido em aberto |
| `POST /api/v1/bot/estoque/remover/` | dá baixa manual no estoque da própria loja |
| `GET /api/v1/bot/relatorio/` | relatório resumido |

Detalhes de configuração (variáveis de ambiente, subida dos containers
`evolution-api` / `evolution-manager`) estão no README do
[backend](https://github.com/znt10/Unistock_Back).

---

## Como está organizado

Este repositório é o guarda-chuva. O código vive em dois repositórios
independentes, ligados aqui como submódulos:

```text
Unistock/
├── backend/    → znt10/Unistock_Back    Django + DRF + MySQL + Celery + Evolution API
└── frontend/   → znt10/Unistock_Front   Next.js + TypeScript
```

Cada um tem o próprio README, com configuração, endpoints e arquitetura interna.

---

## Quick start

### Pré-requisitos

- **Docker** e **Docker Compose** (backend)
- **Node.js 20+** (frontend)
- **Git**

### Clonar

Com os submódulos — sem a flag, `backend/` e `frontend/` vêm vazias:

```bash
git clone --recurse-submodules https://github.com/znt10/Unistock.git
cd Unistock
```

Se já clonou sem a flag:

```bash
git submodule update --init --recursive
```

### Backend

```bash
cd backend
cp .env.example .env      # preencha antes de subir
docker compose up --build
```

Sobe API, MySQL, Redis, os workers do Celery e o trio da Evolution API (bot de
WhatsApp). A API fica em `http://localhost:8000`, com Swagger em
`/api/schema/swagger/`.

### Frontend

O código do front fica em `frontend/` dentro do próprio repositório do front,
daí o caminho repetido:

```bash
cd frontend/frontend
npm install
npm run dev
```

A interface fica em `http://localhost:3000`.

### Autenticação

O backend emite JWT em **cookies HTTP-only**, e o frontend nunca lê o token —
todas as chamadas passam pelo próprio domínio do front, que reescreve para o
Django. O token fica inacessível ao JavaScript, o que fecha a porta do roubo de
sessão por XSS.

---

## Deploy

Produção roda numa VPS com **Coolify**, em
[unistock.online](https://unistock.online/login). Cada repositório tem o próprio
`docker-compose.prod.yml`, que o Coolify usa como Compose:

| Serviço | Repositório | Compose |
|---|---|---|
| API, MySQL, Redis, Celery | [Unistock_Back](https://github.com/znt10/Unistock_Back) | `docker-compose.prod.yml` |
| Interface (Next.js) | [Unistock_Front](https://github.com/znt10/Unistock_Front) | `frontend/docker-compose.prod.yml` |

O deploy sai do branch `main` de cada repositório. O trabalho do dia a dia vai
para `dev-local` e chega ao `main` por pull request.

---

## Documentação

| Recurso | Link |
|---|---|
| Produção | [unistock.online/login](https://unistock.online/login) |
| Swagger (API) | `http://localhost:8000/api/schema/swagger/` (local) |
| Redoc (API) | `http://localhost:8000/api/schema/redoc/` (local) |
| README do backend | [Unistock_Back](https://github.com/znt10/Unistock_Back) |
| README do frontend | [Unistock_Front](https://github.com/znt10/Unistock_Front) |

---

## Tecnologias

**Backend** — Python 3.12, Django, Django REST Framework, Simple JWT, MySQL 8,
Celery + Redis, drf-spectacular (OpenAPI), Evolution API (bot de WhatsApp),
Docker Compose

**Frontend** — Next.js 16 (App Router), React 19, TypeScript, Tailwind CSS 4,
HeroUI, TanStack Query, Zustand

---

## Para onde vai

O UniStock nasceu para uma rede de lojas de salgados, mas o desenho — estoque
isolado por unidade, pedidos entre unidades, permissão por perfil — não tem nada
de específico desse ramo.

A evolução planejada é trocar o modelo de "loja com estoque" por **"galpão com
estoque"**: cada centro de distribuição gerencia o próprio inventário e os
pedidos fluem entre eles conforme a demanda. É o mesmo arranjo das operações
logísticas grandes, onde cada galpão regional tem estoque próprio.

O bot de WhatsApp já recebe os comandos da loja pelo webhook da Evolution API
(catálogo, pedido, confirmação e baixa de estoque); falta ligar o número
dedicado em produção.

---

## Licença

MIT — veja o arquivo `LICENSE` em cada repositório
([backend](https://github.com/znt10/Unistock_Back/blob/main/LICENSE),
[frontend](https://github.com/znt10/Unistock_Front/blob/main/LICENSE)).

---

<p align="center">
  Feito por <a href="https://github.com/znt10">znt10</a>
</p>
