# Integração com Cleverbot

{% hint style="danger" %}
Cleverbot **NÃO OFERECE MAIS** um período de teste gratuito; esta página do guia **NÃO** será atualizada para usar qualquer outro módulo.
{% endhint %}

Tive este pedido desde que comecei o An Idiot's Guide, na verdade foi um dos primeiros pedidos que tive, mas tive um pressentimento de que seria um episódio decepcionante curto, talvez um episódio de 5 minutos. Mas para um guia escrito seria perfeito!

Então para começar, vamos pegar o exemplo de [começando](../comecando/versao-longa.md) e jogá-lo em um arquivo.

```javascript
const { Client, Intents } = require("discord.js");
const client = new Client({
  intents: [Intents.FLAGS.GUILDS, Intents.FLAGS.GUILD_MESSAGES]
});

client.on("ready", () => {
  console.log("Estou pronto!");
});

client.on("messageCreate", (message) => {
  if (message.content.startsWith("ping")) {
    message.channel.send("pong!");
  }
});

client.login("SuperSecretBotTokenHere");
```

Uma vez que você tem isso, devemos verificar `cleverbot-node` em [npmjs.com](https://www.npmjs.com/package/cleverbot-node) e pegar o código de exemplo deles

```javascript
var Cleverbot = require("cleverbot-node");
cleverbot = new Cleverbot;
cleverbot.configure({ botapi: "IAMKEY" });
cleverbot.write(cleverMessage, function (response) {
   console.log(response.output);
});
```

Ok, temos ambas as partes que precisamos, agora antes de continuarmos devemos obter o módulo instalado, apenas execute `npm i cleverbot-node`.

Instalado? Bom! Agora, vamos para o passo final... o código.

Temos ambos os nossos códigos de exemplo, agora precisamos combiná-los para um bot funcionando.

> **NOTA:** Muitos dos desenvolvedores ingênuos apenas jogariam o exemplo do cleverbot direto em seu evento de mensagem e se perguntariam por que não estava funcionando. Criaria uma nova instância de Cleverbot e eventualmente causaria um vazamento de memória.

Certo, precisamos pegar as duas primeiras linhas do exemplo do cleverbot...

```javascript
var Cleverbot = require("cleverbot-node");
cleverbot = new Cleverbot;
```

...e colocá-las com nossas definições do discord.

```javascript
const { Client, Intents } = require("discord.js");
const Cleverbot = require("cleverbot-node");
const client = new Client({
  intents: [Intents.FLAGS.GUILDS, Intents.FLAGS.GUILD_MESSAGES]
});
const clbot = new Cleverbot;
clbot.configure({ botapi: "IAMKEY" });
```

Renomeei `cleverbot` para `clbot` para reduzir qualquer possível confusão entre os nomes das variáveis já que JavaScript diferencia maiúsculas de minúsculas.

Então pegamos o resto do código e colocamos isso dentro do nosso manipulador de evento de mensagem, mas para este exemplo só quero que o bot fale comigo em DMs, então vamos verificar o `type` do canal com o seguinte código, você pode fazê-lo responder em menções ou até mesmo em canais (eu honestamente aconselharia contra isso.) mas também precisamos adicionar o intent `DIRECT_MESSAGES` e o parcial `CHANNEL` já que canais DM podem não estar em cache.

Verificando o tipo do canal.

```javascript
if (message.channel.type === "DM") {
  // Código do Cleverbot vai aqui.
}
```

Não se esqueça de adicionar isso dentro do seu objeto client.

```javascript
[Intents.FLAGS.DIRECT_MESSAGES],
partials: ["CHANNEL"]
```

Seu código deve parecer algo assim...

```javascript
const { Client, Intents } = require("discord.js");
const Cleverbot = require("cleverbot-node");
const client = new Client({
  intents: [Intents.FLAGS.GUILDS, Intents.FLAGS.GUILD_MESSAGES, Intents.FLAGS.DIRECT_MESSAGES],
  partials: ["CHANNEL"]
});
const clbot = new Cleverbot;
clbot.configure({ botapi: "IAMKEY" });

client.on("messageCreate", message => {
  if (message.channel.type === "DM") {
    clbot.write(message.content, (response) => {
      message.channel.sendTyping();
      setTimeout(() => {
        message.channel.send(response.output).catch(console.error);
      }, Math.random() * (1 - 3) + 1 * 1000);
    });
  }
});
 
client.on("ready", () => {
  console.log("Estou pronto!");
});
 
client.login("superSecretBotTokenHere");
```

Se tudo estiver como acima, então apenas envie uma DM para seu bot e veja a mágica acontecer!

![Sucesso!](../.gitbook/assets/cleverbot-results.png)
