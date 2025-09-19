# AWS-Step-Functions---Orquestracao-de-Workflows-na-AWS

![AWS Step Functions](https://img.shields.io/badge/AWS-Step%20Functions-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-blue.svg?style=for-the-badge)
![Status](https://img.shields.io/badge/status-Em%20Desenvolvimento-green?style=for-the-badge)

## 📋 Sobre o Projeto

Este repositório documenta minha jornada de aprendizado com **AWS Step Functions**, um serviço de orquestração de fluxos de trabalho serverless que permite integrar e automatizar serviços da AWS de forma visual e com pouco código.

### 🎯 Objetivos

- **Explorar** o AWS Step Functions e suas capacidades
- **Integrar** serviços AWS (Lambda, S3, SNS, SQS, DynamoDB)
- **Criar** workflows automatizados para diferentes cenários
- **Aplicar** boas práticas de arquitetura em nuvem
- **Documentar** processos e insights para referência futura

## 🏗️ Estrutura do Projeto

```
aws-step-functions-study/
│
├── README.md                     # Documentação principal
├── docs/                         # Documentação técnica
│   ├── conceitos-fundamentais.md
│   ├── tipos-de-workflow.md
│   ├── integracoes-aws.md
│   └── boas-praticas.md
├── examples/                     # Exemplos práticos
│   ├── basic-workflow/
│   ├── file-processing/
│   ├── error-handling/
│   └── parallel-processing/
├── templates/                    # Templates JSON
│   ├── state-machines/
│   └── cloudformation/
├── images/                       # Capturas de tela e diagramas
└── scripts/                      # Scripts auxiliares
```

## 🚀 O que é AWS Step Functions?

O **AWS Step Functions** é um serviço de orquestração serverless que permite coordenar múltiplos serviços AWS em workflows visuais. Ele usa uma linguagem baseada em JSON chamada **Amazon States Language (ASL)** para definir máquinas de estado.

### ✨ Principais Características

- **Visual Workflow Builder** - Interface gráfica para criar workflows
- **Serverless** - Sem necessidade de gerenciar servidores
- **Integração Nativa** - Conecta diretamente com 220+ serviços AWS
- **Error Handling** - Tratamento robusto de erros e retry
- **Monitoring** - Monitoramento detalhado de execuções

## 📚 Conceitos Fundamentais

### Estados (States)

| Tipo | Descrição | Uso Principal |
|------|-----------|---------------|
| **Task** | Executa uma tarefa específica | Invocar Lambda, enviar SNS |
| **Choice** | Decisão condicional | Lógica if/else |
| **Wait** | Pausa por tempo determinado | Delays, polling |
| **Parallel** | Execução paralela | Processamento simultâneo |
| **Pass** | Passa dados sem execução | Transformação de dados |
| **Fail** | Termina com erro | Tratamento de falhas |
| **Succeed** | Termina com sucesso | Finalizações bem-sucedidas |

### Tipos de Máquinas de Estado

#### Standard Workflows
- **Duração**: Até 1 ano
- **Execuções**: Até 2.000 por segundo
- **Pricing**: Por transição de estado
- **Uso**: Workflows de longa duração

#### Express Workflows
- **Duração**: Até 5 minutos  
- **Execuções**: Mais de 100.000 por segundo
- **Pricing**: Por duração e memória
- **Uso**: Alto volume, baixa latência

## 🛠️ Exemplos Práticos

### 1. Workflow Básico - Hello World

```json
{
  "Comment": "Workflow básico de exemplo",
  "StartAt": "HelloWorld",
  "States": {
    "HelloWorld": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:HelloWorld",
      "End": true
    }
  }
}
```

### 2. Processamento de Arquivos

Este exemplo demonstra um workflow completo para processar arquivos uploadados no S3:

```json
{
  "Comment": "Workflow de processamento de arquivos",
  "StartAt": "ValidateFile",
  "States": {
    "ValidateFile": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:ValidateFile",
      "Next": "FileValid?",
      "Retry": [
        {
          "ErrorEquals": ["States.TaskFailed"],
          "IntervalSeconds": 2,
          "MaxAttempts": 3
        }
      ]
    },
    "FileValid?": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.isValid",
          "BooleanEquals": true,
          "Next": "ProcessFile"
        }
      ],
      "Default": "InvalidFile"
    },
    "ProcessFile": {
      "Type": "Parallel",
      "Branches": [
        {
          "StartAt": "ResizeImage",
          "States": {
            "ResizeImage": {
              "Type": "Task",
              "Resource": "arn:aws:lambda:us-east-1:123456789012:function:ResizeImage",
              "End": true
            }
          }
        },
        {
          "StartAt": "ExtractMetadata",
          "States": {
            "ExtractMetadata": {
              "Type": "Task",
              "Resource": "arn:aws:lambda:us-east-1:123456789012:function:ExtractMetadata",
              "End": true
            }
          }
        }
      ],
      "Next": "SaveToDatabase"
    },
    "SaveToDatabase": {
      "Type": "Task",
      "Resource": "arn:aws:states:::dynamodb:putItem",
      "Parameters": {
        "TableName": "ProcessedFiles",
        "Item": {
          "FileId": {"S.$": "$.fileId"},
          "Status": {"S": "PROCESSED"},
          "ProcessedAt": {"S.$": "$$.State.EnteredTime"}
        }
      },
      "Next": "NotifySuccess"
    },
    "NotifySuccess": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sns:publish",
      "Parameters": {
        "TopicArn": "arn:aws:sns:us-east-1:123456789012:file-processed",
        "Message": "Arquivo processado com sucesso"
      },
      "End": true
    },
    "InvalidFile": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sns:publish",
      "Parameters": {
        "TopicArn": "arn:aws:sns:us-east-1:123456789012:file-error",
        "Message": "Arquivo inválido"
      },
      "End": true
    }
  }
}
```

### 3. Tratamento de Erros e Retry

```json
{
  "Comment": "Workflow com tratamento robusto de erros",
  "StartAt": "ProcessData",
  "States": {
    "ProcessData": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:ProcessData",
      "Retry": [
        {
          "ErrorEquals": ["Lambda.ServiceException", "Lambda.AWSLambdaException"],
          "IntervalSeconds": 2,
          "MaxAttempts": 6,
          "BackoffRate": 2.0
        }
      ],
      "Catch": [
        {
          "ErrorEquals": ["States.TaskFailed"],
          "Next": "HandleError",
          "ResultPath": "$.error"
        }
      ],
      "Next": "Success"
    },
    "HandleError": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:HandleError",
      "End": true
    },
    "Success": {
      "Type": "Succeed"
    }
  }
}
```

## 🔗 Integrações com Serviços AWS

### Lambda Functions
```json
{
  "Type": "Task",
  "Resource": "arn:aws:lambda:region:account:function:FunctionName"
}
```

### DynamoDB
```json
{
  "Type": "Task",
  "Resource": "arn:aws:states:::dynamodb:putItem",
  "Parameters": {
    "TableName": "MyTable",
    "Item": {
      "id": {"S.$": "$.id"}
    }
  }
}
```

### SNS
```json
{
  "Type": "Task",
  "Resource": "arn:aws:states:::sns:publish",
  "Parameters": {
    "TopicArn": "arn:aws:sns:region:account:topic-name",
    "Message.$": "$.message"
  }
}
```

### SQS
```json
{
  "Type": "Task",
  "Resource": "arn:aws:states:::sqs:sendMessage",
  "Parameters": {
    "QueueUrl": "https://sqs.region.amazonaws.com/account/queue-name",
    "MessageBody.$": "$"
  }
}
```

## 📊 Monitoramento e Observabilidade

### CloudWatch Metrics
- **ExecutionsSucceeded** - Execuções bem-sucedidas
- **ExecutionsFailed** - Execuções com falha
- **ExecutionTime** - Tempo de execução
- **ExecutionThrottled** - Execuções limitadas

### X-Ray Tracing
```json
{
  "Type": "Task",
  "Resource": "arn:aws:lambda:us-east-1:123456789012:function:MyFunction",
  "Parameters": {
    "_X_AMZN_TRACE_ID.$": "$._X_AMZN_TRACE_ID"
  }
}
```

## 💰 Considerações de Custo

### Standard Workflows
- **$0.025** por 1.000 transições de estado
- Inclui todos os tipos de estado

### Express Workflows
- **$1.00** por 1 milhão de execuções
- **$0.00001667** por GB-segundo

## 🚨 Boas Práticas

### Design de Workflows
1. **Mantenha estados pequenos** - Cada estado deve ter uma responsabilidade única
2. **Use timeouts** - Evite execuções infinitas
3. **Implemente retry e catch** - Trate erros adequadamente
4. **Monitore execuções** - Use CloudWatch e X-Ray

### Segurança
1. **Princípio do menor privilégio** - IAM roles mínimas necessárias
2. **Criptografia** - Use KMS para dados sensíveis
3. **VPC endpoints** - Para comunicação segura

### Performance
1. **Express workflows** - Para alto volume
2. **Processamento paralelo** - Use estados Parallel
3. **Otimize payloads** - Minimize tamanho dos dados

## 📁 Estrutura de Arquivos

### `/docs/`
Contém documentação técnica detalhada sobre conceitos e práticas.

### `/examples/`
Exemplos práticos organizados por caso de uso:
- Processamento básico
- Workflows paralelos
- Tratamento de erros
- Integrações com serviços

### `/templates/`
Templates prontos para uso em diferentes cenários.

### `/images/`
Diagramas visuais e capturas de tela dos workflows.

## 🎓 Aprendizados e Insights

### O que funcionou bem
- **Interface visual** facilita a compreensão de workflows complexos
- **Integrações nativas** reduzem código boilerplate
- **Monitoramento detalhado** ajuda na depuração

### Desafios encontrados
- **Limitações de payload** (256KB para Standard)
- **Curva de aprendizado** da Amazon States Language
- **Debugging** pode ser complexo em workflows grandes

### Recomendações
1. Comece com workflows simples
2. Use o simulador local para testes
3. Documente bem os estados e transições
4. Monitore custos regularmente

## 🔧 Como usar este repositório

1. **Clone o repositório**
```bash
git clone https://github.com/seu-usuario/aws-step-functions-study.git
cd aws-step-functions-study
```

2. **Explore os exemplos**
```bash
cd examples/basic-workflow
# Cada pasta contém um README específico
```

3. **Deploy usando CloudFormation**
```bash
aws cloudformation create-stack \
  --stack-name step-functions-example \
  --template-body file://templates/cloudformation/basic-workflow.yaml \
  --capabilities CAPABILITY_IAM
```

## 📚 Recursos Adicionais

### Documentação Oficial
- [AWS Step Functions Developer Guide](https://docs.aws.amazon.com/step-functions/)
- [Amazon States Language](https://states-language.net/spec.html)
- [AWS Step Functions Best Practices](https://docs.aws.amazon.com/step-functions/latest/dg/bp-express.html)

### Ferramentas
- [Step Functions Local](https://docs.aws.amazon.com/step-functions/latest/dg/sfn-local.html)
- [AWS Toolkit for VS Code](https://aws.amazon.com/visualstudiocode/)
- [Step Functions Data Flow Simulator](https://docs.aws.amazon.com/step-functions/latest/dg/sfn-simulator.html)

### Cursos e Tutoriais
- AWS Training and Certification
- A Cloud Guru
- Linux Academy

## 🤝 Contribuindo

Este é um projeto de estudo pessoal, mas sugestões são sempre bem-vindas! Sinta-se livre para:

1. Abrir issues para discussões
2. Sugerir melhorias na documentação
3. Compartilhar casos de uso interessantes

## 📝 Licença

Este projeto está licenciado sob a [MIT License](LICENSE).


⭐ Se este repositório foi útil para você, considere dar uma estrela!

