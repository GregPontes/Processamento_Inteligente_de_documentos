# Componentes da solução

# Extração de Texto do PDF
Responsável por receber bucket e chave do objeto S3 e delegar à Lambda a extração do conteúdo do PDF usando Amazon Textract.

# Bedrock Generate Summary and Category
Responsável por chamar diretamente a API InvokeModel do Amazon Bedrock com Anthropic Claude Haiku 4.5 para classificação e sumarização do texto.

# Get Summary and Category
Responsável por interpretar a resposta estruturada do modelo e separar os campos `summary` e `category`, mantendo o workflow simples e modular.

# Encrypt Summary
Responsável por proteger o resumo por meio do AWS KMS antes da persistência.

# Save Summary
Responsável por gravar categoria, nome do documento, resumo e resumo criptografado no Amazon DynamoDB.

# Falha no Processamento
Responsável por encerrar a execução de maneira explícita quando uma etapa crítica falha após as tentativas de retry.

## Observação sobre JSON
Linhas iniciadas por `#` não podem ser inseridas no arquivo `.json` sem torná-lo inválido. Por isso, as explicações pedidas foram colocadas nesta documentação, enquanto o ASL permanece JSON estritamente válido.
