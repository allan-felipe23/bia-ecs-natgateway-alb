# Plano de Implementação: Aplicação BIA com ECS, RDS, NAT Gateway e ALB

Este documento detalha o passo a passo para construir a arquitetura de alta disponibilidade para a aplicação BIA, operando de forma segura em subnets privadas e conectada a um banco de dados relacional. O plano também indica os momentos exatos onde você deve registrar telas (prints) como prova da realização do desafio.

## Fase 1: Infraestrutura de Rede (VPC, Subnets e Internet Gateway)
Nesta fase, prepararemos o terreno da rede.

1. **Criar a VPC:**
   - Crie uma VPC com um bloco CIDR adequado (ex: `10.0.0.0/16`).
2. **Criar as Subnets:**
   - **Públicas:** Crie duas subnets públicas em Zonas de Disponibilidade (AZs) diferentes (ex: `us-east-1a` e `us-east-1b`).
   - **Privadas:** Crie duas subnets privadas nas mesmas AZs (estas não devem atribuir IP público automaticamente).
3. **Configurar o Internet Gateway (IGW):**
   - Crie um IGW e anexe-o à sua VPC.
4. **Tabela de Roteamento Pública:**
   - Crie/Edite uma Route Table para as subnets públicas.
   - Adicione uma rota para `0.0.0.0/0` apontando para o **Internet Gateway**.
   - Associe esta Route Table às duas subnets públicas.

## Fase 2: Configuração do NAT Gateway (Saída segura para a internet)
Para que as instâncias no ECS consigam baixar pacotes, atualizações e se registrar no cluster sem expor seus IPs, usaremos o NAT Gateway.

1. **Alocar Elastic IP:**
   - Aloque um ou dois endereços Elastic IP na sua conta AWS.
2. **Criar o(s) NAT Gateway(s):**
   - Crie um NAT Gateway e posicione-o em uma das **Subnets Públicas**.
   - Associe o Elastic IP alocado ao NAT Gateway.
3. **Tabela de Roteamento Privada:**
   - Crie uma Route Table para as subnets privadas.
   - Adicione uma rota para `0.0.0.0/0` apontando para o **NAT Gateway**.
   - Associe esta Route Table às duas subnets privadas.

> [!IMPORTANT]
> 📸 **PRINT ESTRATÉGICO 1:** Tire um print da Tabela de Roteamento associada às subnets privadas, mostrando claramente a rota de destino `0.0.0.0/0` apontando para o NAT Gateway. Tire outro print da tela do próprio NAT Gateway mostrando o status como **Available**.

## Fase 3: Segurança (Security Groups)
Vamos isolar o acesso aplicando o princípio do menor privilégio.

1. **Security Group do ALB (SG-ALB):**
   - **Entrada (Inbound):** Permitir HTTP (80) e/ou HTTPS (443) da origem `0.0.0.0/0`.
   - **Saída (Outbound):** Permitir todo o tráfego de saída.
2. **Security Group das Instâncias ECS (SG-ECS):**
   - **Entrada (Inbound):** Permitir tráfego (portas efêmeras ou a da aplicação) **exclusivamente** com origem do **SG-ALB**.
   - **Saída (Outbound):** Permitir todo o tráfego de saída.
3. **Security Group do Banco de Dados RDS (SG-RDS):**
   - **Entrada (Inbound):** Permitir a porta do banco de dados (ex: 3306 para MySQL, 5432 para PostgreSQL) **exclusivamente** com origem do **SG-ECS**.
   - **Saída (Outbound):** Permitir todo o tráfego de saída.

> [!IMPORTANT]
> 📸 **PRINT ESTRATÉGICO 2:** Tire um print das regras "Inbound" do `SG-ECS` provando a permissão do `SG-ALB`. Tire também um print das regras "Inbound" do `SG-RDS` provando a permissão exclusiva para o `SG-ECS`.

## Fase 4: Banco de Dados (RDS)
Vamos provisionar o banco de dados na rede privada.

1. **Subnet Group do RDS:**
   - Acesse o RDS e crie um **Subnet Group**.
   - Selecione sua VPC e as **duas subnets privadas**. Isso garante que o banco não terá acesso direto pela internet.
2. **Criar o Banco de Dados:**
   - Escolha o motor (MySQL, PostgreSQL, etc) e a versão compatível com a sua aplicação.
   - **Connectivity:** Desative o acesso público (Public Access: `No`). Selecione sua VPC, o Subnet Group criado e o Security Group `SG-RDS`.
   - Crie as credenciais (usuário e senha) e anote-as juntamente com o **Endpoint** gerado após a criação, pois eles serão injetados na sua aplicação BIA.

## Fase 5: Imagem Docker (ECR)
Como você tem o código-fonte, precisaremos "conteinerizar" a aplicação.

1. **Criar Repositório no ECR (Elastic Container Registry):**
   - Crie um repositório privado para a sua aplicação (ex: `bia-app-repo`).
2. **Build e Push:**
   - Na sua máquina local (onde está o código), certifique-se de ter um `Dockerfile` configurado.
   - Siga os comandos "View push commands" disponíveis no painel do repositório ECR criado.
   - Basicamente, você vai: fazer login no ECR, realizar o `docker build`, aplicar a tag `docker tag` e enviar a imagem com `docker push`.
   - Copie a **URI da imagem** gerada no ECR, pois você precisará dela na Fase 8.

## Fase 6: Application Load Balancer (ALB)
1. **Criar Target Group:**
   - Tipo de Target: `Instances`.
   - Protocolo/Porta: HTTP na porta mapeada.
   - VPC: Selecione a VPC do projeto.
2. **Criar o Load Balancer:**
   - Tipo: Application Load Balancer (Internet-facing).
   - Rede: Selecione a sua VPC e as **duas subnets públicas**.
   - Security Group: Selecione o `SG-ALB`.
   - Listener: Porta HTTP 80 encaminhando para o Target Group.

> [!IMPORTANT]
> 📸 **PRINT ESTRATÉGICO 3:** Tire um print das configurações do ALB, evidenciando o DNS Name, o estado `Active` e que ele está associado às subnets públicas.

## Fase 7: Cluster ECS e Auto Scaling
1. **Criar Cluster ECS:** Escolha a infraestrutura baseada em Amazon EC2.
2. **Criar Launch Template:**
   - **AMI:** ECS-Optimized Amazon Linux.
   - **Security Group:** `SG-ECS`.
   - **IAM Role:** `ecsInstanceRole` (que permite as instâncias EC2 conversarem com a API do ECS).
   - **User Data:**
     ```bash
     #!/bin/bash
     echo ECS_CLUSTER=NOME-DO-SEU-CLUSTER >> /etc/ecs/ecs.config
     ```
3. **Criar o Auto Scaling Group (ASG):**
   - Selecione o Launch Template.
   - **Network:** VPC do projeto e as **duas subnets privadas**.

> [!IMPORTANT]
> 📸 **PRINT ESTRATÉGICO 4:** Tire um print do painel de instâncias EC2 mostrando as instâncias do ASG rodando nas subnets privadas e confirmando que estão sem IP Público (Public IPv4 vazio).

## Fase 8: Task Definition e ECS Service (Deploy)
Agora vamos juntar a imagem e a conexão com o banco de dados.

1. **Criar Task Definition:**
   - Compatibilidade de lançamento: `EC2`.
   - **Container:**
     - Imagem: A **URI do repositório ECR** que você fez o push na Fase 5.
     - Port Mappings: Porta do host = `0` (dinâmica); Porta do container = porta exposta pela aplicação.
     - **Variáveis de Ambiente (Environment Variables):** Adicione as variáveis que o código-fonte da BIA utiliza para se conectar ao banco de dados. Por exemplo:
       - `DB_HOST`: [Endpoint do seu RDS]
       - `DB_PORT`: [Porta do RDS]
       - `DB_USER`: [Seu usuário]
       - `DB_PASSWORD`: [Sua senha]
       - `DB_NAME`: [Nome do banco de dados]
2. **Criar Service no ECS:**
   - Selecione a Task Definition.
   - Desired Tasks: 2 (para HA).
   - **Load Balancing:** Application Load Balancer. Escolha o Target Group criado na Fase 6. O ECS cuidará do registro.

> [!IMPORTANT]
> 📸 **PRINT ESTRATÉGICO 5:** 
> - Tire um print da aba **Infrastructure** do Cluster ECS mostrando as EC2 registradas (prova de que o NAT Gateway funcionou).
> - Tire um print da aba **Services** ou **Tasks** mostrando que as Tasks estão no estado `RUNNING`.

## Fase 9: Validação e Acesso Final
1. Copie o **DNS Name** do seu Application Load Balancer.
2. Cole-o no navegador. Como as variáveis do banco foram passadas corretamente, a aplicação deve conectar no RDS perfeitamente.

> [!IMPORTANT]
> 📸 **PRINT ESTRATÉGICO FINAL:** Um print do navegador exibindo a aplicação BIA conectada e rodando, evidenciando dados sendo lidos do banco de dados (se houver alguma tela que comprove a conexão) e sendo acessada via o DNS do ALB.
