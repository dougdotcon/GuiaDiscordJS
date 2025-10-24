# Usando Logs de Auditoria

De vez em quando, usuários no "Servidor Oficial do An Idiot's Guide" precisam consultar logs de auditoria por qualquer motivo, seja visualizando-os para certas ações ou adicionando coisas aos logs de auditoria. Este guia explicará tudo sobre logs de auditoria e como usá-los.

A primeira coisa que você precisará é um Bot Discord funcionando. Se você não tem um, por favor visite [Seu Primeiro Bot](../primeiro-bot/seu-primeiro-bot.md) para começar. Agora, algumas coisas para notar. Este guia está usando discord.js. O bot precisará de algumas permissões. A principal permissão que o bot precisará é `'VIEW_AUDIT_LOGS'`. Esta permissão permite que o bot visualize os logs de auditoria.

Agora que a permissão foi estabelecida. Vamos começar!

Primeiro, precisamos saber o que estamos fazendo com os logs de auditoria. Vamos registrar quem deletou uma mensagem usando o evento messageDelete. Este evento disparará sempre que uma mensagem em cache for deletada em um servidor.

```javascript
const { Permissions } = require("discord.js");

client.on("messageDelete", async (message) => {
  // Primeiro, precisamos de um canal de logs. 
  const logs = message.guild.channels.cache.find(channel => channel.name === "logs");

  // Se não houver canal de logs, podemos criá-lo se tivermos a permissão 'MANAGE_CHANNELS'
  // Lembre-se, isso é completamente opcional. Use seu melhor julgamento.
  if (message.guild.me.permissions.has(Permissions.FLAGS.MANAGE_CHANNELS) && !logs) {
    await message.guild.channels.create("logs", { type: "GUILD_TEXT" });
  }

  // Se não tivermos permissões, registrar ambos os erros no console
  if (!logs) { 
    return console.log("O canal de logs não existe e não pode ser criado");
  }

  /*
  Vamos estabelecer quem deletou a mensagem. Esta é a parte real dos logs de auditoria, yay!
  O "type" é como você vai estar pesquisando através dos logs de auditoria, como atualizações de cargo 
  ou membros banidos. Uma lista completa de tipos pode ser encontrada no final desta página.
  Tenha em mente que a seguinte linha usa alguma manipulação avançada de promise async/await. 
  Explicar exatamente como isso funciona está além do escopo deste guia.
  */
  const entry = await message.guild.fetchAuditLogs({ type: "MESSAGE_DELETE" }).then(audit => audit.entries.first())

  // Definir um usuário vazio por enquanto. Isso será usado mais tarde no guia.
  let user;

})
```

Isso é o que é retornado de `entry`. Esta informação nos permite comparar o timestamp, o usuário, o canal, e o executor.

```javascript
GuildAuditLogsEntry {
  targetType: 'MESSAGE',
  actionType: 'DELETE',
  action: 'MESSAGE_DELETE',
  reason: null,
  executor: [Object],
  changes: null,
  id: '434116792567201792',
  extra: [Object],
  target: [Object] }
```

Observe o campo `reason`. Alguns logs de auditoria, como expulsar e banir, podem fornecer uma razão. Você provavelmente pode fazer o bot registrar quando um usuário é banido e por qualquer razão. O que queremos é o executor da ação. Fazemos isso indo para o alvo `executor` já que é onde o usuário está armazenado. Abaixo está mostrando as informações que a propriedade `executor` nos fornece:

```javascript
User {
  id: '286270602820452353',
  username: 'Zoth The Wumpus',
  discriminator: '6066',
  avatar: '57361ef0f8e23e02a44069c3dd5f5f47',
  bot: false,
  lastMessageID: '435165896198062091',
  lastMessage: [Object] }
```

Com todas as informações acima, podemos começar a criar comparações para identificar quem realmente deletou a mensagem, seja o autor da mensagem ou outra pessoa.

```javascript
// Por favor, tenha em mente: Os logs de auditoria do Discord não registrarão as informações se o autor dessa mensagem a deletou.
// Fiz isso com uma série de verificações:
​ 
//definimos entry acima, então podemos usá-lo aqui para verificar o id do canal
if (entry.extra.channel.id === message.channel.id
​ 
//Então estamos verificando se o alvo é o mesmo que o id do autor
&& (entry.target.id === message.author.id)
​ 
// Estamos comparando o tempo já que os logs de auditoria às vezes são lentos. 
&& (entry.createdTimestamp > (Date.now() - 5000)

// Queremos verificar a contagem já que os logs de auditoria armazenam a quantidade deletada em um canal
&& entry.extra.count >= 1) {
  user = entry.executor.username
}

else { 
  // Quando tudo mais falha, podemos assumir que o autor deletou sua mensagem.
  user = message.author.username
}
```

Vamos fazer uma pausa para explicar exatamente o que está acontecendo no bloco de código acima. O `Date.now()` está obtendo o tempo atual (em milissegundos). Queremos subtrair 5 segundos para o atraso potencial nos logs de auditoria. O `entry` estará recuperando a entrada de log de auditoria mais recente e todas as suas informações que vêm com ela. O que essas informações contêm? Tudo que precisamos para registro. Com todas as informações dadas acima, vamos começar a enviar tudo para um canal.

```javascript
  // Definimos o canal de logs anteriormente neste guia, então agora podemos enviá-lo para o canal!
  logs.send(`Uma mensagem foi deletada em ${message.channel.name} por ${user}`;);
```

O código final deve parecer assim:

```javascript
const { Permissions } = require('discord.js');

client.on("messageDelete", async (message) => {
  const logs = message.guild.channels.cache.find(channel => channel.name === "logs");
  if (message.guild.me.permissions.has(Permissions.FLAGS.MANAGE_CHANNELS) && !logs) {
    message.guild.channels.create("logs", { type: "GUILD_TEXT" });
  }
  if (!message.guild.me.permissions.has(Permissions.FLAGS.MANAGE_CHANNELS) && !logs) { 
    console.log("O canal de logs não existe e tentou criar o canal mas estou faltando permissões")
  }  
  const entry = await message.guild.fetchAuditLogs({ type: "MESSAGE_DELETE" }).then(audit => audit.entries.first())
  let user = ""
    if (entry.extra.channel.id === message.channel.id
      && (entry.target.id === message.author.id)
      && (entry.createdTimestamp > (Date.now() - 5000))
      && (entry.extra.count >= 1)) {
    user = entry.executor.username
  }

  else { 
    user = message.author.username
  }
  logs.send(`Uma mensagem foi deletada em ${message.channel.name} por ${user}`);
})
```

E aí está. É assim que você pode visualizar logs de auditoria, pois a maioria deles, se não todos, funcionam da mesma forma.

[Tipos de Logs de Auditoria:](https://discord.js.org/#/docs/main/master/typedef/AuditLogAction)

```javascript
ALL: null
GUILD_UPDATE: 1
CHANNEL_CREATE: 10
CHANNEL_UPDATE: 11
CHANNEL_DELETE: 12
CHANNEL_OVERWRITE_CREATE: 13
CHANNEL_OVERWRITE_UPDATE: 14
CHANNEL_OVERWRITE_DELETE: 15
MEMBER_KICK: 20
MEMBER_PRUNE: 21
MEMBER_BAN_ADD: 22
MEMBER_BAN_REMOVE: 23
MEMBER_UPDATE: 24
MEMBER_ROLE_UPDATE: 25
MEMBER_MOVE: 26
MEMBER_DISCONNECT: 27
BOT_ADD: 28,
ROLE_CREATE: 30
ROLE_UPDATE: 31
ROLE_DELETE: 32
INVITE_CREATE: 40
INVITE_UPDATE: 41
INVITE_DELETE: 42
WEBHOOK_CREATE: 50
WEBHOOK_UPDATE: 51
WEBHOOK_DELETE: 52
EMOJI_CREATE: 60
EMOJI_UPDATE: 61
EMOJI_DELETE: 62
MESSAGE_DELETE: 72
MESSAGE_BULK_DELETE: 73
MESSAGE_PIN: 74
MESSAGE_UNPIN: 75
INTEGRATION_CREATE: 80
INTEGRATION_UPDATE: 81
INTEGRATION_DELETE: 82
```
