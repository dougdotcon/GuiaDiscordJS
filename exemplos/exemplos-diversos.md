# Exemplos Diversos

## Convenções Usadas nos Exemplos

Convenções são importantes - elas são os acordos nos quais a sociedade funciona. Então vamos tirar um momento para concordar com algumas.

### Placeholders

Um "Placeholder" é um pedaço de texto que _substitui algo mais_. Nestes FAQs assumimos as seguintes variáveis como "placeholders" para as suas:

* `client` é um placeholder que corresponde à sua variável `client`, como cobrimos no final do guia [Começando](../comecando/versao-longa.md). `client.on("ready", () => {` por exemplo.
* `message` é um placeholder que corresponde à variável do evento `messageCreate` que se parece com algo assim: `client.on("messageCreate", message => {`.

## Exemplos

### awaitMessages

{% hint style="info" %}
Exemplo por: Lewdcario (84484653687267328)
{% endhint %}

Este exemplo envia uma pergunta e espera receber uma resposta de mensagem que diz `test`.

```javascript
message.channel.send("Qual tag você gostaria de ver? Este await será cancelado em 30 segundos. Terminará quando você fornecer uma mensagem que passe pelo filtro pela primeira vez.")
.then(() => {
  message.channel.awaitMessages(response => response.content === "test", {
    max: 1,
    time: 30000,
    errors: ["time"],
  })
  .then((collected) => {
      message.channel.send(`A mensagem coletada foi: ${collected.first().content}`);
    })
    .catch(() => {
      message.channel.send("Não houve mensagem coletada que passou pelo filtro dentro do limite de tempo!");
    });
});
```

### Criando um servidor

Discord silenciosamente mudou o endpoint da API Criar Servidor, bots pequenos (10 servidores ou menos) são capazes de criar servidores programaticamente agora. Este exemplo fará seu bot criar um novo servidor e criar um cargo com a permissão de administrador, e a única linha de código na parte inferior o aplicará a você quando você executá-lo quando entrar no servidor.

```javascript
const { Permissions } = require("discord.js");

/* Promises ES6 */
client.guilds.create("Servidor de Exemplo").then(guild => {
  guild.channels.cache.first().createInvite()
    .then(invite => client.users.cache.get("<USERID>").send(invite.url));
  guild.roles.create({ name: "Cargo de Exemplo", permissions: Permissions.FLAGS.ADMINISTRATOR })
    .then(role => client.users.cache.get("<UserId>").send(role.id))
    .catch(error => console.log(error))
});

/* async/await ES8 */
async function createGuild(client, message) {
  const { Permissions } = require("discord.js");
  try {
    const guild = await client.guilds.create("Servidor de Exemplo");
    const defaultChannel = guild.channels.cache.find(channel => channel.permissionsFor(guild.me).has(Permissions.FLAGS.SEND_MESSAGES));
    const invite = await defaultChannel.createInvite();
    await message.author.send(invite.url);
    const role = await guild.roles.create({ name: "Cargo de Exemplo", permissions: Permission.FLAGS.ADMINISTRATOR });
    await message.author.send(role.id);
  } catch (e) {
    console.error(e);
  }
}
createGuild(client, message);
// Execute isso uma vez que você entrou no servidor criado pelo bot.
message.member.roles.add("<O ID DO CARGO QUE VOCÊ RECEBEU>");
```

### Cooldown de Comando

{% hint style="info" %}
Exemplo por ItsJordan#4297
{% endhint %}

Adiciona um cooldown aos seus comandos para que o usuário tenha que esperar 2,5 segundos entre cada comando.

Você pode mudar a natureza do cooldown mudando o retorno para outra coisa.

```javascript
// Primeiro, isso deve estar no nível superior do seu código, **NÃO** em nenhum evento!
const talkedRecently = new Set();
```

```javascript
// Dentro do seu evento messageCreate, este código vai parar qualquer comando durante o cooldown.
// Deve ser colocado após seu código que verifica bots e prefixo, para melhor desempenho

if (talkedRecently.has(message.author.id))
  return;

// Adiciona o usuário ao set para que eles não possam falar por 2,5 segundos
talkedRecently.add(message.author.id);
setTimeout(() => {
  // Remove o usuário do set após 2,5 segundos
  talkedRecently.delete(message.author.id);
}, 2500);
```

### Prefixo de Menção

{% hint style="info" %}
Regex ou Expressões Regulares são usadas para corresponder combinações de caracteres em strings. Leia mais sobre elas [aqui](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions). Você pode criar e testá-las [aqui](https://regex101.com/?flavor=javascript)
{% endhint %}

Requerendo um pouco de regex, isso vai capturar quando uma mensagem começa com o bot sendo mencionado.

```javascript
client.on("messageCreate", message => {
  const prefixMention = new RegExp(`^<@!?${client.user.id}> `);
    const prefix = message.content.match(prefixMention) ? message.content.match(prefixMention)[0] : '!';

  // Vá em frente com o resto do seu código!
});
```

### Múltiplos Prefixos

Vamos fazer 3 prefixos, isso é bastante universal. Isso também poderia estar no config.json quando você chegar lá. Então vamos começar encontrando se o conteúdo da mensagem começa com qualquer um dos prefixos mencionados no array. Se não começar, retornamos.

```javascript
client.on("messageCreate", message => {
  const prefixes = ["!", "?", "/"];
  const prefix = prefixes.find(p => message.content.startsWith(p));
  if (!prefix) return;

  // Vá em frente com o resto do seu código!
});
```

### Extensão de Múltiplos Prefixos

```javascript
client.on("messageCreate", async message => {
  const prefixes = ["!", "\\?", "\\/", `<@!?${client.user.id}> `];
  const prefixRegex = new RegExp(`^(${prefixes.join("|")})`);
  const prefix = message.content.match(prefixRegex);

  // Vá em frente com o resto do seu código!
});
```

### Purga de um Canal

{% hint style="info" %}
Exemplo por Hindsight (139412744439988224)
{% endhint %}

Exemplo de uso: !purge @user 10 , ou !purge 25

```javascript
const user = message.mentions.users.first();

if (!/^\d+$/.test(message.content.split(" ")[1])) return message.reply('Por favor forneça um número válido');
// Verificar se o argumento fornecido é completamente um número. Executamos isso porque parseInt pode fazer parse de números assim 564gb, levando a alguns resultados indesejáveis

// Fazer Parse da Quantidade
const amount = !!parseInt(message.content.split(" ")[1]) ? parseInt(message.content.split(" ")[1]) : parseInt(message.content.split(" ")[2])

if (!amount) return message.reply("Deve especificar uma quantidade para deletar!");
if (!amount && !user) return message.reply("Deve especificar um usuário e quantidade, ou apenas uma quantidade, de mensagens para purgar!");
// Buscar 100 mensagens (serão filtradas e reduzidas até a quantidade máxima solicitada)
message.channel.messages.fetch({
 limit: 100,
}).then((messages) => {
 if (user) {
 const filterBy = user ? user.id : Client.user.id;
 messages = messages.filter(m => m.author.id === filterBy).array().slice(0, amount);
 }
 message.channel.bulkDelete(messages).catch(error => console.log(error.stack));
});
```

### Detector de Palavrões

Este detector de palavrões rápido e sujo pega um array de palavras ofensivas que não queremos ver, e dispara nele.

```javascript
const swearWords = ["darn", "shucks", "frak", "shite"]; // Certifique-se de que todas as palavras estão apenas em minúsculas.
if (swearWords.some(word => message.content.toLowerCase().includes(word.toLowerCase()))) { // Minúsculas no conteúdo da mensagem para melhor correspondência
  message.reply("Ah não você disse uma palavra feia!!!");
  // Ou apenas faça message.delete();
}
```

### Expulsando usuários (ou bots) de um canal de voz

Suporte para expulsar membros de canais de voz agora foi adicionado pelo Discord e pode ser alcançado fazendo o seguinte.

```javascript
const { Permissions } = require("discord.js");

// Certifique-se de que o usuário bot tem permissões para mover membros no servidor:
if (!message.guild.me.permissions.has(Permissions.FLAGS.MOVE_MEMBERS)) return message.reply("Faltando a permissão `Mover Membros` necessária.");

// Obter o usuário/bot mencionado e verificar se estão em um canal de voz:
const member = message.mentions.members.first();
if (!member) return message.reply("Você precisa @mencionar um usuário/bot para expulsar do canal de voz.");
if (!member.voice.channel) return message.reply("Esse usuário/bot não está em um canal de voz.");

// Agora definimos o canal de voz do membro como null, em outras palavras desconectando-os do canal de voz.
member.voice.setChannel(null);

// Finalmente, passe alguma resposta de usuário para mostrar que tudo funcionou:
message.react("👌");
/* ou apenas "message.reply", etc.. depende de você! */
```

Isso faz o mesmo que clicar no botão de desconectar em um usuário ou bot enquanto estão em um canal de voz.
