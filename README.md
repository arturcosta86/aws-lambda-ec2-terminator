# Automação para Encerrar Instâncias EC2 com Lambda e EventBridge

Este repositório documenta o laboratório "Automatizando o Fim das Instâncias na AWS", realizado na [Escola da Nuvem](https://www.escoladanuvem.org/), que implementa uma solução serverless para encerrar instâncias EC2 automaticamente.

O objetivo é otimizar custos e automatizar a limpeza de ambientes na nuvem, utilizando serviços gerenciados da AWS.

**Instrutor:** Tomas Alric ([@TomasAlric](https://github.com/TomasAlric/TomasAlric))
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

A seguir estão as 7 capturas de tela que comprovam a criação, configuração e o funcionamento de todos os recursos da automação na AWS.

### 1. Política IAM Criada
**Arquivo:** `Print - Evidência Política IAM.jpg`

[cite_start]Esta política (`PoliticaTerminarEC2-ArturCosta`) foi criada para conceder as permissões necessárias para a função Lambda gerenciar instâncias EC2 e escrever logs no CloudWatch.

![Evidência Política IAM](https://github.com/arturcosta86/aws-lambda-ec2-terminator/blob/main/Print%20-%20Evid%C3%AAncia%20Pol%C3%ADtica%20IAM.jpeg)

### 2. Role IAM Criada
**Arquivo:** `Print - Evidência Função IAM.jpeg`

[cite_start]A Role (`RoleTerminarEC2-ArturCosta`) foi criada para o tipo de entidade confiável "Serviço da AWS", permitindo que o serviço Lambda a utilize para executar as ações definidas na política.

![Evidência Role IAM](https://github.com/arturcosta86/aws-lambda-ec2-terminator/blob/main/Print%20-%20Evid%C3%AAncia%20Fun%C3%A7%C3%A3o%20IAM.jpeg)

### 3. Criação da Função Lambda
**Arquivo:** `Print - Evidência Função Lambda 02.jpeg`

[cite_start]Tela inicial da criação da função `LambdaTerminarEC2-ArturCosta`, mostrando que ela foi criada com sucesso, ainda com o código padrão.

![Criação da Função Lambda](https://github.com/arturcosta86/aws-lambda-ec2-terminator/blob/main/Print%20-%20Evid%C3%AAncia%20Fun%C3%A7%C3%A3o%20Lambda%2002.jpeg)

### 4. Código da Função Lambda Implantado
**Arquivo:** `Print - Evidência Função Lambda 01.jpeg`

[cite_start]O script `Terminator.py` após o upload e deploy na função Lambda, contendo a lógica para encerrar as instâncias.

![Código da Função Lambda](https://github.com/arturcosta86/aws-lambda-ec2-terminator/blob/main/Print%20-%20Evid%C3%AAncia%20Fun%C3%A7%C3%A3o%20Lambda%2001.jpeg)

### 5. Configuração do Handler e Timeout
**Arquivo:** `Print - Evidência Função Lambda 03.jpeg`

[cite_start]Ajuste das configurações de tempo de execução: o "Tempo limite" foi alterado para 10 segundos  [cite_start]e o "Manipulador" (Handler) foi modificado para `Terminator.lambda_handler`, garantindo que o Lambda chame a função correta e tenha tempo suficiente para executar.

![Configuração do Handler](https://github.com/arturcosta86/aws-lambda-ec2-terminator/blob/main/Print%20-%20Evid%C3%AAncia%20Fun%C3%A7%C3%A3o%20Lambda%2003.jpeg)

### 6. Gatilho do EventBridge Configurado
**Arquivo:** `Print - Evidência GatilhoTerminarEC2-ArturCosta.jpeg`

[cite_start]Evidência da criação do gatilho do Amazon EventBridge (`GatilhoTerminarEC2-ArturCosta`) [cite: 75][cite_start], que foi adicionado à função Lambda para acioná-la automaticamente em um agendamento definido.

![Evidência Gatilho EventBridge](https://github.com/arturcosta86/aws-lambda-ec2-terminator/blob/main/Print%20-%20Evid%C3%AAncia%20GatilhoTerminarEC2.jpeg)

### 7. Resultado: Instância Encerrada pela Automação
**Arquivo:** `print - Evidência Instância Encerrada com o Gatilho (1).jpeg`

Prova final do funcionamento da solução. [cite_start]Uma instância EC2 de teste (`ec2-teste-gatilhoterminarinstancia-arturcosta`) foi automaticamente movida para o estado "Encerrado" após a execução agendada da função Lambda.

![Evidência Instância Encerrada](https://github.com/arturcosta86/aws-lambda-ec2-terminator/blob/main/Print%20-%20Evid%C3%AAncia%20Inst%C3%A2ncia%20Encerrada%20com%20o%20Gatilho.jpeg)

---

## 🚀 Como Replicar

1.  **Política IAM:** Crie uma política no IAM com as permissões para `ec2:DescribeInstances`, `ec2:TerminateInstances` e `logs:CreateLogGroup`, `logs:CreateLogStream`, `logs:PutLogEvents`.
2.  **Role IAM:** Crie uma role para o serviço Lambda e anexe a política criada.
3.  **Função Lambda:** Crie uma função com runtime Python, associe a role, cole o código do `Terminator.py`, ajuste o timeout para 10 segundos e o handler para `Terminator.lambda_handler`.
4.  **Gatilho EventBridge:** Adicione um gatilho do tipo EventBridge à função Lambda e defina uma regra de agendamento (`rate`).
5.  **Teste:** Crie uma instância EC2 em uma das regiões do script e aguarde o tempo do agendamento para ver a mágica acontecer!
