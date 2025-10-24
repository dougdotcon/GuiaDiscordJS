# Usando Variáveis de Ambiente

Um arquivo `.env` é um tipo de arquivo que contém variáveis de ambiente de uma aplicação. Variáveis de ambiente permitem que você integre facilmente seu bot com várias plataformas online (ex. Heroku), facilmente separe seu ambiente de produção e desenvolvimento, bem como mantenha informações importantes como seu token do bot, tokens de API e detalhes do banco de dados seguros.

Você não quer que essas informações sejam divulgadas para ninguém. E usar variáveis de ambiente em um arquivo `.env` é uma das melhores maneiras de proteger suas informações, sendo preferido sobre um arquivo `config.json`.

Para começar, é sugerido que você instale o pacote `dotenv` do NPM. Isso permite que você integre facilmente variáveis de ambiente em seu bot. Você pode fazer isso via o console. Certifique-se de estar no diretório raiz do seu bot. Execute este comando:

`npm install dotenv`

Uma vez instalado, você precisará criar um arquivo `.env`. Vou criar um exemplo aqui (vou usar a variável `CLIENT_TOKEN` do branch master do discord.js).

_Se estiver usando o branch v12 do discord.js, use_ `DISCORD_TOKEN=`_._

```text
CLIENT_TOKEN=[SEU_TOKEN_DO_BOT]
OWNER=[SEU_ID_DE_DONO]
PREFIX=[PREFIXO_PADRÃO_DO_BOT]
```

Depois que você criar e atualizar este arquivo, salve-o e vá para seu arquivo principal. Vou construir a partir do código de exemplo padrão do Discord.js.

```javascript
const { Client } = require("discord.js");
// Importar isso permite que você acesse as variáveis de ambiente do processo node em execução
require("dotenv").config();

const client = new Client();

// "process.env" acessa as variáveis de ambiente para o processo node em execução. PREFIX é a variável de ambiente que você definiu em seu arquivo .env
const prefix = process.env.PREFIX;

client.on("ready", () => {
  console.log(`Logado como ${client.user.tag}!`);
});

client.on("messageCreate", message => {

  // Aqui estou usando um dos manipuladores de comando básicos do An Idiot's Guide. Usando a variável de ambiente PREFIX acima, posso fazer o mesmo que o token do bot abaixo
  if (message.author.bot) return;
  if (message.content.indexOf(prefix.length) !== 0) return;

  const args = message.content.slice(prefix.length).trim().split(/ +/g);
  const command = args.shift().toLowerCase();

  if (command === "ping") {
    message.reply("Pong!");
  }
});

// Aqui você pode fazer login no bot. Automaticamente tenta fazer login no bot com a variável de ambiente que você definiu para seu token do bot (ou "CLIENT_TOKEN" ou "DISCORD_TOKEN")
client.login();
```

Isso é tudo que você precisará para começar. Agora vou explicar por que isso é importante e por que você deve usar variáveis de ambiente com um arquivo `.env` sobre um arquivo `config.json`.

## Usando Git (ex. GitHub)

Se você vai publicar seu código com Git para um site como GitHub, então é imperativo que você proteja informações garantindo que seu `.env` não seja commitado em um repositório. Você pode fazer isso usando um arquivo `.gitignore`. Simplesmente adicione o arquivo `.env` ao arquivo `.gitignore` em seu repositório local.

De lá em diante, quaisquer commits futuros ignorarão esse arquivo ou quaisquer outros arquivos ou diretórios em `.gitignore`. Você pode configurar um arquivo `.env.example` em seu lugar, mas isso nem sempre é necessário.

```text
.env
```

Seguindo isso, tudo que os usuários verão em seu código do bot é `process.env.ENV_VARIABLE` que não expõe nada!

## Usando Heroku

Heroku é um site que permite começar e hospedar suas aplicações gratuitamente. Neste caso, seu bot Discord. No entanto, Heroku requer que você use variáveis de ambiente. Se você configurar seus arquivos com o pacote `dotenv` e requeira as variáveis de ambiente específicas em seu código, então tudo que você precisa fazer é ir para as variáveis de ambiente em sua aplicação Heroku e adicionar a chave e o valor.

No Heroku, essas variáveis são chamadas de "Config Vars". Não vou entrar em detalhes aqui sobre Heroku, mas da mesma forma que você configura seu arquivo `.env`, você adicionaria novas variáveis de ambiente no Heroku.

![Config Vars do Heroku](https://i.imgur.com/MSmEO5K.png)

A única maneira que seu token do bot pode ser exposto junto com outras variáveis de ambiente é se você fizer uma ou mais dessas coisas:

* a) Você acidentalmente expõe seu arquivo `.env` porque não o adicionou ao `.gitignore`
* b) Você adicionou um colaborador através do Heroku. Um colaborador tem quase permissões completas como o dono do app

Ambos a e b podem ser controlados. É só você precisar ser esperto.

## Definir variáveis de ambiente no script de inicialização

Ao iniciar sua aplicação, seja localmente no seu computador, em um script npm start no seu `package.json` ou até mesmo em um `Dockerfile` você pode definir em qual ambiente seu bot deve executar.

Por exemplo, você pode querer executar seu bot em produção no Heroku ou Glitch e em desenvolvimento no seu computador. Você pode simplesmente fazer isso adicionando novos scripts no seu `package.json`.

Ao fazer `npm init` quando você primeiro fez seu bot, você deve ter visto um script `test` criado. A porção de scripts do seu `package.json` deve parecer assim se você não adicionou nada.

```javascript
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
},
```

Aqui, você pode adicionar múltiplos scripts. Vamos adicionar alguns scripts. `production`, `development`, e `start`.

```javascript
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "start": "node .",
  "production": "NODE_ENV=production&&npm start",
  "development": "set NODE_ENV=development&&npm start"
},
```

* Para iniciar seu bot em um ambiente de produção, você faria `npm run production`. Isso definirá `process.env.NODE_ENV` como `production`
* Para iniciar seu bot em um ambiente de desenvolvimento, você faria `npm run development`. Isso definirá `process.env.NODE_ENV` como `development`

Em seu código, você pode definir o que deve acontecer dependendo do ambiente carregado. Aqui está um exemplo onde seu bot deve mostrar o status de streaming apenas se o ambiente estiver em `production` quando o evento `ready` é disparado:

```javascript
require("dotenv").config();

// process.env.NODE_ENV permite que você obtenha o ambiente em que o processo node está
let ver = process.env.NODE_ENV;

client.on("ready", () => {

  if (ver === "production") {
    client.user.setActivity("An Idiot's Guide", { type: "STREAMING", url: "https://twitch.tv/something" })
  }

  else {
    client.user.setActivity("na terra do código", { type: "PLAYING" });
  }
});
```

Aqui me certifico de que o bot está definido para um status de streaming apenas se meu ambiente node estiver em `production`. Não quero que meu bot apareça como streaming enquanto estou trabalhando nele em um ambiente de desenvolvimento. Há muito mais que você pode fazer com isso, no entanto.

Por exemplo, você pode mudar os scripts `production` e `development` para usar arquivos principais completamente diferentes se desejar. Mas não vou entrar nisso aqui.

Se você não quiser usar scripts de inicialização, sempre pode definir o ambiente node diretamente na linha de comando. Aqui está como:

```bash
# Windows
SET NODE_ENV=development&&npm start
SET NODE_ENV=development&&node .
SET NODE_ENV=development&&node app.js

# Linux/MacOS
NODE_ENV=development&&npm start
NODE_ENV=development&&node .
NODE_ENV=development&&node app.js
```

Executar esses comandos diretamente através da linha de comando ou em scripts de inicialização deve funcionar. É totalmente com você. Mas para facilidade de uso ao implantar seu bot, você deve usar scripts de inicialização. Então, por exemplo, no Heroku, você pode adicionar isso no seu `Procfile`:

```text
# Heroku executará o bot em modo de produção
worker npm run production
```

Se você estiver usando Glitch, pode adicionar isso no seu script `start` no seu `package.json`:

```javascript
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "start": "NODE_ENV=production&&node app.js"
},
```

Você pode precisar mudar o nome do arquivo principal dependendo de como você o chamou/onde está localizado em seu projeto de bot.

## Mais informações

Aqui estão alguns links para mais informações que você pode ler sobre variáveis de ambiente e Git se não estiver muito familiarizado com isso:

* [Usando Git para compartilhar e atualizar código (An Idiot's Guide)](https://anidiots.guide/outros-guias/usando-git#ignoring-files)
* [Trabalhando com Variáveis de Ambiente em Node.js (Twilio Blog)](https://www.twilio.com/blog/2017/08/working-with-environment-variables-in-node-js.html)
* [`dotenv` NPM](https://www.npmjs.com/package/dotenv)
* [`dotenv-flow` NPM (Usado para múltiplos arquivos `.env`)](https://www.npmjs.com/package/dotenv-flow)
