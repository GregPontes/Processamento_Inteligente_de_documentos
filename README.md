# Processamento Inteligente de Documentos

Solução baseada em **AWS Step Functions + Amazon Bedrock + Amazon Textract + AWS KMS + Amazon DynamoDB**, derivada do racional "Criando um Assistente de Delivery com AWS Step Functions e Bedrock" para o desafio como publicado neste repertório.

## Objetivo

Automatizar o processamento de documentos PDF enviados para Amazon S3. O workflow extrai o texto, utiliza um foundation model no Amazon Bedrock para gerar **resumo e categoria**, normaliza a resposta, criptografa o resumo e grava os resultados no DynamoDB.

## Arquitetura

`S3/EventBridge -> Step Functions -> Lambda/Textract -> Bedrock -> Lambda/Parser -> KMS -> DynamoDB`

A solução escreve a sequência: S3 com EventBridge dispara a máquina; uma Lambda utiliza Textract para extrair conteúdo; Bedrock produz resumo e categoria; KMS criptografa o resumo; e DynamoDB persiste os dados.

## Nome da State Machine

**Processamento_Inteligente_de_Documentos**

## Estrutura para GitHub

```text
Processamento_Inteligente_de_Documentos/
├── README.md
├── src/
│   └── Processamento_Inteligente_de_Documentos.json
└── docs/
    └── COMPONENTES.md
```

## Componentes do workflow

1. **Extração de Texto do PDF** — chama uma Lambda que encapsula o Amazon Textract.
2. **Bedrock Generate Summary and Category** — integração nativa do Step Functions com `bedrockruntime:invokeModel`.
3. **Get Summary and Category** — normaliza a saída do modelo e separa `summary` e `category`.
4. **Encrypt Summary** — utiliza KMS para criptografar o resumo.
5. **Save Summary** — persiste os dados no DynamoDB.
6. **Falha no Processamento** — caminho centralizado para falhas permanentes.

## Insights desenvolvidos

- **Integração nativa com AWS SDK:** reduz código intermediário na orquestração.
- **Componentização:** cada etapa possui responsabilidade única e pode evoluir independentemente.
- **Segurança:** o resumo é criptografado antes da persistência.
- **Resiliência:** foram adicionadas políticas de retry para chamadas suscetíveis a falhas transitórias.
- **Prompt estruturado:** o modelo é orientado a devolver JSON com `summary` e `category`.

## Possibilidades de evolução

- Roteamento por tipo de documento usando `Choice`.
- Processamento de múltiplos PDFs com `Map`.
- Aprovação humana para documentos sensíveis.
- Integração com pipeline de RAG e embeddings.
- SNS/SQS ou DLQ para falhas permanentes.
- CloudWatch/X-Ray para observabilidade.
- Versionamento e aliases de modelos Bedrock.

## Parâmetros a substituir

- `${TEXTRACT_LAMBDA_ARN}`
- `${GET_SUMMARY_CATEGORY_LAMBDA_ARN}`
- `${KMS_KEY_ID}`
- `${DYNAMODB_TABLE_NAME}`

## Permissões IAM esperadas

A role da State Machine deve permitir, conforme os recursos reais do ambiente:

- `lambda:InvokeFunction`
- `bedrock:InvokeModel`
- `kms:Encrypt`
- `dynamodb:PutItem`

## Teste funcional

1. Faça upload de um PDF no bucket de entrada.
2. O EventBridge deverá iniciar a State Machine.
3. Valide a execução no Step Functions.
4. Confirme o item persistido no DynamoDB.

## Observação sobre comentários `#`

Para manter o arquivo publicável e válido, as explicações dos componentes estão em `docs/COMPONENTES.md`, com cada seção precedida por `#`, enquanto o arquivo principal permanece JSON válido.

## Validação

O arquivo ASL foi validado como JSON sintaticamente válido. A parametrização com `${...}` é proposital e deve ser substituída pelo recurso/valor real do ambiente antes da publicação da State Machine.
