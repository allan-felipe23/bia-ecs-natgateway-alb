# Projeto BIA

![Logo do Projeto](./docs/prints/logo-placeholder.png) <!-- Adicione aqui o link para a logo do projeto -->

## Visão Geral do Projeto
**Nome:** BIA  
**Versão:** 4.2.0  
**Autor:** Henrylle Maia (@henryllemaia)  
**Evento Relacionado:** Imersão AWS & IA (21/03 a 22/03/2026 - Online e ao Vivo)  
**Inscrição do Evento:** [Página de Inscrição](https://org.imersaoaws.com.br/github/readme)

O projeto BIA é uma aplicação educacional criada para servir de base em eventos de tecnologia e treinamentos como a Formação AWS. Ele foi concebido em 2021 e tem evoluído constantemente para adotar as melhores práticas de desenvolvimento e arquitetura Cloud. O objetivo principal é fornecer uma infraestrutura de código onde o aluno pode progredir do básico a cenários complexos de deploy e nuvem.

## 📸 Screenshots

Aqui estão algumas capturas de tela do sistema em funcionamento:

### Tela de Login/Home
![Tela Principal](./docs/prints/home-placeholder.png) <!-- Substitua pelo caminho do print real -->

### Dashboard de Tarefas
![Dashboard de Tarefas](./docs/prints/dashboard-placeholder.png) <!-- Substitua pelo caminho do print real -->

---

## 🛠 Stack Tecnológica

A BIA é construída sob uma arquitetura moderna dividida entre Frontend e Backend:

### Frontend
- **Framework:** React 17
- **Bundler:** Vite
- **Roteamento:** React Router DOM
- **Ícones:** React Icons

### Backend
- **Ambiente:** Node.js
- **Framework Web:** Express.js
- **Integração Cloud:** AWS SDK (Secrets Manager, STS)
- **Segurança e Sessões:** CORS, Express Session
- **Logs:** Morgan

### Banco de Dados e ORM
- **SGBD:** PostgreSQL (driver `pg`)
- **ORM:** Sequelize

### Infraestrutura e Deploy
- **Containers:** Docker e Docker Compose

---

## 🚀 Como Executar o Projeto Localmente

O projeto já conta com o arquivo `compose.yml` para facilitar a subida do ambiente.

1. **Subir os containers (Banco de Dados, Backend e Frontend):**
   ```bash
   docker compose up -d
   ```

2. **Rodar as migrations no banco de dados:**
   Após os containers subirem, você deve criar as tabelas no PostgreSQL executando o comando abaixo:
   ```bash
   docker compose exec server bash -c 'npx sequelize db:migrate'
   ```

3. **Acesso:**
   Aguarde os logs indicarem que o Vite e a API subiram e acesse as portas mapeadas (geralmente `http://localhost:3000` para o Frontend e `http://localhost:3001` para o Backend, dependendo da sua configuração de ambiente).

## 📝 Documentação da Arquitetura AWS
Para detalhes de como esse projeto deve ser hospedado na AWS (arquitetura ECS), consulte o [Plano de Projeto ECS BIA](./docs/plano_projeto_ecs_bia.md) na pasta `docs/`.
