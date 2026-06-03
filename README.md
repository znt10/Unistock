# Unistock

Unistock é um sistema web de **gerenciamento de estoque para redes de lojas**. O sistema centraliza o controle de produtos, quantidades e pedidos entre diferentes unidades de uma mesma rede, permitindo que gestores acompanhem o estoque de cada loja em tempo real e que responsáveis de unidade façam pedidos de reposição diretamente pelo sistema.

## O que é o estoque no Unistock

No contexto do Unistock, **estoque** representa a quantidade disponível de cada produto em uma loja específica. Cada registro de estoque associa um produto a uma unidade e guarda informações como quantidade atual, quantidade mínima e estado do produto (normal, congelado ou resfriado). Quando a quantidade de um item cai abaixo do mínimo definido, o sistema emite notificações automáticas para os usuários responsáveis.

## Funcionalidades principais

- Controle de estoque por loja, com alertas de quantidade mínima
- Cadastro e categorização de produtos (salgados, esfihas, fogazzas, mercado, entre outros)
- Criação e acompanhamento de pedidos de reposição entre unidades
- PDV (ponto de venda) integrado ao estoque
- Histórico de movimentações
- Gerenciamento de usuários com perfis distintos: Admin, Gerente e Responsável
- Notificações em tempo real via WebSocket

## Tecnologias

**Backend** — Python, Django, Django REST Framework, JWT (autenticação por cookies HTTP-only), MySQL, Docker

**Frontend** — Next.js, React, TypeScript, Tailwind CSS, HeroUI

## Repositórios

| Repositório | Descrição | Link |
|---|---|---|
| Unistock Back | API REST, autenticação, regras de negócio e banco de dados | [github.com/znt10/Unistock_Back](https://github.com/znt10/Unistock_Back) |
| Unistock Front | Interface web da aplicação | [github.com/znt10/Unistock_Front](https://github.com/znt10/Unistock_Front) |
