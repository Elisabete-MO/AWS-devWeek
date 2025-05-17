# ☁️ Sistema Serverless de Processamento de Pedidos - AWS

Este projeto implementa uma arquitetura 100% serverless na AWS para processar pedidos de forma **escalável, eficiente** e com **baixo custo**. O sistema suporta dois tipos de entrada:

### 🔄 Entradas do sistema:

- **API REST** (tempo real): recebe pedidos de usuários, faz pré-validação, enfileira e processa com validação profunda.
- **Upload S3** (lote): permite envio de arquivos JSON com vários pedidos. Os dados passam por validação, registro de histórico e seguem o mesmo fluxo de pedidos do tempo real.

---

## 🧩 Diagrama da Arquitetura

![Arquitetura AWS](docs/arquitetura.png)

---

## 🔧 Tecnologias e Serviços AWS

- **Amazon API Gateway (REST)**
- **AWS Lambda** (pré-validação, validação profunda, processamento)
- **Amazon SQS FIFO** + DLQs (fila principal e controle de erros)
- **Amazon S3** (entrada de pedidos em lote)
- **Amazon EventBridge** (Custom Event Bus)
- **Amazon DynamoDB** (armazenamento principal e histórico)
- **Amazon SNS** (notificações de erro)
- **AWS CloudFormation** (infraestrutura como código)
- **Python** para funções Lambda
- **Git + GitHub** para versionamento e controle

---

## 📥 Fluxo do Sistema

### Entrada 1: API REST (tempo real)
1. Usuário envia um pedido via **API Gateway**
2. Lambda de **pré-validação** verifica estrutura e dados básicos
3. Pedido é enfileirado em **SQS FIFO**
4. Lambda de **validação profunda** consome a fila
5. Se válido, o evento é publicado no **EventBridge**
6. EventBridge direciona o evento para filas específicas (alteração, cancelamento, etc.)
7. Outras Lambdas processam e armazenam no **DynamoDB**

### Entrada 2: Arquivo JSON no S3 (lote)
1. Arquivo JSON enviado para o **bucket S3**
2. Evento do S3 dispara Lambda de **validação de lote**
3. A Lambda:
   - Valida os dados
   - Armazena histórico no **DynamoDB Histórico**
   - Publica no **SNS** em caso de erro
   - Enfileira pedidos válidos na mesma **SQS FIFO** (fila principal)

---

## 📁 Estrutura do Repositório

```plaintext
infra/stacks/        # Templates CloudFormation separados por serviço ou função
infra/templates/     # Arquivos com parâmetros padrão ou snippets reutilizáveis
lambdas/             # Código das funções Lambda (Python)
docs/                # Diagramas e documentação visual
scripts/             # Scripts auxiliares (deploy/testes)
README.md            #  Este arquivo
.gitignore           #  Ignora arquivos desnecessários
```

---

## 📦 Como rodar
```bash
# Exemplo (se usar a CLI):
aws cloudformation deploy \
  --template-file infra/stacks/sqs-pedidos.yaml \
  --stack-name fila-pedidos \
  --capabilities CAPABILITY_NAMED_IAM
```

---

## 🎯 Objetivos do Projeto
✅ Garantir ordenação dos pedidos (SQS FIFO)<br>
✅ Processar pedidos de forma assíncrona e escalável<br>
✅ Separar responsabilidades entre validação, processamento e armazenamento em funções simples e independentes<br>
✅ Permitir entrada de dados em tempo real(API) e em lote(JSON)<br>
✅ Processar eventos com resiliência(SQS DLQ) e notificar erros com SNS<br>
✅ Registrar histórico de pedidos processados<br>
✅ Reduzir custos usando arquitetura serverless<br>

---

## 🤝 Como contribuir
Este projeto foi criado com fins educacionais, mas pode ser expandido ou adaptado. Para sugerir melhorias:
1. Crie uma issue explicando sua sugestão
2. Faça um fork do repositório
3. Crie uma branch com sua feature ou correção
4. Envie um pull request

---

## ⚖️ Licença
Este projeto é disponibilizado apenas para fins educacionais e não deve ser utilizado em produção sem revisão completa.

---

## 🏫 Créditos

Este projeto foi desenvolvido como parte da **Semana do Desenvolvedor** promovida pela [Escola da Nuvem](https://escoladanuvem.org), que detém os direitos autorais sobre o conteúdo técnico e pedagógico utilizado como base para a arquitetura e o fluxo do sistema.

Organização, documentação e estrutura de repositório adaptadas por:<br>
[<img loading="lazy" src="https://avatars.githubusercontent.com/Elisabete-MO?v=4" width=115><br><sub>Elisabete Oliveira</sub>](https://github.com/Elisabete-MO)

---

