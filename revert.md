# 🔁 Revertendo CloudFront para S3

### 📚 Sumário

1. [Desabilitando o CloudFront](#1-desabilitando-o-cloudfront)  
2. [Revertendo caminho no FrontEnd](#2-revertendo-caminho-no-frontend)  
3. [Revertendo variáveis nos serviços](#3-revertendo-variáveis-nos-serviços)  
4. [Revertendo prefixos dos arquivos no banco de dados](#4-revertendo-prefixos-dos-arquivos-no-banco-de-dados)  
5. [Comandos no banco de dados](#5-comandos-no-banco-de-dados)

### ⚠️ Certifique-se de estar com a versão mais recente da tools

---

### 1. Desabilitando o CloudFront

> No repositório [`sm-click-infra`](https://github.com/service-marketing/sm-click-infra), acesse o diretório `environments` e edite o arquivo `prd.tfvars`.

1. Altere a variável:

```hcl
create_cloudfront = true
```
para:

```hcl
create_cloudfront = false
```

<hr />


### 2. Revertendo caminho no FrontEnd
> No repo [`sm-click-zap-front`](https://github.com/service-marketing/sm-zap-front)

Os arquivos `new_urls.js` e `register.vue` devem ter a URL revertida de CloudFront para: `https://sm-click-client-files-prd.s3.us-east-1.amazonaws.com/`

> No repo [`sm-click-attendance-screen`](https://github.com/service-marketing/sm-click-attendance-screen)

O arquivo `chatsStore.ts` deve ser revertido para usar: `https://sm-click-client-files-prd.s3.us-east-1.amazonaws.com/`

##### ❗  DEVEM SER TROCADOS PELA URL DO BUCKET S3 PADRÃO

<hr />

### 3. Revertendo variáveis nos serviços

> Nos repositórios [`sm-click-back-app`](https://github.com/service-marketing/sm-click-back-app), [`sm-click-back-attendances`](https://github.com/service-marketing/sm-click-back-attendances), [`sm-click-back-integrations`](https://github.com/service-marketing/sm-click-back-integrations) [`sm-click-back-app-attendant`](https://github.com/service-marketing/sm-click-back-app-attendant), acesse o diretório `environments` e edite o arquivo `prd.tfvars`.

```hcl
"USE_CLOUDFRONT_URL": "True",
"CLOUDFRONT_URL": "https://dg80pau6gx30b.cloudfront.net",
```

para:

```hcl
"USE_CLOUDFRONT_URL": "False",
"CLOUDFRONT_URL": "",
```

<hr />


### 4. Revertendo prefixos dos arquivos no banco de dados

> No repositório [`sm-click-back-utils`](https://github.com/service-marketing/sm-click-back-utils), acesse os diretórios `infra` > `environments` e edite o arquivo `prd.tfvars`.

1. Altere as variáveis:

```hcl
"USE_CLOUDFRONT_URL": "True",
"CLOUDFRONT_URL": "https://dg80pau6gx30b.cloudfront.net",
```

para:

```hcl
"USE_CLOUDFRONT_URL": "False",
"CLOUDFRONT_URL": "",
```

2. (dev) é necessário aumentar a memória e cpu do celery-worker

3. Suba e ative o `SSMExecPolicyUtils` force uma nova implantação

4. Entre no container do celery worker `sm-click-back-utils-service-celery-worker` via CLI

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
  
<hr />

5. Comandos no banco de dados

> O mais fácil de ver é a tabela de contatos que tem bastate photo com a url

```hcl
SELECT * FROM contacts_contact
```
