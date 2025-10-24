# Webhooks (Parte 1)

Este tem sido um tópico bastante demandado recentemente, todos querem saber como usar os webhooks, então aqui estou com este guia para explicar a cobertura básica dos webhooks.

Como de costume vamos pegar o código de exemplo.

```javascript
const { Client, Intents } = require("discord.js");
const client = new Client({
  intents: [Intents.FLAGS.GUILD, Intents.FLAGS.GUILD_MESSAGES]
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

Certo, vamos começar devagar, precisamos criar um webhook primeiro, se olharmos a [documentação](https://discord.js.org/#/docs/main/stable/class/TextChannel?scrollTo=createWebhook) ela vem com um exemplo, isso é basicamente tudo que precisamos para criar um webhook, mas vamos adicionar algum polimento a ele e jogá-lo em um comando básico.

```javascript
// Isso criará o webhook com o nome "Webhook de Exemplo" e um avatar de exemplo.
message.channel.createWebhook("Webhook de Exemplo", { avatar: "https://i.imgur.com/p2qNFag.png" })
// Isso fará o bot enviar uma DM para você com o webhook, se você usar isso em um selfbot,
// mude para console.log já que você não pode enviar DM para si mesmo
.then(wb => message.author.send(`Aqui está seu webhook ${wb.url}`))
.catch(console.error);
```

É assim que deve ficar se você testar o código.

![Criado o webhook](../.gitbook/assets/wh01.png)

![Webhook criado com sucesso](../.gitbook/assets/wh02.png)

Agora, isso está tudo bem e bom, podemos criar os webhooks e fazer nosso bot nos enviar DMs, mas os valores estão _hardcoded_, o que significa que se executarmos esse comando, obteríamos webhooks com o mesmo nome/avatar o tempo todo, vamos consertar isso, certo? vamos olhar a página [argumentos de comando](../primeiro-bot/comando-com-argumentos.md).

Você deve ter um manipulador de mensagens que se parece com algo assim.

```javascript
let prefix = "~";
client.on("messageCreate", message => {
  let args = message.content.split(" ").slice(1);
  if (message.content.startsWith(`${prefix}createHook`)) {
    message.channel.createWebhook("Webhook de Exemplo", { avatar: "https://i.imgur.com/p2qNFag.png" })
      .then(wb => message.author.send(`Aqui está seu webhook ${wb.url}`))
      .catch(console.error);
  }
});
```

Até aqui tudo bem, mas vamos nos deparar com um problema, e se você quiser dar ao seu webhook um nome que contém espaços? Agora você acabaria com a url do avatar no nome, então vamos ter que usar algum _regex_, [Expressões Regulares](https://developer.mozilla.org/en/docs/Web/JavaScript/Guide/Regular_Expressions) são muito poderosas, e muito intimidantes para começar, mas não se preocupe, o regex que vou fornecer para este exemplo funciona, apenas solte no seu código e está bom.

Aqui está o regex por si só

```javascript
/https?:\/\/.+\.(?:png|jpg|jpeg)/gi
```

Usar o regex acima com `match`, `replace` e `test` permitirá que você isole a url da imagem na string e deixe a string restante para ser usada como o nome do webhook, há uma ferramenta online incrível chamada [regex101.com](https://regex101.com/), com essa ferramenta fui capaz de criar o regex acima, aqui está uma imagem dele em ação.

![Regex em ação.](../.gitbook/assets/wh03.png)

Não vou entrar em muitos detalhes, mas o fato de que ambas as strings de teste estão destacadas, e está dizendo que há 2 correspondências é tudo que precisamos saber, funciona com links começando com `http` e `https`, e procura por extensões válidas, que são jpg, jpeg e png.

vamos fazer o resto do comando, certo? temos nosso regex, e sabemos que precisamos usar `match`, `replace` e `test`, então devemos `test` por um link primeiro usando nosso regex, se retornar false precisamos notificar o usuário, se retornar true precisamos `match` e `replace` para o resto, o código pode parecer assim.

```javascript
const nameAvatar = args.join(" ");
const linkCheck = /https?:\/\/.+\.(?:png|jpg|jpeg)/gi;
if (!linkCheck.test(nameAvatar)) return message.reply("Você deve fornecer um link de imagem.");
const avatar = nameAvatar.match(linkCheck)[0];
const name = nameAvatar.replace(linkCheck, "");
message.channel.createWebhook(name, { avatar })
  .then(wb => message.author.send(`Aqui está seu webhook ${wb.url}. \n\nPor favor mantenha isso seguro, pois você poderia ser explorado.`))
    .catch(error => console.log(error));
```

Ok, agora vamos juntar isso com nosso código do bot e emitir o comando!

![Uso do comando.](../.gitbook/assets/wh04.png)

E vamos verificar os webhooks do canal!

![Webhooks do Canal](../.gitbook/assets/wh05.png)

Wooo! conseguimos!

Agora podemos criar webhooks sob demanda via nosso código do bot, mas no próximo [_capítulo_](discord-webhooks-part-2.md) vamos ver o que podemos fazer com eles!
