<div align="center">

# UniStock

**Controle de estoque para redes com várias lojas.**

Cada unidade enxerga o próprio estoque. O gerente enxerga todas.
Quando um produto cruza o mínimo, o responsável recebe um aviso antes de faltar.

[![Demo](https://img.shields.io/badge/demo-online-2ea44f?style=flat-square)](https://5-periodo.vercel.app/)
[![Backend](https://img.shields.io/badge/backend-Django%20%2B%20DRF-092E20?style=flat-square&logo=django)](https://github.com/znt10/Unistock_Back)
[![Frontend](https://img.shields.io/badge/frontend-Next.js%2016-000000?style=flat-square&logo=nextdotjs)](https://github.com/znt10/Unistock_Front)
[![License](https://img.shields.io/badge/licen%C3%A7a-MIT-blue?style=flat-square)](#licença)

[**Ver demo**](https://5-periodo.vercel.app/) · [Backend](https://github.com/znt10/Unistock_Back) · [Frontend](https://github.com/znt10/Unistock_Front)

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
| 👤 **Acesso por perfil** | Admin, Gerente e Responsável enxergam coisas diferentes — e isso vale na API, não só na tela |

**Stack:** Django + DRF + MySQL + Celery no backend, Next.js 16 + TypeScript no
frontend, tudo em Docker.

---

## O que é o UniStock

O UniStock é um sistema web de gerenciamento de estoque pensado para **redes**,
não para uma loja só. A diferença importa: o problema não é saber quanto tem, é
saber quanto tem **onde**, e conseguir mover produto entre unidades sem telefone
e caderno.

O sistema centraliza produtos, quantidades e pedidos entre as unidades da mesma
rede. O gerente acompanha o estoque de cada loja; o responsável de unidade cuida
da sua e pede reposição direto pelo sistema.

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
- Bot de WhatsApp para consulta de catálogo e abertura de pedido

---

## Como está organizado

Este repositório é o guarda-chuva. O código vive em dois repositórios
independentes, ligados aqui como submódulos:

```text
Unistock/
├── backend/    → znt10/Unistock_Back    Django + DRF + MySQL + Celery
└── frontend/   → znt10/Unistock_Front   Next.js + TypeScript
```

Cada um tem o próprio README, com configuração, endpoints e arquitetura interna.

### Rodando o projeto

Clone **com os submódulos** — sem a flag, as pastas `backend/` e `frontend/`
vêm vazias:

```bash
git clone --recurse-submodules https://github.com/znt10/Unistock.git
cd Unistock
```

Se já clonou sem a flag:

```bash
git submodule update --init --recursive
```

**Backend** (precisa de Docker):

```bash
cd backend
cp .env.example .env      # preencha antes de subir
docker compose up --build
```

Sobe API, MySQL, Redis e os workers do Celery. A API fica em
`http://localhost:8000`, com Swagger em `/api/schema/swagger/`.

**Frontend** (o código fica em `frontend/` dentro do próprio repositório do
front, daí o caminho repetido):

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

## Tecnologias

**Backend** — Python 3.12, Django, Django REST Framework, Simple JWT, MySQL 8,
Celery + Redis, drf-spectacular (OpenAPI), Docker Compose

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

Também no plano: notificação por WhatsApp. A camada de canais no backend já foi
escrita para receber esse plugue — hoje só o canal de email está implementado.

---

## Licença

MIT — veja o arquivo `LICENSE` em cada repositório
([backend](https://github.com/znt10/Unistock_Back/blob/main/LICENSE),
[frontend](https://github.com/znt10/Unistock_Front/blob/main/LICENSE)).
