# Visão Geral

A Fast Engineering S/A enfrenta desafios com sua infraestrutura on-premise devido ao crescimento acelerado do eCommerce, que não suporta mais a alta demanda de acessos e compras. Para modernizar a infraestrutura, garantir escalabilidade, alta disponibilidade e segurança, será realizada uma migração para a Amazon Web Services em duas etapas: uma migração inicial "As-Is" para garantir continuidade operacional, seguida pela modernização da arquitetura utilizando Kubernetes.

## Objetivos

- Migrar a infraestrutura atual para a AWS de forma rápida e segura, utilizando uma abordagem "Lift-and-Shift" (As-Is) para minimizar interrupções.
- Modernizar a arquitetura após a migração inicial, adotando Kubernetes para garantir escalabilidade automática, alta disponibilidade e resiliência.
- Garantir alta disponibilidade e segurança do sistema, atendendo às demandas crescentes do eCommerce.
- Adotar um banco de dados gerenciado (PaaS) com suporte a Multi-AZ (Zonas de Disponibilidade).
- Estabelecer uma política de backup de dados, garantindo a integridade e a recuperabilidade das informações em caso de falhas ou perdas.
- Implementar um sistema de persistência de objetos para armazenamento eficiente de mídias (imagens, vídeos, etc.), com alta disponibilidade e escalabilidade.

## 1. Ordem de Migração Recomendada

- **Banco de Dados**
- **Backend**
- **Frontend**

A ordem de migração entre frontend, backend e banco de dados é essencial para garantir que a aplicação continue funcionando corretamente durante e após a migração. A abordagem recomendada é migrar o banco de dados primeiro, seguido pelo backend e, finalmente, o frontend.

### 1.1. Migração do Banco de Dados

**Por que começar pelo banco de dados?**

O banco de dados é onde todos os dados críticos são armazenados. Migrar o banco de dados primeiro permite que o backend e o frontend continuem funcionando com o banco de dados original durante a migração. Uma vez que o banco de dados esteja migrado e validado, o backend e o frontend podem ser atualizados para apontar para o novo banco de dados na AWS.

### 1.2. Migração do Backend

**Por que migrar o backend em seguida?**

O backend depende do banco de dados para funcionar corretamente. Com o banco de dados já migrado e validado, o backend pode ser migrado com a confiança de que os dados estão disponíveis e acessíveis.

### 1.3. Migração do Frontend

**Por que migrar o frontend por último?**

O frontend depende do backend para funcionar corretamente. Com o backend já migrado e validado, o frontend pode ser migrado com a confiança de que as APIs estão disponíveis e funcionando.

## 2. Componentes da Arquitetura

### 2.1. Frontend (React)

- **Amazon S3**: Armazena os arquivos estáticos do frontend (HTML, CSS, JavaScript).
- **Amazon CloudFront**: Distribui o conteúdo do S3 globalmente, reduzindo a latência e melhorando o desempenho.
- **Amazon Route 53**: Gerencia o DNS do domínio.

### 2.2. Backend (APIs)

- **AWS Application Migration Service (MGN)**: Realiza a migração do backend local para a AWS.
- **Amazon EC2**: Lança Instância EC2 que hospeda as APIs em contêineres Docker.
- **Application Load Balancer (ELB)**: Distribui o tráfego entre as instâncias EC2 do backend.
- **Auto Scaling**: Ajusta o número de instâncias EC2 com base na demanda.

### 2.3. Banco de Dados

- **AWS Database Migration Service**: Migra o banco de dados MySQL local para o Amazon RDS com o mínimo de tempo de inatividade.
- **Amazon RDS**: Banco de dados MySQL em configuração Multi-AZ para alta disponibilidade.

### 2.4. Armazenamento de Objetos

- **Amazon S3**: Armazena arquivos estáticos, como imagens e vídeos.

### 2.5. Segurança

- **Amazon Virtual Private Cloud (VPC)**: Isola os recursos em subnets públicas e privadas.
- **Security Groups**: Controlam o tráfego de rede.
- **AWS Web Application Firewall (WAF)**: Protege as APIs e o CloudFront contra ataques comuns na web.
- **AWS Shield**: Protege contra ataques DDoS.

### 2.6. Monitoramento

- **Amazon CloudWatch**: Monitora métricas (CPU, memória, tráfego de rede) e coleta logs.

## 3. Diagrama de Arquitetura

![image](https://github.com/user-attachments/assets/280ccda6-c5ef-4a94-9099-19cdbdd267d3)


## 4. Etapas de Migração

### 4.1. Migração do Banco de Dados com AWS Database Migration Service (DMS)

1. Criar uma instância do Amazon RDS com o MySQL:
    - Instance Class: db.m5.large (2 vCPUs, 8 GB de RAM).
    - Multi-AZ: Habilitar para alta disponibilidade.
    - 500 GB com provisionamento de IOPS.
    - Habilitar backups automáticos.

2. Criar uma Instância de Replicação do DMS:
    - Instance Class: dms.t3.medium (2 vCPUs, 4 GB de RAM).
    - VPC: Associar à VPC onde o RDS está localizado.
    - Subnet Group

3. Configurar dois Endpoints:
    - Endpoint de origem: especificar as configurações do banco de dados local.
    - Endpoint de destino: especificar as configurações da instância MySQL criada no RDS.

4. Criar e Executar uma Tarefa de Migração:
    - Criar uma tarefa de migração no DMS utilizando os endpoints criados anteriormente.
    - Usar Change Data Capture (CDC) para replicar alterações em tempo real durante a migração.

### 4.2. Migração do Servidor Backend com AWS Application Migration Service (MGN)

1. Instalar o AWS Replication Agent:
    - O AWS Replication Agent deve ser instalado nos servidores que hospedam o backend (APIs + Nginx).

2. Configurar o AWS MGN:
    - Criar um Source Server para o servidor backend.
    - O AWS MGN detecta automaticamente os servidores com o agente instalado.

3. Configurar as Launch Settings:
    - Instance Type: t3.medium (tipo de instância EC2).
    - Subnet: Associe a uma subnet na VPC onde o RDS e outros recursos estão localizados.

4. Teste de Migração:
    - Validar os Servidores de Teste.
    - Testar a integração com o banco de dados RDS.

5. Execução do Cutover:
    - Após validar os servidores de teste, executar o Cutover para migrar os servidores de produção para a AWS.

6. Configurar o Application Load Balancer (ALB):
    - Criar um ALB para distribuir o tráfego entre as instâncias EC2 escaláveis.

7. Implementar Auto Scaling Group:
    - Criar um Template com a mesma configuração da instância criada na migração com o serviço AWS MGN.

### 4.3. Frontend (React) com CloudFront, Bucket S3 e Route 53

1. **Amazon S3**:
    - Criar um bucket S3: o CloudFront precisa de uma origem para distribuir o conteúdo. No caso do frontend, a origem será um bucket no Amazon S3 que armazena os arquivos estáticos (HTML, CSS, JavaScript).

2. **Amazon CloudFront**:
    - Criar uma distribuição CloudFront com o bucket S3 como origem.
    - Configurar o redirecionamento HTTP para HTTPS para garantir que os usuários acessem o site apenas por HTTPS.

3. **Amazon Route 53**:
    - Criar um registro DNS do tipo IPv4 e distribuição do CloudFront.

## Arquitetura da Modernização

### 1. Ferramentas utilizadas

- **Dockerfile**: Documento de texto que contém todos os comandos para montar uma imagem.
- **Amazon Elastic Container Registry (ECR)**: Registro de contêiner totalmente gerenciado.
- **Amazon Elastic Kubernetes Service (EKS)**: Serviço totalmente gerenciado do Kubernetes.
- **Nginx Ingress Controller**: Implementação do Ingress Controller para NGINX.
- **AWS Load Balancer Controller (LBC)**: Provisiona balanceadores de carga da AWS.
- **Horizontal Pod Autoscaler (HPA)**: Escala automaticamente o número de pods em uma implantação.
- **Cluster Autoscaler**: Garantia de nós suficientes para programar os pods.
- **Amazon CloudWatch**: Monitora aplicações e desempenho.
- **AWS Identity and Access Management (IAM)**: Controle de acesso aos serviços e recursos da AWS.
- **Kubernetes Role-based Access Control (RBAC)**: Regula o acesso com base nas funções de usuários.
- **AWS WAF**: Criação de regras de segurança para proteger contra bots e ataques comuns.
- **Velero**: Backup e restauração de recursos do Kubernetes.

### 2. Diagrama da Arquitetura

### 3. Etapas de Modernização

1. **Enviar as APIs contêinerizadas para o Amazon Elastic Container Registry (ECR) em Dockerfile**.
2. **Criar um cluster no AWS EKS**:
    - Configurar Node Groups.
    - Configurar o Ingress Controller (Nginx) no cluster.
3. **Migração das APIs para o EKS**:
    - Criar Deployments e Services para expor as APIs.
    - Configurar Horizontal Pod Autoscaler (HPA).
4. **Auto Scaling Group (ASG)**:
    - Configurar um Auto Scaling Group para garantir que os pods estejam distribuídos.
5. **AWS CloudWatch**:
    - Configurar métricas e alarmes para monitorar o desempenho da aplicação.

---

A migração para a AWS e a modernização com Kubernetes vão permitir à Fast Engineering S/A escalabilidade e flexibilidade para crescer junto com a demanda, além de melhorar a performance e a segurança.
