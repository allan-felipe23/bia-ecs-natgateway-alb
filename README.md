# Projeto BIA

![Logo do Projeto](./docs/prints/logo-placeholder.png) <!-- Adicione aqui o link para a logo do projeto -->

## Visão Geral do Projeto
**Nome:** BIA  
**Versão:** 4.2.0  
**Autor:** Henrylle Maia (@henryllemaia)  
**Evento Relacionado:** Imersão AWS & IA (21/03 a 22/03/2026 - Online e ao Vivo)  
**Inscrição do Evento:** [Página de Inscrição](https://org.imersaoaws.com.br/github/readme)

O projeto BIA é uma aplicação educacional criada para servir de base em eventos de tecnologia e treinamentos como a Formação AWS. Ele foi concebido em 2021 e tem evoluído constantemente para adotar as melhores práticas de desenvolvimento e arquitetura Cloud. O objetivo principal é fornecer uma infraestrutura de código onde o aluno pode progredir do básico a cenários complexos de deploy e nuvem.

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
   Aguarde os logs indicarem que o Vite e a API subiram e acesse as portas mapeadas (geralmente `http://localhost:3000` para o Frontend e `http://localhost:3001` para o Backend).

---

## ☁️ Arquitetura AWS e Deploy (Alta Disponibilidade)

Este projeto foi desenhado para ser implementado na AWS utilizando uma arquitetura robusta, segura e de alta disponibilidade. O deploy completo segue um plano de implementação de 9 fases:

1. **Infraestrutura de Rede (VPC):** Criação de VPC com Subnets Públicas e Privadas em múltiplas zonas de disponibilidade, além de Internet Gateway.
2. **Saída Segura (NAT Gateway):** Configuração de NAT Gateways nas subnets públicas para permitir que as instâncias privadas acessem a internet com segurança.
3. **Segurança (Security Groups):** Isolamento de tráfego (Menor Privilégio) entre Load Balancer, Instâncias ECS e o Banco de Dados.
4. **Banco de Dados (RDS):** Provisionamento do PostgreSQL em Subnet Group privado, acessível apenas pelas instâncias da aplicação.
5. **Container Registry (ECR):** Criação de repositório privado, build da imagem Docker e push para a AWS.
6. **Balanceamento de Carga (ALB):** Configuração de um Application Load Balancer internet-facing distribuindo tráfego para as instâncias.
7. **Cluster e Auto Scaling:** Criação de Cluster ECS utilizando EC2 (ECS-Optimized) dentro de um Auto Scaling Group (ASG) nas subnets privadas.
8. **Task Definition e Service:** Definição da tarefa com a imagem do ECR, variáveis de ambiente do RDS e implantação do Service no ECS acoplado ao ALB.
9. **Validação:** Teste de acesso final utilizando o DNS Name do Load Balancer.

Para consultar o documento completo e detalhado do passo a passo, leia o [Plano de Projeto ECS BIA](./docs/plano_projeto_ecs_bia.md).

---

## 📸 Evidências da Implementação (Screenshots)

Abaixo estão os espaços para documentar e provar a execução bem-sucedida das fases de implementação na AWS e da aplicação funcionando:

### 1. Aplicação Funcionando
![Tela Principal da BIA](./docs/prints/home-placeholder.png) <!-- Substitua pelo print da aplicação -->

### 2. Tabela de Roteamento (Subnets Privadas e NAT Gateway)
![Tabela de Roteamento](./docs/prints/route-table-placeholder.png) <!-- Substitua pelo print da Route Table apontando para o NAT Gateway -->

### 3. NAT Gateway Ativo
![NAT Gateway](./docs/prints/nat-gateway-placeholder.png) <!-- Substitua pelo print do NAT Gateway com status Available -->

### 4. Security Groups (Regras Inbound)
![Security Group ECS](./docs/prints/sg-ecs-placeholder.png) <!-- Print provando a permissão do SG-ALB no SG-ECS -->
![Security Group RDS](./docs/prints/sg-rds-placeholder.png) <!-- Print provando a permissão do SG-ECS no SG-RDS -->

### 5. Application Load Balancer (ALB)
![Application Load Balancer](./docs/prints/alb-placeholder.png) <!-- Print do ALB (DNS Name, estado Active e subnets públicas) -->

### 6. Instâncias EC2 (Auto Scaling Group)
![Instâncias EC2 Privadas](./docs/prints/ec2-private-placeholder.png) <!-- Print mostrando as instâncias rodando sem IP Público -->

### 7. Cluster ECS (Infrastructure e Tasks)
![Cluster ECS - Infraestrutura](./docs/prints/ecs-infra-placeholder.png) <!-- Print da aba Infrastructure do ECS (EC2 registradas) -->
![Cluster ECS - Tasks RUNNING](./docs/prints/ecs-tasks-placeholder.png) <!-- Print da aba Tasks com estado RUNNING -->

### 8. Acesso Final via DNS do Load Balancer
![Acesso Final ALB](./docs/prints/final-access-placeholder.png) <!-- Print do navegador acessando a aplicação via DNS do ALB conectada ao banco -->
