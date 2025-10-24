# Webhooks (Parte 2)

No [último capítulo](discord-webhooks-part-1.md) cobrimos como criar os webhooks via código, que para ser honesto não é muito útil, neste capítulo vamos continuar de onde paramos e vamos realmente usar os webhooks que criamos em algum código de bot.

Agora, uma maneira que poderíamos usar isso, é pegar menções... Por alguma razão as pessoas pensam que é aceitável me mencionar e depois pouco tempo depois remover a menção se eu não responder dentro de X segundos ou minutos.

Temos duas escolhas, ou criar um bot autônomo, ou jogá-lo em um bot existente... Para os propósitos deste guia vou jogar o código em um bot autônomo, mas deve ser bastante autoexplicativo como adicionar isso a um bot existente.

Vamos pegar algum código de exemplo...

```javascript
const { Client, Intents } = require("discord.js");
const client = new Client({
  intents: [Intents.FLAGS.GUILDS, Intents.FLAGS.GUILD_MESSAGES]
});

client.on("ready", () => {
  console.log("Estou pronto!");
});

let prefix = "~";
client.on("messageCreate", (message) => {
  if (message.author.id === client.user.id || message.author.bot) return;
  let args = message.content.split(" ").slice(1);
  if (message.content.startsWith(`${prefix}ping`)) {
    message.channel.send("pong!");
  }
});

client.login("SuperSecretBotTokenHere");
```

Agora, temos o código de exemplo, queremos pegar nosso webhook feito anteriormente e pegar o `id` e `token` da URL, vamos fazer isso juntos!

Você quer começar definindo seu webhook no topo do seu código, não se esqueça de substituir `Webhook ID` e `Webhook Token` com seus respectivos valores.

```javascript
const { Client, Intents, WebhookClient } = require("discord.js");
const client = new Client({
  intents: [Intents.FLAGS.GUILDS, Intents.FLAGS.GUILD_MESSAGES]
});
const mentionHook = new WebhookClient({ id: "Webhook ID", token: "Webhook Token" });

client.on("ready", () => {
  console.log("Estou pronto!");
});

let prefix = "~";
client.on("messageCreate", (message) => {
  if (message.author.id === client.user.id || message.author.bot) return;
  let args = message.content.split(" ").slice(1);
  if (message.content.startsWith(`${prefix}ping`)) {
    message.channel.send("pong!");
  }
});

client.login("SuperSecretBotTokenHere");
```

Agora, esta parte será um pouco longa; mas dentro do evento de mensagem você quer verificar menções, agora quanto mais menções você puder capturar melhor, por exemplo há as menções `@everyone` e `@here`, menções de cargo e as menções diretas.

A documentação oficial tem o maravilhoso booleano `Message.mentions.has(data)`, esses dados podem ser um objeto `GuildChannel`, `User` ou `Role`, ou uma `string` representando o ID de qualquer uma das coisas mencionadas anteriormente, então dentro do evento de mensagem crie uma nova `declaração if`.

```javascript
client.on("messageCreate", (message) => {
  if (message.author.id === client.user.id || message.author.bot) return;
  if (message.mentions.has("SEU ID DE USUÁRIO")) {
      // Código Adicional
  }
  let args = message.content.split(" ").slice(1);
  if (槽essage.content.startsWith(`${prefix}ping`)) {
    message.channel.send("pong!");
  }
});
```

Ok, isso cobre menções diretas, mas e as menções mencionadas `@everyone`, `@here` e menções de cargo?

Bem o objeto `message` tem `mentions` que tem tanto `everyone` quanto `roles`, então é assim que o código vai parecer até agora...

```javascript
client.on("messageCreate", (message) => {
  if (message.author.id === client.user.id || message.author.bot) return;
  if (message.mentions.has("SEU ID DE USUÁRIO") || message.mentions.everyone || (message.guild && message.mentions.roles.filter(r => message.guild.members.cache.get("SEU ID DE USUÁRIO").roles.cache.has(r.id تك.size > 0)) {
      // Código Adicional
  }
  let args = message.content.split(" ").slice(1);
  if (message.content.startsWith(`${prefix}ping`)) {
    message.channel.send("pong!");
  }
});
```

Ok, isso está terminado, as coisas de detecção de menção, mas deixe-me cobrir aquela última parte...

```javascript
(message.guild && message.mentions.roles.filter(r => message.guild.members.cache.get("SEU ID DE USUÁRIO").roles.cache.has(r.id)).size > 0)
```

A verificação `message.guild` garantirá que estamos sendo mencionados dentro de um canal de servidor, agora a próxima parte é um pouco mais complexa basicamente o código está verificando por quaisquer cargos que foram mencionados e filtrando-os contra nossos próprios cargos se algum deles corresponder (fazendo o tamanho maior que 0) retornará true.

Até aqui tudo bem, estamos quase na metade do caminho... Configuramos as condições para verificar menções, agora só precisamos ignorar algumas coisas, a saber, nós mesmos, e bots.

```javascript
client.on("messageCreate", (message) => {
  if (message.author.id === client.user.id || message.author.bot) return;
  if (message.mentions.has("SEU ID DE USUÁRIO") || message.mentions.everyone || (message.guild && message.mentions.roles.filter(r => message.guild.members.cache.get("SEU ID DE USUÁRIO").roles.cache.has(r.id)).size > 0)) {
      if (message.author.id === "SEU ID DE USUÁRIO") return;
      // Código Adicional
  }
  let args = message.content.split(" ").slice(1);
  if (message.content.startsWith(`${prefix}ping`)) {
    message.channel.send("pong!");
  }
});
```

Este código pode parecer familiar, e você estaria certo. É o que usamos para fazer bots ignorarem a si mesmos e outros bots, mas mudamos um pouco para que se o `message.author` for o usuário alvo (você), então ignore.

Ok, agora terminamos com as condições para o webhook, vamos realmente usar o webhook! Pegue seu código até agora (ou copie o código de baixo)...

```javascript
const { Client, WebhookClient } = require("discord.js");
const client = new Client();
const mentionHook = new WebhookClient({ id: 'Webhook ID', token: 'Webhook Token' });

client.on("ready", () => {
  console.log("Estou pronto!");
});

let prefix = "~";
client.on("messageCreate", (message) => {
  if (message.author.id === client.user.id || message.author.bot) return;
  if (message.mentions.has("SEU ID DE USUÁRIO") || message.mentions.everyone || (message.guild && message.mentions.roles.filter(r => message.guild.members.cache.get("SEU ID DE USUÁRIO").roles.cache.has(r.id)).size > 0)) {
      if (message.author.id === "SEU ID DE USUÁRIO") return;
      // Código Adicional
  }
  let args = message.content.split(" ").slice(1);
  if (message.content.startsWith(`${prefix}ping`)) {
    message.channel.send("pong!");
  }
});

client.login("SuperSecretBotTokenHere");
```

... e adicione a seguinte linha abaixo de onde diz `// Código Adicional`

```javascript
mentionHook.send("Você foi mencionado!");
```

Agora, vamos preencher todos os detalhes que precisamos para fazer isso funcionar (webhook `id` e `token`, e seu `id` de usuário)

{% hint style="info" %}
Este webhook já foi deletado há muito tempo.
{% endhint %}

```javascript
const { Client, WebhookClient } = require("discord.js");
const client = new Client();
const mentionHook = new WebhookClient({ id: "WEBHOOK_ID", token: "WEBHOOK_TOKEN" });

client.on("ready", () => {
  console.log("Estou pronto!");
});

let prefix = "~";
client.on("messageCreate", (message) => {
  if (message.author.id === client.user.id || message.author.bot) return;
  if (message.mentions.has("146048938242211840") || message.mentions.everyone || (message.guild && message.mentions.roles.filter(r => message.guild.members.cache.get("146048938242211840").roles.cache.has(r.id)).size > 0)) {
      if (message.author.id === "146048938242211840") return;
      // Código Adicional
      mentionHook.send("Você foi mencionado!");
  }
  let args = message.content.split(" ").slice(1);
  if (message.content.startsWith(`${prefix}ping`)) {
    message.channel.send("pong!");
  }
});

client.login("SuperSecretBotTokenHere");
```

Agora, apenas peça para alguém te mencionar!

![Sendo Mencionado](../.gitbook/assets/wh06.png)

Se tudo correr bem, verifique o canal que você configurou o webhook e deve ver algo assim...

![Você foi Mencionado!](../.gitbook/assets/wh07.png)

Isso é basicamente isso para este guia, mas você pode apimentar a notificação do webhook pegando quem te mencionou, em qual canal/servidor você foi mencionado e qual era o conteúdo (em casos de deleção.)

No _Capítulo 3_ vou cobrir sites de terceiros como [Zapier](https://zapier.com/) e [IFTT](https://ifttt.com/), que permite expandir seu alcance de webhook para coisas como Facebook, Twitter, GMail, e mais!
