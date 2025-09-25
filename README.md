# aws-cloudformation-desafio
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

## Primeiro criamos o arquivo do template que você vai validar com o cfn-lint e checkov.

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

Salve o arquivo (CTRL+O, depois ENTER, depois CTRL+X para sair).

Próximo passo:

Mostrar a saída do arquivo infra.yaml criado.:
```
ls -l ~/aws-cloudformation-desafio/templates
```
## próxima etapa: criar e ativar a venv, depois instalar o cfn-lint e o checkov.

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

No diretório raiz do projeto (~/aws-cloudformation-desafio), rode:
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


