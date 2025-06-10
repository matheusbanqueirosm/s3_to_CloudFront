# 🔄 Alterando S3 para CloudFront

## 📚 Sumário

1. [Ativando o uso de CloudFront](#1-ativando-o-uso-de-cloudfront)  
2. [Trocando prefixos dos arquivos no banco de dados](#2-trocando-prefixos-dos-arquivos-no-banco-de-dados)

### 1. Ativando o uso de CloudFront

No repositório [`sm-click-infra`](https://github.com/service-marketing/sm-click-infra), acesse o diretório `environments` e edite o arquivo `prd.tfvars`.

Altere a variável:

```hcl
create_cloudfront = false
```

para:

```hcl
create_cloudfront = true
```

#### 🔧 O que essa alteração faz?

- ✅ Uma distribuição CloudFront configurada para servir arquivos diretamente do bucket S3 sm-click-client-files-{env}.

- ✅ Um OAC (Origin Access Control), garantindo que apenas o CloudFront tenha permissão para acessar os objetos do bucket.

- ❌ A política pública de leitura direta do S3 deixa de ser criada, tornando o bucket privado por padrão.

<hr />

### 2. Trocando Prefixos dos arquivos no banco de dados 

No repositório [`sm-click-back-utils`](https://github.com/service-marketing/sm-click-back-utils), acesse os diretórios `infra` > `environments` e edite o arquivo `prd.tfvars`.

1. Altere a variável:

```hcl
"USE_CLOUDFRONT_URL": "False",
```

para:

```hcl
"USE_CLOUDFRONT_URL": "True",
```

2. Entre no container do gunicorn `sm-click-back-utils-service-gunicorn` na AWS

Entre via CLI, no container:

```hcl
aws ecs execute-command --cluster sm-click-back-utils-cluster --task de4b787dd3014fbfafb178962c82d148 --container sm-click-back-utils-celery-worker --interactive --command "/bin/bash" 
```

Execute o comando para entrar no shell e executar a função:

```hcl
# Importa a função
from tools.tasks import change_s3_link_task

# Chama a função
change_s3_link_task.delay()
```

#### 🔧 O que essa alteração faz?

- ✅ Dispara a função change_s3_link, responsável por atualizar os prefixos das URLs de arquivos salvos no banco de dados.
  
- ✅ Varredura completa: percorre todos os modelos do Django, incluindo campos CharField, TextField e JSONField.
  
- ✅ Utiliza cache interno para evitar reprocessamentos e otimizar o desempenho.
  
- ✅ Processamento em lotes de 100 registros, com uso de transações atômicas para garantir integridade e performance.

- ✅ Função que retorna as urls get_public_file_url


#### 3. Trocando variavel USE_CLOUDFRONT_URL de false para true, Nos repositórios [`sm-click-back-app`](https://github.com/service-marketing/sm-click-back-app), [`sm-click-back-app-attendant`](https://github.com/service-marketing/sm-click-back-app-attendant), acesse o diretório `environments` e edite o arquivo `prd.tfvars`.

```hcl
"USE_CLOUDFRONT_URL": "False",
```

para:

```hcl
"USE_CLOUDFRONT_URL": "True",
```

