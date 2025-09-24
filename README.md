# aws-cloudformation-desafio
# Desafio DIO — Infra AWS com CloudFormation

## Objetivo
Implementar infraestrutura automatizada em AWS CloudFormation e documentar o processo para demonstrar entendimento.

## Entregáveis
- Template CloudFormation: `templates/infra.yaml`
- Documentação: este `README.md`
- (Opcional) Prints: `images/`

## O que está no template
O template cria:
- VPC (10.0.0.0/16)
- Subnet pública (10.0.1.0/24)
- Internet Gateway + rota padrão
- Security Group (SSH e HTTP)
- Instância EC2 (Amazon Linux 2)

## Como validar localmente (sem AWS Console)
1. Instale `cfn-lint` e `checkov`:
```bash
python3 -m pip install --user cfn-lint checkov
export PATH="$HOME/.local/bin:$PATH"

