# meuStepFuncion
Criando meu primeiro Step Function 
# Desafio de Orquestração com AWS Step Functions

Este repositório documenta a solução do desafio de orquestração de fluxos de trabalho utilizando **AWS Step Functions**, como parte da formação da DIO.

## 📄 Descrição do Projeto

O objetivo deste projeto é demonstrar a capacidade do AWS Step Functions de orquestrar diferentes serviços da AWS para construir um fluxo de trabalho robusto e escalável. O cenário implementado é um pipeline de processamento de mensagens, onde uma mensagem é recebida, processada e, dependendo do resultado, uma ação subsequente é tomada.

## 🏗️ Arquitetura da Solução

O fluxo de trabalho é orquestrado pelo AWS Step Functions e utiliza os seguintes serviços da AWS:

(images/di<img width="466" height="401" alt="stepfunctions_graph" src="https://github.com/user-attachments/assets/09fad181-04f9-4b9f-b4dd-ea03b8610fa4" />
agrama.png)

1.  **AWS Lambda (`ProcessarMensagemLambda`)**: Uma função Lambda que simula o processamento de uma mensagem. Ela retorna um status de sucesso (código 200) ou falha para o Step Functions.
2.  **AWS Step Functions**: O orquestrador central. Ele invoca a função Lambda de processamento e, com base no status retornado, decide qual o próximo passo.
3.  **Amazon SNS (`NotificacaoSucessoTopic`)**: Um tópico do SNS para onde o Step Functions envia uma notificação se o processamento for bem-sucedido.
4.  **AWS Lambda (`LogarFalhaLambda`)**: Uma segunda função Lambda que é acionada pelo Step Functions em caso de falha no processamento, para logar o erro e permitir a depuração.

## ⚙️ Código do Fluxo de Trabalho (State Machine)

A lógica do fluxo de trabalho é definida em um arquivo JSON utilizando a Amazon States Language (ASL). O código abaixo representa a máquina de estado que orquestra o processo de tomada de decisão.

```json
{
  "Comment": "Um fluxo de trabalho para processar mensagens SQS e notificar sobre o sucesso ou falha.",
  "StartAt": "Processar Mensagem",
  "States": {
    "Processar Mensagem": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGION:ACCOUNT_ID:function:ProcessarMensagemLambda",
      "Next": "Verificar Status de Processamento"
    },
    "Verificar Status de Processamento": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.Payload.statusCode",
          "NumericEquals": 200,
          "Next": "Notificar Sucesso"
        }
      ],
      "Default": "Logar Falha"
    },
    "Notificar Sucesso": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sns:publish",
      "Parameters": {
        "TopicArn": "arn:aws:sns:REGION:ACCOUNT_ID:NotificacaoSucessoTopic",
        "Message": "Processamento de mensagem concluído com sucesso!"
      },
      "End": true
    },
    "Logar Falha": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGION:ACCOUNT_ID:function:LogarFalhaLambda",
      "End": true
    }
  }
}
