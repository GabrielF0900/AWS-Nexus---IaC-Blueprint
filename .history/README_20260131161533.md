# 🚀 AWS-Nexus: IaC Blueprint

> **A jornada de aperfeiçoar conhecimentos AWS através da automação de infraestrutura**

---

## 🏗️ Arquitetura Geral

![Arquitetura AWS-Nexus](./AWS-Nexus%20-%20IaC%20Blueprint.drawio)

A arquitetura completa foi desenhada e documentada no arquivo `AWS-Nexus - IaC Blueprint.drawio`. Visualize o diagrama interativo para entender melhor os componentes e suas relações na infraestrutura.

### Componentes Principais:
- **VPC (Virtual Private Cloud)**: Rede isolada com CIDR 10.0.0.0/16
- **Public Subnet**: Sub-rede pública (10.0.1.0/24) para componentes expostos
- **Internet Gateway**: Conectividade com a internet
- **Security Group**: Regras de firewall para HTTP e SSH
- **EC2 Instance**: Servidor t2.micro rodando Apache Web Server
- **CloudWatch**: Monitoramento de métricas (CPUUtilization)
- **SNS**: Notificações de alarmes via email

---

## 📖 A História Por Trás do Projeto

Este é o **terceiro projeto de uma série de 10 projetos com AWS Cloud** que estou realizando para aperfeiçoar meus conhecimentos e me autodesenvolver. Algo que me move. Algo que me inspira.

### O Início: Console da AWS

![Página Inicial AWS](./assets/01-pagina-inicial.jpeg)

Tudo começou na página inicial do console da AWS. Um simples clique, um novo desafio. Mais um projeto na minha lista para dominar a plataforma de cloud da Amazon.

### Primeiro Encontro com CloudFormation

![CloudFormation Inicial](./assets/02-pagina-inicial-cloudfront.jpeg)

E lá estava ele: **CloudFormation**. Mas o que é CloudFormation?

#### O que é CloudFormation?

CloudFormation é um recurso da AWS que cria infraestrutura **baseado em código**. Você se pergunta: *"Como assim baseado em código?"*

Bom, é simples: esse recurso se baseia em um **modelo do tipo YAML ou JSON**. Você escreve o que quer que ele crie — como a arquitetura que desenhamos — e o envia para o CloudFormation. Ele faz um mapeamento e cria **toda a sua infraestrutura automaticamente** a partir do código que você criou.

#### Por quê usar CloudFormation?

Porque um dos **pilares do Well Architected Framework** (que tenho estudado bastante, pensando na certificação AWS Solutions Architect Associate) é que é **mais recomendado e eficiente utilizar serviços de Infrastructure as Code** para criar infraestruturas de maneira automática.

Isso **praticamente anula erros humanos** comparado ao desenvolver a mesma infraestrutura manualmente.

Estamos cumprindo uma das recomendações mais importantes da AWS. ✨

---

## 🛠️ O Início da Configuração

### Escolhendo o Modelo

![Configuração CloudFormation](./assets/03-cloudformation-configuracao.jpeg)

Selecionei a opção "Escolher um modelo existente" (já vem marcada por padrão).

![Primeira Entrada](./assets/04-primeiro-codigo.jpeg)

E criei o código inicial da infraestrutura. Um template básico com VPC, subnet, Internet Gateway e uma instância EC2 com um servidor web.

### Salvando o Modelo YAML

![Upload do Arquivo](./assets/05-arquivo-yaml.jpeg)

Salvei o arquivo como `AwsCloudFormation.yaml`. Este é o modelo que serve como uma "imagem" para o CloudFormation mapear e construir.

### Selecionando o Arquivo

![Escolhendo Arquivo](./assets/06-escolhendo-arquivo.jpeg)

E então, escolhi meu arquivo modelo...

### Nomeando a Pilha

![Nome da Pilha](./assets/07-nome-da-pilha.jpeg)

Coloquei o nome: **"AWS-Nexus-Project"** e segui para o próximo passo.

---

## 🚨 Primeiro Erro: Permissões

### O Desafio das Permissões

![Permissões IAM](./assets/08-permissao-01.jpeg)

CloudFormation solicitava uma **IAM Role** de forma opcional. Como queria evitar qualquer problema relacionado ao **princípio de menor privilégio** (que enfrentei em praticamente todos os projetos da série), decidi lidar com isso desde o início.

![Selecionando LabRole](./assets/09-labrole.jpeg)

Escolhi a IAM Role chamada **"LabRole"**.

### 💥 BOOM: O Primeiro Erro

![Primeiro Erro](./assets/10-primeiro-erro.jpeg)

**Erro de permissões negadas!** O clássico erro de menor privilégio.

#### O que é Menor Privilégio?

É quando você dá permissão para um usuário fazer **apenas o que ele precisa fazer e nada mais**. Você fica limitado. E mesmo usando uma IAM Role já criada para criar infraestrutura via CloudFormation, não foi possível por causa do menor privilégio.

### A Solução: Analisando a Pilha Existente

Mas percebi um detalhe que já havia usado em projetos anteriores:

Quando inicio um laboratório no Sandbox (plataforma da Escola da Nuvem), é criado **tudo o que podemos fazer**, e quando iniciei o Sandbox, foi criada uma pilha automaticamente.

**Qual era minha estratégia?** Analisar a pilha já existente para saber qual **IAM Role foi utilizada para criá-la** e, assim, utilizar essa mesma role para criar minha pilha.

![Analisando Pilha](./assets/11-pilha-existente-metodo-contorno.jpeg)

Entrei na pilha para analisar...

![LabInstanceProfile](./assets/12-lab-instance-profile.jpeg)

E descobri: **LabInstanceProfile**!

#### O que é LabInstanceProfile?

É basicamente um **container que dentro dele fica uma IAM Role**. Quando a EC2 é criada, ela "veste" essa role. Eu poderia usar esse profile nas propriedades da EC2 no meu template, **contornando o problema de permissões!**

---

## ✏️ Primeira Modificação: Corrigindo o Template

Fiz uma modificação no código `AwsCloudFormation.yaml`:

```yaml
# ANTES:
IamInstanceProfile: # não estava definido

# DEPOIS:
IamInstanceProfile: LabInstanceProfile 
```

Também corrigi alguns detalhes:

```yaml
# Tipo de recurso corrigido:
Type: AWS::EC2::VPCGatewayAttachment # Estava incompleto

# Usando parâmetro de AMI:
ImageId: /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-6.1-x86_64
```

![Primeira Modificação](./assets/13-primeira-modificacao.jpeg)

Fiz o upload do modelo atualizado.

![Sem IAM Role](./assets/14-sem-permissao.jpeg)

**Desta vez, não selecionei nenhuma IAM Role** na tela de permissões. Por quê? Porque usamos o **LabInstanceProfile** — um container que contém a IAM Role que a EC2 utilizará.

---

## 🚨 Segundo Erro: Sintaxe YAML

![Segundo Erro](./assets/15-segundo-erro.jpeg)

Erro de sintaxe! Investigando, descobri um simples problema no tipo do recurso:

```yaml
# ANTES:
Type: AWS::EC2::SubnetRouteAssociation # Incorrecto

# DEPOIS:
Type: AWS::EC2::SubnetRouteTableAssociation # Correcto
```

Além disso, para resolver o problema da AMI, implementei uma **solução elegante usando SSM Parameters**:

```yaml
Parameters:
  LatestAmiId:
    Type: 'AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>'
    Default: '/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-6.1-x86_64'

Resources:
  NexusEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      # ...
      ImageId: !Ref LatestAmiId  # Agora usa o parâmetro resolvido dinamicamente
```

---

## ✅ SUCESSO: Infraestrutura Criada!

![CloudFormation Criada](./assets/16-cloudformation-criado.jpeg)

**BOOM!** A pilha foi criada com **TOTAL SUCESSO!**

### Comprovando o Sucesso

A **instância EC2** foi criada com sucesso:

![EC2 Criada](./assets/17-instancia-criada.jpeg)

A **VPC** foi criada com sucesso:

![VPC Criada](./assets/18-vpc-criada.jpeg)

A **Subnet Pública** onde fica a instância foi criada com sucesso:

![Subnet Criada](./assets/19-subnete-criada.jpeg)

**SUCESSO TOTAL!** 🎉

---

## 📊 Implementando Observabilidade: CloudWatch

Após a criação bem-sucedida, fui para a parte de **métricas** — algo muito importante.

Aqui busquei aplicar o **pilar de Excelencia Operacional** do Well Architected Framework: **OBSERVABILIDADE**.

![CloudWatch Métricas](./assets/20-cloudwatch-metricas.jpeg)

Utilizamos o **CloudWatch** para observar a instância EC2, monitorando:
- Uso da CPU
- Logs do sistema

### Criando um Alarme para CPU

![Histórico de Métricas](./assets/21-historico-metricas.jpeg)

Existe uma lista de métricas disponíveis. Escolhi: **CPUUtilization** — para monitorar o uso de CPU.

![CloudWatch Home](./assets/22-pagina-inicial-cloudwatch.jpeg)

Fui para a página de "Todos os Alarmes" onde criamos novos alarmes.

![Selecionando Métrica](./assets/23-criando-metrica-ec2.jpeg)

Selecionei a métrica EC2 → CPUUtilization.

![Métrica CPUUtilization](./assets/24-metrica-cpuUtilization-selecionada.jpeg)

### Configurando a Condição

![Condição CPU](./assets/25-condicao-cpuUtilization.jpeg)

Configurei a condição assim:
- **Tipo**: Estático
- **Condição**: Maior que (>)
- **Valor**: 80%

**O que isso significa?** Sempre que a CPUUtilization for **maior que 80%**, o CloudWatch gerará um log e enviará para o SNS, que notificará via email.

---

## 🔔 Integrando SNS (Simple Notification Service)

### Criando o Tópico SNS

![Configurando SNS](./assets/26-criando-sns.jpeg)

Configurei o SNS:
- **Ação**: Alarme
- **Tipo**: Criar um novo tópico
- **Nome do tópico**: `Nexus-CPU-Alert`
- **Destino**: Meu email

### Confirmando a Inscrição

![Confirmando SNS](./assets/27-confirmando-sns.jpeg)

Uma confirmação foi enviada para meu email, que recebi e confirmei. ✅

### Alarme Criado

![Alarme Criado](./assets/28-sns-metrica-criada.jpeg)

O alarme foi criado com sucesso!

![Métrica OK](./assets/29-metrica-ok.jpeg)

O alarme mostra status **OK**. A partir de agora, **a cada 5 minutos** o CloudWatch receberá informações do uso de CPU da instância.

---

## 🔐 Acessando a Instância: SSH vs SSM Session Manager

### 🚨 Terceiro Erro: Porta SSH Bloqueada

![Terceiro Erro SSH](./assets/30-terceiro-erro.jpeg)

Tentei me conectar via SSH diretamente pelo console... **erro!** A porta SSH estava bloqueada.

### Abrindo SSH (Apenas para Teste)

![Abrindo SSH](./assets/31-abrindo-ssh.jpeg)

Para validar o problema, abri a porta 22:

```yaml
SecurityGroupIngress:
  - IpProtocol: tcp
    FromPort: 22
    ToPort: 22
    CidrIp: 0.0.0.0/0  # ⚠️ NUNCA FAÇA ISSO EM PRODUÇÃO!
```

![Modificação Realizada](./assets/32-modificacao-feita.jpeg)

### Aprendendo a Melhor Prática: SSM Session Manager

Porém, durante minha pesquisa, descobri uma técnica **muito mais segura**: **SSM Session Manager**.

#### O que é SSM Session Manager?

> "Você utiliza o AWS Systems Manager para abrir um túnel seguro de terminal."

**Vantagens:**
- Não precisa gerenciar chaves SSH
- Não precisa abrir portas públicas
- Auditoria nativa de todas as conexões
- Funciona mesmo atrás de NAT/firewalls

![System Manager](./assets/35-system-manager-feito.jpeg)

Depois fechei a porta 22 por segurança e fiz a conexão via SSM Session Manager.

### Atualizando o Template

![Terceira Modificação](./assets/33-terceira-modificacao.jpeg)

Fiz update do template no CloudFormation:

![Update Realizado](./assets/34-atualizacao-feita.jpeg)

O **UPDATE foi feito com sucesso!** Sem precisar recriar a pilha do zero. A beleza do CloudFormation! 🎯

---

## 🔥 Teste de Estresse: Validando o Alarme

### Executando o Teste

Conectei via SSM Session Manager e executei um **teste de estresse** na instância. Rodei um comando de carga intensiva e depois um `top` para monitorar:

```bash
# Teste de estresse para aumentar CPU
yes > /dev/null &

# Monitoramento
top
```

**Resultado?** A CPU chegou a **~90%**!

### O Alarme Acionado

![Em Alarme](./assets/36-em-alarme.jpeg)

Aproximadamente **7 minutos depois**, a métrica no CloudWatch mudou para **ALERTA**, mostrando que o uso de CPU ultrapassou os 80%.

### Email de Notificação

![Email Recebido](./assets/37-email-recebido.jpeg)

**BINGO!** O email de aviso chegou! O SNS funcionou perfeitamente! 📧

O pipeline inteiro funcionou:
1. CloudWatch detectou CPU > 80%
2. Disparou um alarme
3. SNS recebeu a notificação
4. Email foi enviado com sucesso

---

## 🎨 Personalizando a Landing Page

Para finalizar com chave de ouro, personalizei a landing page:

![Página Personalizada](./assets/38-pagina-personalizada.jpeg)

```html
<h1>AWS-Nexus: Landing Page Online em Oregon</h1>
```

---

## 📐 Arquitetura Final

```
┌─────────────────────────────────────────────┐
│         AWS-Nexus Infrastructure             │
├─────────────────────────────────────────────┤
│                  VPC (10.0.0.0/16)           │
│  ┌──────────────────────────────────────┐   │
│  │    Public Subnet (10.0.1.0/24)       │   │
│  │  ┌────────────────────────────────┐  │   │
│  │  │   EC2 Instance (t2.micro)      │  │   │
│  │  │   - Apache Web Server          │  │   │
│  │  │   - CloudWatch Agent           │  │   │
│  │  │   - SSM Session Manager Ready  │  │   │
│  │  └────────────────────────────────┘  │   │
│  └──────────────────────────────────────┘   │
│           │                                   │
│           ↓                                   │
│    Internet Gateway                          │
│           │                                   │
│    Security Group (HTTP, SSH)               │
└─────────────────────────────────────────────┘
           │
           ↓
    ┌──────────────┐
    │  CloudWatch  │ ← Monitora CPU
    │              │
    └──────┬───────┘
           │
           ↓ (se CPU > 80%)
    ┌──────────────┐
    │     SNS      │ ← Envia notificação
    │              │
    └──────┬───────┘
           │
           ↓
      📧 Email Alert
```

---

## 🎯 Principais Aprendizados

1. **CloudFormation é poderoso** - Infrastructure as Code reduz erros e permite versionamento
2. **Menor Privilégio é essencial** - Mesmo em sandboxes, os limites de permissão fazem diferença
3. **Pesquisa e documentação salvam o dia** - Descobrir LabInstanceProfile e SSM foram game-changers
4. **Observabilidade é fundamental** - CloudWatch + SNS = alertas proativos
5. **SSM Session Manager > SSH** - Mais seguro, sem chaves, sem portas abertas

---

## 📁 Estrutura do Projeto

```
AWS-Nexus - IaC Blueprint/
├── README.md                          # Você está aqui!
├── AwsCloudFormation.yaml             # Template CloudFormation
├── AWS-Nexus - IaC Blueprint.drawio   # Diagrama da arquitetura
└── assets/                            # Screenshots da jornada
    ├── 01-pagina-inicial.jpeg
    ├── 02-pagina-inicial-cloudfront.jpeg
    ├── ... (36 imagens documentando o processo)
    └── 38-pagina-personalizada.jpeg
```

---

## 🚀 Como Usar Este Template

1. **Clonar ou copiar o arquivo `AwsCloudFormation.yaml`**
2. **Abrir AWS CloudFormation Console**
3. **Criar nova pilha**
4. **Fazer upload do arquivo YAML**
5. **Nomear a pilha** (ex: AWS-Nexus-Project)
6. **Usar LabInstanceProfile** como IAM Instance Profile
7. **Revisar e criar**

---

## 📚 Próximos Passos

- [ ] Implementar Auto Scaling baseado em CPU
- [ ] Adicionar RDS Database
- [ ] Implementar Load Balancer
- [ ] Adicionar CloudFront CDN
- [ ] Configurar CI/CD Pipeline
- [ ] Melhorar segurança com Bastion Host

---

## � Análise Financeira: Investimento vs. Retorno

Este projeto não é caro para uma empresa — é um **investimento em eficiência e segurança**.

### Comparativo de Custos

| Aspecto | Tradicional (On-premise) | AWS-Nexus (IaC) |
|--------|------------------------|-----------------|
| **Investimento Inicial** | Alto (hardware + infraestrutura física) | **Zero** (pague apenas pelo uso) |
| **Manutenção** | Equipe técnica 24/7 presencial | **Automatizada** via CloudFormation |
| **Segurança** | Complexo e caro | **Nativo** (IAM, SSM, VPC) |
| **Observabilidade** | Softwares caros de terceiros | **Integrado** (CloudWatch gratuito) |

### Onde a Empresa Economiza (ROI)

✅ **Redução de Erros Humanos**: IaC reduz tempo de correção de horas para segundos  
✅ **Continuidade do Negócio**: CloudWatch previne downtime e perda de vendas  
✅ **Segurança Reduzida de Riscos**: SSM elimina exposição SSH, reduzindo risco de vazamentos (multas podem chegar a milhões)  
✅ **Escalabilidade Automática**: Infraestrutura cresce conforme a demanda, não há sobre-investimento inicial  

### Estimativa de Custos Mensais

- **EC2 t2.micro** (gratuito no free tier, depois ~$10/mês)
- **Transferência de Dados** (centavos por GB)
- **CloudWatch + SNS** (praticamente gratuito)
- **Total**: Entre **$0-15/mês** no free tier, escalando conforme tráfego

**Conclusão**: O modelo AWS-Nexus é altamente custo-efetivo e segue os princípios de **Otimização de Custos** do Well Architected Framework.

---

## �💡 Conclusão

Este projeto representa mais que código e infraestrutura. Representa **persistência**, **aprendizado contínuo** e a busca por **excelência operacional**. 

Cada erro encontrado foi uma oportunidade de aprender. Cada solução implementada foi um passo em direção ao domínio da AWS.

A jornada de 10 projetos continua. E estou ansioso para os próximos desafios! 🚀

---

**Criado com 💚 e determinação em desenvolver excelência na Cloud**

*"Seu conhecimento é seu superpoder. Invista nele."*
