![Banner attendances (1)](https://github.com/user-attachments/assets/f5a004e1-1e4a-4ec2-a83a-68268484a092)

# 🔄 Alterando S3 para CloudFront

### 📚 Sumário

1. [Habilitando o CloudFront](#1-habilitando-o-cloudfront)
2. [Atualizando caminho no FrontEnd](#2-atualizando-caminho-no-FrontEnd)  
3. [Atualizando variáveis nos serviços](#3-atualizando-variáveis-nos-serviços)  
4. [Atualizando prefixos dos arquivos no banco de dados](#3-atualizando-prefixos-dos-arquivos-no-banco-de-dados)

### 1. Habilitando o CloudFront

> No repositório [`sm-click-infra`](https://github.com/service-marketing/sm-click-infra), acesse o diretório `environments` e edite o arquivo `prd.tfvars`.

1. Altere a variável:

```hcl
create_cloudfront = false
```

para:

```hcl
create_cloudfront = true
```

<hr />

#### 2. Atualizando caminho no FrontEnd
> No repo [`sm-click-zap-front`](https://github.com/service-marketing/sm-zap-front)

Os arquivos `new_urls.js` e `register.vue` contem  a url do bucket: `https://sm-click-client-files-prd.s3.us-east-1.amazonaws.com/`

> No repo [`sm-click-attendance-screen`](https://github.com/service-marketing/sm-click-attendance-screen)

O arquivo `chatsStore.ts` contem  a url do bucket: `https://sm-click-client-files-prd.s3.us-east-1.amazonaws.com/`

##### ❗ DEVEM SER TROCADOS PELA URL DO CLOUDFRONT DADA NO PASSO 1

<hr />

#### 3. Atualizando variáveis nos serviços

> Nos repositórios [`sm-click-back-app`](https://github.com/service-marketing/sm-click-back-app), [`sm-click-back-attendances`](https://github.com/service-marketing/sm-click-back-attendances), [`sm-click-back-integrations`](https://github.com/service-marketing/sm-click-back-integrations) [`sm-click-back-app-attendant`](https://github.com/service-marketing/sm-click-back-app-attendant), acesse o diretório `environments` e edite o arquivo `prd.tfvars`.

```hcl
"USE_CLOUDFRONT_URL": "False",
"CLOUDFRONT_URL": "",
```

para:

```hcl
"USE_CLOUDFRONT_URL": "True",
"CLOUDFRONT_URL": "https://URL DADA PELO CLOUDFRONT NA HORA QUE FOI CRIADO NO PASSO 1",
```

<hr />

### 3. Atualizando prefixos dos arquivos no banco de dados

> No repositório [`sm-click-back-utils`](https://github.com/service-marketing/sm-click-back-utils), acesse os diretórios `infra` > `environments` e edite o arquivo `prd.tfvars`.

1. Altere as variáveis:

```hcl
"USE_CLOUDFRONT_URL": "False",
"CLOUDFRONT_URL": "",
```

para:

```hcl
"USE_CLOUDFRONT_URL": "True",
"CLOUDFRONT_URL": "https://URL DADA PELO CLOUDFRONT NA HORA QUE FOI CRIADO NO PASSO 1",
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

