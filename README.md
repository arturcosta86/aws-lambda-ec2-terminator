# Automação para Encerrar Instâncias EC2 com Lambda e EventBridge

Este repositório documenta o laboratório "Automatizando o Fim das Instâncias na AWS", realizado na [Escola da Nuvem](https://www.escoladanuvem.org/), que implementa uma solução serverless para encerrar instâncias EC2 automaticamente.

O objetivo é otimizar custos e automatizar a limpeza de ambientes na nuvem, utilizando serviços gerenciados da AWS.

**Instrutor:** Tomas Alric
**Aluno:** Artur Costa ([@arturcosta86](https://github.com/arturcosta86))

---

## 🎯 Objetivos do Projeto

* **Criar Políticas e Roles no IAM:** Definir permissões específicas e seguras para a função Lambda.
* **Desenvolver uma Função Lambda:** Utilizar Python e a biblioteca Boto3 para criar a lógica de encerramento das instâncias.
* **Agendar com Amazon EventBridge:** Configurar um gatilho (trigger) para executar a função Lambda em intervalos regulares.
* **Validar a Automação:** Testar a solução de ponta a ponta, desde a criação de uma instância até seu encerramento automático.

---

## 🛠️ Arquitetura da Solução

A automação é baseada na seguinte arquitetura de serviços AWS:

1.  **Amazon EventBridge (CloudWatch Events):** Funciona como um cron job na nuvem, acionando a função Lambda com base em uma programação (ex: `rate(12 hours)`).
2.  **AWS Lambda:** Executa o código Python que contém a lógica para encontrar e encerrar as instâncias EC2. É o cérebro da operação.
3.  **AWS IAM (Identity and Access Management):** Concede à função Lambda as permissões necessárias para interagir com o serviço EC2 de forma segura, através de uma IAM Role.
4.  **Amazon EC2:** O serviço onde as instâncias a serem gerenciadas estão localizadas.

---

## 👨‍💻 Código da Função Lambda (`Terminator.py`)

O script em Python abaixo usa a biblioteca `boto3` para interagir com a API da AWS. Ele itera sobre uma lista de regiões, filtra as instâncias que estão ativas ou paradas e executa o comando de encerramento.

```python
import boto3

def terminator():
    regions = ['us-east-1', 'us-east-2', 'us-west-1', 'us-west-2', 'sa-east-1']
    for region in regions:
        print(f"Verificando instâncias na região: {region}")
        ec2 = boto3.resource('ec2', region_name=region)
        instances_to_terminate = ec2.instances.filter(
            Filters=[{'Name': 'instance-state-name', 'Values': ['running', 'stopped']}]
        )
        for instance in instances_to_terminate:
            instance.terminate()
            print(f"Instância {instance.id} encerrada com sucesso na região {region}.")

def lambda_handler(event, context):
    terminator()
```

---

## ✅ Evidências de Execução do Laboratório

A seguir estão as capturas de tela que comprovam a configuração e o funcionamento da automação.

**1. Política IAM Criada (`PoliticaTerminarEC2-ArturCosta`)**
![Evidência Política IAM](Print%20-%20Evidência%20Política%20IAM%20(1).jpg)

**2. Role IAM Criada (`RoleTerminarEC2-ArturCosta`)**
![Evidência Role IAM](Print%20-%20Evidência%20Função%20IAM%20(1).jpeg)

**3. Código da Função Lambda**
![Evidência Código Lambda](Print%20-%20Evidência%20Função%20Lambda%2001%20(1).jpeg)

**4. Configuração do Handler e Timeout**
![Evidência Configuração do Handler](Print%20-%20Evidência%20Função%20Lambda%2003%20(1).jpeg)

**5. Gatilho do EventBridge Configurado**
![Evidência Gatilho EventBridge](Print%20-%20Evidência%20GatilhoTerminarEC2-ArturCosta%20(1).jpeg)

**6. Resultado: Instância Encerrada pela Automação**
![Evidência Instância Encerrada](print%20-%20Evidência%20Instância%20Encerrada%20com%20o%20Gatilho%20(1).jpeg)

---

## 🚀 Como Replicar

1.  **Política IAM:** Crie uma política no IAM com as permissões para `ec2:DescribeInstances`, `ec2:TerminateInstances` e `logs:CreateLogGroup`, `logs:CreateLogStream`, `logs:PutLogEvents`.
2.  **Role IAM:** Crie uma role para o serviço Lambda e anexe a política criada.
3.  **Função Lambda:** Crie uma função com runtime Python, associe a role, cole o código do `Terminator.py`, ajuste o timeout para 10 segundos e o handler para `Terminator.lambda_handler`.
4.  **Gatilho EventBridge:** Adicione um gatilho do tipo EventBridge à função Lambda e defina uma regra de agendamento (`rate`).
5.  **Teste:** Crie uma instância EC2 em uma das regiões do script e aguarde o tempo do agendamento para ver a mágica acontecer!