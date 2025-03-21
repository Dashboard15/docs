---
title: Criar uma ação JavaScript
intro: 'Neste guia, você aprenderá como criar uma ação JavaScript usando o conjunto de ferramentas de ações.'
redirect_from:
  - /articles/creating-a-javascript-action
  - /github/automating-your-workflow-with-github-actions/creating-a-javascript-action
  - /actions/automating-your-workflow-with-github-actions/creating-a-javascript-action
  - /actions/building-actions/creating-a-javascript-action
versions:
  fpt: '*'
  ghes: '*'
  ghae: '*'
  ghec: '*'
type: tutorial
topics:
  - Action development
  - JavaScript
shortTitle: Ação do JavaScript
---

{% data reusables.actions.enterprise-beta %}
{% data reusables.actions.enterprise-github-hosted-runners %}

## Introdução

Neste guia, você aprenderá os componentes básicos necessários para criar e usar uma ação JavaScript empacotada. Para manter o foco deste guia nos componentes necessários para empacotar a ação, a funcionalidade do código da ação é mínima. A ação imprime "Olá, mundo" nos registros ou "Olá, [who-to-greet]", se você fornecer um nome personalizado.

Este guia usa o módulo Node.js do kit de ferramentas {% data variables.product.prodname_actions %} para acelerar o desenvolvimento. Para obter mais informações, consulte o repositório [ações/conjuntos de ferramentas](https://github.com/actions/toolkit).

Ao terminar esse projeto, você entenderá como criar sua própria ação JavaScript e poderá testá-la em um fluxo de trabalho.

{% data reusables.actions.pure-javascript %}

{% data reusables.actions.context-injection-warning %}

## Pré-requisitos

Antes de começar, você deverá fazer o download do Node.js e criar um repositório público em {% data variables.product.prodname_dotcom %}.

1. Faça o download e instale o Node.js {% ifversion fpt or ghes > 3.3 or ghae-issue-5504 or ghec %}16.x{% else %}12.x{% endif %}, o que inclui npm.

  {% ifversion fpt or ghes > 3.3 or ghae-issue-5504 or ghec %}https://nodejs.org/en/download/{% else %}https://nodejs.org/en/download/releases/{% endif %}

1. Crie um novo repositório público em {% data variables.product.product_location %} e chame-o de "hello-world-javascript-action". Para obter mais informações, consulte "[Criar um repositório novo](/articles/creating-a-new-repository)".

1. Clone o repositório para seu computador. Para obter mais informações, consulte "[Clonar um repositório](/articles/cloning-a-repository)".

1. No seu terminal, mude os diretórios para seu novo repositório.

  ```shell{:copy}
  cd hello-world-javascript-action
  ```

1. No terminal, inicialize o diretório com npm para gerar um arquivo `package.json`.

  ```shell{:copy}
  npm init -y
  ```

## Criar um arquivo de metadados de ação

Crie um novo arquivo denominado `action.yml` no diretório `hello-world-javascript-action` com o código de exemplo a seguir. Para obter mais informações, consulte "[Sintaxe dos metadados para {% data variables.product.prodname_actions %}}](/actions/creating-actions/metadata-syntax-for-github-actions)."

```yaml{:copy}
name: 'Hello World'
description: 'Greet someone and record the time'
inputs:
  who-to-greet:  # id of input
    description: 'Who to greet'
    required: true
    default: 'World'
outputs:
  time: # id of output
    description: 'The time we greeted you'
runs:
  using: {% ifversion fpt or ghes > 3.3 or ghae-issue-5504 or ghec %}'node16'{% else %}'node12'{% endif %}
  main: 'index.js'
```

Esse arquivo define a entrada `who-to-greet` e a saída `time`. O arquivo também diz ao executor da ação como começar a executar essa ação JavaScript.

## Adicionar pacotes do kit de ferramenta de ações

O conjunto de ferramentas de ações é uma coleção de pacotes Node.js que permite a rápida criação de ações JavaScript com mais consistência.

O pacote do kit de ferramentas [`@actions/core`](https://github.com/actions/toolkit/tree/main/packages/core) fornece uma interface com os comandos do fluxo de trabalho, variáveis de entrada e saída, status de saída e mensagens de depuração.

O conjunto de ferramentas também oferece um pacote [`@actions/github`](https://github.com/actions/toolkit/tree/main/packages/github) que retorna um cliente REST Octokit autenticado e acesso aos contexto do GitHub Actions.

O conjunto de ferramentas oferece mais do que pacotes `core` and `github`. Para obter mais informações, consulte o repositório [ações/conjuntos de ferramentas](https://github.com/actions/toolkit).

No seu terminal, instale os pacotes de conjunto de ferramentas de ações `core` e `github`.

```shell{:copy}
npm install @actions/core
npm install @actions/github
```

Você pode ver agora um diretório `node_modules` com três módulos recém-instalados e um arquivo `package-lock.json` com as dependências do módulo instalado e as versões de cada módulo instalado.

## Gravar um código de ação

Esta ação usa o conjunto de ferramentas para obter a variável de entrada obrigatória `who-to-greet` no arquivo de metadados da ação e imprime "Hello [who-to-greet]" em uma mensagem de depuração no log. Na sequência, o script obtém a hora atual e a configura como uma variável de saída que pode ser usada pelas ações executadas posteriormente em um trabalho.

O GitHub Actions fornece informações de contexto sobre o evento webhook, Git refs, fluxo de trabalho, ação e a pessoa que acionou o fluxo de trabalho. Para acessar as informações de contexto, você pode usar o pacote `github`. A ação que você vai escrever imprimirá a carga do evento webhook no log.

Adicione um arquivo novo denominado `index.js`, com o seguinte código:

{% raw %}
```javascript{:copy}
const core = require('@actions/core');
const github = require('@actions/github');

try {
  // `who-to-greet` input defined in action metadata file
  const nameToGreet = core.getInput('who-to-greet');
  console.log(`Hello ${nameToGreet}!`);
  const time = (new Date()).toTimeString();
  core.setOutput("time", time);
  // Get the JSON webhook payload for the event that triggered the workflow
  const payload = JSON.stringify(github.context.payload, undefined, 2)
  console.log(`The event payload: ${payload}`);
} catch (error) {
  core.setFailed(error.message);
}
```
{% endraw %}

Se um erro for lançado no exemplo `index.js` acima, `core.setFailed(error.message);` usará o pacote do conjunto de ferramentas de ações [`@actions/core`](https://github.com/actions/toolkit/tree/main/packages/core) para registrar uma mensagem em log e definir um código de saída de falha. Para obter mais informações, consulte "[Definindo os códigos de saída para as ações](/actions/creating-actions/setting-exit-codes-for-actions)".

## Criar README

Para que as pessoas saibam como usar sua ação, você pode criar um arquivo README. Um arquivo README é útil quando você planeja compartilhar publicamente sua ação, mas também é uma ótima maneira de lembrá-lo ou sua equipe sobre como usar a ação.

No diretório `hello-world-javascript-action`, crie um arquivo `README.md` que especifica as seguintes informações:

- Descrição detalhada do que a ação faz;
- Argumentos obrigatórios de entrada e saída;
- Argumentos opcionais de entrada e saída;
- Segredos usados pela ação;
- Variáveis de ambiente usadas pela ação;
- Um exemplo de uso da ação no fluxo de trabalho.

```markdown
# Hello world javascript action

Esta ação imprime "Hello World" ou "Hello" + o nome de uma pessoa a ser cumprimentada no log.

## Inputs

## `who-to-greet`

**Required** The name of the person to greet. Padrão `"World"`.

## Outputs

## `time`

The time we greeted you.

## Exemplo de uso

usa: ações/hello-world-javascript-action@v1.1
com:
  quem cumprimentar: 'Mona, a Octocat'
```

## Fazer commit, tag e push da sua ação para o GitHub

O {% data variables.product.product_name %} faz o download de cada ação executada em um fluxo de trabalho durante o tempo de execução e executa-a como um pacote completo do código antes de você poder usar os comandos do fluxo de trabalho como, por exemplo, `executar` para interagir com a máquina executora. Isso significa que você deve incluir quaisquer dependências de pacotes necessárias para executar o código JavaScript. Você precisará verificar os pacotes de conjuntos de ferramentas `core` e `github` no repositório de ação.

No seu terminal, faça commit dos arquivos `action.yml`, `index.js`, `node_modules`, `package.json`, `package-lock.json` e `README.md`. Se você adicionar um arquivo `.gitignore` que lista `node_modules`, será necessário remover essa linha para fazer commit do diretório `node_modules`.

Adicionar uma tag da versão para versões da sua ação é considerada uma prática recomendada. Para obter mais informações sobre versões da sua ação, consulte "[Sobre ações](/actions/automating-your-workflow-with-github-actions/about-actions#using-release-management-for-actions)".

```shell{:copy}
git add action.yml index.js node_modules/* package.json package-lock.json README.md
git commit -m "My first action is ready"
git tag -a -m "My first action release" v1.1
git push --follow-tags
```

Verificar seu diretório `node_modules` pode causar problemas. Como alternativa, você pode usar uma ferramenta denominada [`@vercel/ncc`](https://github.com/vercel/ncc) para compilar o seu código e módulos em um arquivo usado para distribuição.

1. Instale o `vercel/ncc` executando este comando no seu terminal. `npm i -g @vercel/ncc`

1. Compile seu arquivo `index.js`. `ncc build index.js --license licenses.txt`

  Você verá um novo arquivo `dist/index.js` com seu código e os módulos compilados. Você também verá um arquivo que acompanha `dist/licenses.txt` e contém todas as licenças dos `node_modules` que você está usando.

1. Altere a palavra-chave `main` (principal) no arquivo `action.yml` para usar o novo arquivo `dist/index.js`. `main: 'dist/index.js'`

1. Se você já verificou o diretório `node_modules`, remova-o. `rm -rf node_modules/*`

1. No seu terminal, faça commit das atualizações para os arquivos `action.yml`, `dist/index.js` e `node_modules`.
```shell
git add action.yml dist/index.js node_modules/*
git commit -m "Use vercel/ncc"
git tag -a -m "My first action release" v1.1
git push --follow-tags
```

## Testar sua ação em um fluxo de trabalho

Agora você está pronto para testar sua ação em um fluxo de trabalho. Quando uma ação está em um repositório privado, a ação somente pode ser usada em fluxos de trabalho no mesmo repositório. Ações públicas podem ser usadas por fluxos de trabalho em qualquer repositório.

{% data reusables.actions.enterprise-marketplace-actions %}

### Exemplo usando uma ação pública

Este exemplo demonstra como sua nova ação pública pode ser executada dentro de um repositório externo.

Copie o seguinte YAML em um novo arquivo em `.github/workflows/main.yml` e atualize a linha `uses: octocat/hello-world-javascript-action@v1.1` com seu nome de usuário e o nome do repositório público que você criou acima. Você também pode substituir a entrada `who-to-greet` pelo seu nome.

{% raw %}
```yaml{:copy}
on: [push]

jobs:
  hello_world_job:
    runs-on: ubuntu-latest
    name: A job to say hello
    steps:
      - name: Hello world action step
        id: hello
        uses: octocat/hello-world-javascript-action@v1.1
        with:
          who-to-greet: 'Mona the Octocat'
      # Use the output from the `hello` step
      - name: Get the output time
        run: echo "The time was ${{ steps.hello.outputs.time }}"
```
{% endraw %}

Quando este fluxo de trabalho é acionado, o executor fará o download da ação `hello-world-javascript-action` do repositório público e, em seguida, irá executá-la.

### Exemplo usando uma ação privada

Copie o código do fluxo de trabalho em um arquivo `.github/workflows/main.yml` no repositório da ação. Você também pode substituir a entrada `who-to-greet` pelo seu nome.

{% raw %}
**.github/workflows/main.yml**
```yaml{:copy}
em: [push]

trabalhos:
  hello_world_job:
    runs-on: ubuntu-latest
    nome: Um trabalho para dizer "Olá"
    etapas:
      # Para usar a ação privada desse repositório,
      # você deve verificar o repositório
      - nome: Checkout
        usa: actions/checkout@v2
      - nome: Etapa da ação "Olá, mundo"
        usa: ./ # Usa uma ação no diretório-raiz
        id: olá
        com:
          who-to-greet: 'Mona the Octocat'
      # Usa a saída da etapa `hello`
      - nome: Obtém o tempo de saída
        executar: echo "O tempo foi ${{ steps.hello.outputs.time }}"
```
{% endraw %}

No seu repositório, clique na aba **Ações** e selecione a última execução do fluxo de trabalho. {% ifversion fpt or ghes > 3.0 or ghae or ghec %}Em **Trabalhos** ou no gráfico de visualização, clique em **A job to say hello**. {% endif %}Você deverá ver "Hello Mona the Octocat" ou o nome que você usou como entrada em `who-to-greet` e o horário impresso no log.

{% ifversion fpt or ghes > 3.0 or ghae or ghec %}
![Uma captura de tela de sua ação em um fluxo de trabalho](/assets/images/help/repository/javascript-action-workflow-run-updated-2.png)
{% elsif ghes %}
![Uma captura de tela de sua ação em um fluxo de trabalho](/assets/images/help/repository/javascript-action-workflow-run-updated.png)
{% else %}
![Uma captura de tela de sua ação em um fluxo de trabalho](/assets/images/help/repository/javascript-action-workflow-run.png)
{% endif %}
