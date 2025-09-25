# AWS-cloudformation-desafio
# Desafio DIO — Infra AWS com CloudFormation

## Objetivo
Implementar infraestrutura automatizada em AWS CloudFormation e documentar o processo para demonstrar entendimento.

## Entregáveis
- Template CloudFormation: `templates/infra.yaml`
- Documentação: este `README.md`
- Prints: `images/`

## O que está no template
O template cria:
- VPC (10.0.0.0/16)
- Subnet pública (10.0.1.0/24)
- Internet Gateway + rota padrão
- Security Group (SSH e HTTP)
- Instância EC2 (Amazon Linux 2)

## Primeiro criamos o arquivo do template que você vai validar com o cfn-lint e checkov
***O Checkov é uma ferramenta de código aberto para análise estática de código-fonte que verifica a infraestrutura como código (IaC) em busca de configurações incorretas, vulnerabilidades de segurança e não conformidade com padrões regulatórios e melhores práticas. Ele suporta uma ampla gama de plataformas IaC, como Terraform, CloudFormation, Kubernetes e Dockerfile, e é integrado a pipelines de CI/CD para detetar problemas antes da implementação.***

***AWS CloudFormation O Linter (cfn-lint) é uma ferramenta de código aberto que você pode usar para realizar uma validação detalhada em seus modelos.***

Passo 1 — Criar estrutura de pastas

No WSL:
```
mkdir -p ~/aws-cloudformation-desafio/templates
cd ~/aws-cloudformation-desafio/templates
```
Passo 2 — Criar arquivo infra.yaml

```
nano infra.yaml
```

Template:
```
AWSTemplateFormatVersion: '2010-09-09'
Description: Exemplo simples — VPC + Subnet + Security Group + EC2

Parameters:
  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: "Nome do key pair existente (necessário para SSH)"
  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues: [t2.micro, t3.micro]
    Description: "Tipo da instância EC2"

Resources:
  MinhaVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: dio-desafio-vpc

  MinhaSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MinhaVPC
      CidrBlock: 10.0.1.0/24
      MapPublicIpOnLaunch: true
      AvailabilityZone: !Select [0, !GetAZs '']
      Tags:
        - Key: Name
          Value: dio-desafio-subnet

  MeuSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: "Allow SSH"
      VpcId: !Ref MinhaVPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0 # usei meu IP real
          Description: "Permite SSH apenas para administração"
      Tags:
        - Key: Name
          Value: dio-desafio-sg

  MinhaInstancia:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      ImageId: ami-0c02fb55956c7d316 # Amazon Linux 2 (us-east-1, fixo só p/ teste de lint)
      KeyName: !Ref KeyName
      SubnetId: !Ref MinhaSubnet
      SecurityGroupIds:
        - !Ref MeuSecurityGroup
      Tags:
        - Key: Name
          Value: dio-desafio-instance
```

Salvar o arquivo (CTRL+O, depois ENTER, depois CTRL+X para sair).

Próximo passo:

Mostrar a saída do arquivo infra.yaml criado.:
```
ls -l ~/aws-cloudformation-desafio/templates
```
## Próxima etapa: criar e ativar a venv, depois instalar o cfn-lint e o checkov.

Passo 1 — Criar ambiente virtual
No diretório do projeto (raiz, não dentro de templates/):
```
cd ~/aws-cloudformation-desafio
python3 -m venv .venv
```
Passo 2 — Ativar ambiente
```
source .venv/bin/activate
```
No prompt vai aparecer (.venv - pasta especial criada dentro do seu projeto para isolar as dependências Python) no começo → isso indica que a venv está ativa.
Passo 3 — Instalar pacotes dentro da venv
```
pip install --upgrade pip
pip install cfn-lint checkov
```
Passo 4 — Validar se instalou
```
cfn-lint --version
checkov --version
```
## Rodar as validações do ´infra.yam´.
Passo 1 — Validar sintaxe com cfn-lint

No diretório raiz do projeto (~/aws-cloudformation-desafio), rodar:
```
cfn-lint templates/infra.yaml
```
Verifica se o template CloudFormation está correto na sintaxe e na estrutura (ex.: se propriedades estão no lugar certo, tipos válidos, etc.).
Passo 2 — Analisar segurança com checkov
```
checkov -f templates/infra.yaml
```
<img width="1360" height="768" alt="aws-cloudformation-desafio" src="https://github.com/user-attachments/assets/27fe776b-2124-4a79-9914-cc950ba011e2" />
<img width="1360" height="768" alt="checov2" src="https://github.com/user-attachments/assets/40b3cb1a-4f5b-497a-abcc-5d05f01d4273" />

# Pré-requisitos (no WSL / Ubuntu)
Atualizar o apt e instalar ferramentas básicas:
```
sudo apt update
sudo apt install -y git python3-pip zip unzip
```

Instalei git e Python/pip que usaremos para ferramentas locais.

Instalar ferramentas de validação/lint local (nesse momento não usarei AWS):

```
python3 -m pip install --user cfn-lint checkov
```
garantir ~/.local/bin no PATH (adicionar no .bashrc se necessário)

```
export PATH="$HOME/.local/bin:$PATH"
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
```
Por que o cfn-lint valida a sintaxe e boas práticas de CloudFormation localmente; checkov faz análises de segurança/infra-as-code.

Nota: aws cloudformation validate-template usa a API da AWS — exige credenciais e rede. Enquanto não tenho acesso, usei cfn-lint + checkov.

---
<img width="1360" height="768" alt="version" src="https://github.com/user-attachments/assets/995b7bd8-6d5d-4302-ab7e-dc04c02a83a9" />


---

# Automação da Validação com GitHub Actions
Para garantir que o template esteja sempre em conformidade com as boas práticas de segurança e sintaxe, foi configurada uma pipeline de Integração Contínua (CI) utilizando o GitHub Actions.

Essa automação é acionada automaticamente a cada push ou pull request para a branch principal (main), executando as seguintes ferramentas:

* cfn-lint: Valida a sintaxe e a estrutura do template CloudFormation.

* Checkov: Analisa o código em busca de vulnerabilidades de segurança e não conformidades.

Caso qualquer uma das ferramentas encontre um erro, a pipeline falhará, impedindo que o código vulnerável seja integrado.

## Como a Automação Foi Configurada
* Criação do Workflow: Um arquivo de configuração YAML (.github/workflows/validate-cloudformation.yaml) foi criado na raiz do repositório.

### Pasta do GitHub Actions
Criei o diretório .github/workflows na raiz do meu projeto.

```Bash
mkdir -p .github/workflows
```
Criei o Arquivo da Pipeline
```
touch .github/workflows/validate-cloudformation.yaml
nano .github/workflows/validate-cloudformation.yaml
```
```yaml
name: CloudFormation Validation

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  validate-template:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.x'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install cfn-lint checkov

      - name: Run cfn-lint
        run: cfn-lint templates/infra.yaml

      - name: Run Checkov
        run: checkov -f templates/infra.yaml
```

* Configuração da Pipeline: O arquivo validate-cloudformation.yaml define os passos da automação, incluindo: A execução em um ambiente Ubuntu. A instalação das dependências cfn-lint e checkov. A execução dos comandos de validação em sequência.

<img width="1355" height="467" alt="github-action-cloudformation" src="https://github.com/user-attachments/assets/531daf7a-adaf-49ea-b090-35eca8f0468b" />


