# Adicionando um Arquivo de Configuração

Agora que você tem um bot funcionando, podemos começar a dividi-lo em partes mais úteis. E a primeira parte disso é separar algumas das variáveis que definimos em um arquivo de configuração, `config.json`. Vamos carregar este arquivo na inicialização.

{% hint style="danger" %}
Colocar seu token em um arquivo de configuração está bom, mas **NÃO FAÇA COMMIT PARA O GITHUB** ou qualquer outro local público. Normalmente, adicionar um arquivo `.gitignore` ao seu projeto deve ser suficiente. [Aqui está um exemplo](https://github.com/github/gitignore/blob/master/Node.gitignore). Simplesmente adicione uma linha que diga `config.json` nesse arquivo, salve na raiz do seu projeto e você deve estar pronto. [Mais detalhes neste vídeo](https://www.youtube.com/watch?v=iyNIHQkVGao), e [nesta página](../outros-guias/usando-git.md).
{% endhint %}

## Por que um arquivo de configuração?

Uma das vantagens de ter um arquivo de configuração é que você pode copiar com segurança o código do seu bot para, digamos, hastebin.com para mostrar às pessoas, e seu token não estará lá. Uma segunda vantagem é que você pode fazer upload do código para um repositório como github e, desde que ignore o arquivo de configuração, seu bot pode ser compartilhado, mas permanecer seguro. Veremos isso em ação em um futuro walkthrough.

## Passo 1: O arquivo de configuração

As 2 coisas que podemos adicionar ao arquivo de configuração são:

* O token do bot
* O prefixo

Simplesmente pegue o seguinte exemplo e crie um novo arquivo na mesma pasta do arquivo do seu bot, chamando-o de `config.json`:

```javascript
{
  "token": "insira-o-token-do-bot-aqui",
  "prefix": "!"
}
```

## Passo 2: Requerir o arquivo de configuração

No topo do arquivo do seu bot, você precisa adicionar uma linha que carregará esta configuração e a colocará em uma variável. É assim que fica:

```javascript
const { Client,  Intents } = require("discord.js");
const client = new Client({
  intents: [Intents.FLAGS.GUILDS, Intents.FLAGS.GUILD_MESSAGES]
});
const config = require("./config.json");
```

Isso significa que agora, `config` é seu objeto de configuração. `config.token` é seu token, `config.prefix` é seu prefixo! Simples o suficiente.

## Passo 3: Usando `config` em seu código

Então vamos usar o que fizemos antes e usar o token do arquivo de configuração, em vez de colocá-lo diretamente no arquivo. A última linha do nosso bot fica assim:

```javascript
client.login("SuperSecretBotTokenHere");
```

E simplesmente precisamos mudá-la para isto:

```javascript
client.login(config.token);
```

A outra coisa que temos é, é claro, o prefixo. Novamente de antes, temos esta linha em nosso manipulador de mensagens:

```javascript
const prefix = "!";
client.on("messageCreate", (message) => {
  if (!message.content.startsWith(prefix) || message.author.bot) return;

  if (message.content.startsWith(`${prefix}ping`)) {
    message.channel.send("pong!");
  } else

  if (message.content.startsWith(`${prefix}foo`)) {
    message.channel.send("bar!");
  }
});
```

Estamos usando `prefix` em alguns lugares, então precisamos mudá-los todos. Aqui está como fica após as mudanças:

```javascript
client.on("messageCreate", (message) => {
  if (!message.content.startsWith(config.prefix) || message.author.bot) return;

  if (message.content.startsWith(`${config.prefix}ping`)) {
    message.channel.send("pong!");
  } else

  if (message.content.startsWith(`${config.prefix}foo`)) {
    message.channel.send("bar!");
  }
});
```

{% hint style="info" %}
Removemos a linha que define o prefixo. Não precisamos mais dela!
{% endhint %}

## Mudando a configuração

Se você está se perguntando "mas como mudo o prefixo agora?" não se preocupe, temos alguma ajuda para você. Sugerimos que você comece lendo o resto desta seção do guia ("Primeiro Bot") e depois vá para o [Guia de Configuração por Servidor na Documentação do Enmap](https://enmap.evie.dev/examples/per-server-settings)!

## Estendendo a ideia

Então há mais alguma coisa que você poderia colocar nesse arquivo de configuração? Absolutamente. Uma coisa que uso é para armazenar meu ID de usuário pessoal, para que meu bot possa usá-lo para me reconhecer e me dar acesso exclusivo a alguns comandos.

```javascript
{
  "token": "insira-o-token-do-bot-aqui",
  "prefix": "!",
  "ownerID": "seu-id-de-usuario"
}
```

Então, em um comando protegido ([eval](../exemplos/criando-comando-eval.md) por exemplo), poderia usar a seguinte linha para impedir o acesso a todos os usuários que pensam que podem usá-lo!:

```javascript
if (message.author.id !== config.ownerID) return;
```

## O que vem agora?

Então agora que temos um bot funcional com um arquivo de configuração, vamos adicionar mais coisas a ele! Siga-me para [Comando com argumentos](comando-com-argumentos.md) para a próxima parte!
