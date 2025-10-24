---
description: >-
  Alguns exemplos de código muito rápidos para ações comuns no bot, usuários, membros,
  canais e mensagens!
---

# Perguntas Frequentes

Nesta página, algumas perguntas muito básicas e frequentes são respondidas. É importante entender que **esses exemplos são genéricos** e provavelmente não funcionarão se você apenas copiar/colar em seu código. Você precisa **entender** essas linhas, não apenas empurrá-las cegamente em seu código.

## Exemplos de Código

## Bot e Cliente do Bot

```javascript
// Definir o status "Assistindo: " do bot (deve estar em um evento!)
client.on("ready", () => {
    client.user.setActivity("meu código", { type: "WATCHING"})
})
```

```javascript
// Definir o status online/ausente/dnd/invisível do bot
client.on("ready", () => {
    client.user.setStatus("online");
});
```

```javascript
// Definir a presença do bot (atividade e status)
client.on("ready", () => {
    client.user.setPresence({
        activities: [{ 
          name: "meu código",
          type: "WATCHING"
        }],
        status: "idle"
    })
})
```

Nota: Você pode encontrar uma lista de todos os tipos de atividade possíveis [aqui](https://discord.js.org/#/docs/main/stable/typedef/ActivityType).

{% hint style="info" %}
Se você quiser que o status do seu bot apareça como `STREAMING`, você precisa fornecer uma URL do Twitch ou YouTube.
Para `setActivity`, você precisa fornecer um objeto de opções, que precisa ter a URL, e o tipo deve ser definido como streaming.
Para `setPresence`, você precisa fornecer o objeto de dados de presença, que precisa conter o array de atividades, com a url e o tipo (Defina-o como "STREAMING").
{% endhint %}

```javascript
client.on("ready", () => {
    client.user.setActivity("meu código", { type: "STREAMING", url: "https://www.twitch.tv/an_idiots_guide" })
})
```

## Usuários e Membros

{% hint style="info" %}
Nestes exemplos `Guild` é um placeholder para onde você obtém o servidor. Isso pode ser `message.guild` ou `member.guild` ou apenas `guild` dependendo do evento. Ou você pode obter o servidor por ID (veja a próxima seção) e usar isso também!
{% endhint %}

```javascript
// Obter um Usuário por ID
client.users.cache.get("id do usuário aqui");
// Retorna <User>
```

```javascript
// Obter um Membro por ID
message.guild.members.cache.get("ID do usuário aqui");
// Retorna <Member>
```

```javascript
// Obter um Membro da Menção na Mensagem
message.mentions.members.first();
// Retorna <Member>, se houver um membro mencionado
```

```javascript
// Enviar uma Mensagem Direta para um usuário
message.author.send("olá");
// Com Member também funciona:
message.member.send("Ei!");
```

```javascript
// Mencionar um usuário em uma mensagem
message.channel.send(`Olá ${message.author.toString()}, e bem-vindo!`);
// ou
message.channel.send("Olá " + message.author.toString() + ", e bem-vindo!");
```

```javascript
// Restringir um comando a um usuário específico por ID
if (message.content.startsWith(`${prefix}commandname`)) {
    if (message.author.id !== "Um ID de usuário") return;
    // Seu Comando Aqui
}
```

```javascript
message.guild.members.fetch(message.author)
  .then(member => {
    // O membro está disponível aqui.
  })
  .catch(() => {
    // Quando ocorre um erro.
  })
```

## Canais e Servidores

```javascript
// Obter um Servidor por ID
client.guilds.cache.get("o id do servidor");
// Retorna <Guild>
```

```javascript
// Obter um Canal por ID
client.channels.cache.get("o id do canal");
// Retorna <Channel>
```

```javascript
// Obter um Canal por Nome
message.guild.channels.cache.find(channel => channel.name === "nome-do-canal");
// retorna <Channel>
```

```javascript
// Criar um convite e enviá-lo no canal
// Você só pode criar um convite de um GuildChannel
// Mensagens só podem ser enviadas para um TextChannel
message.guild.channels.cache.get('<ID DO CANAL>').createInvite().then(invite =>
    message.channel.send(invite.url)
);
```

## Mensagens

```javascript
// Editando uma mensagem que o bot enviou
message.channel.send("Teste").then(sentMessage => sentMessage.edit("Blah"));
// mensagem agora diz: "Blah"
```

```javascript
message.channel.messages.fetch("352292052538753025")
  .then(message => {
    // fazer algo com isso
    // Verificar se o autor da mensagem é o bot
    if (message.client.user.id !== message.id) return console.log("Eu não sou o autor dessa mensagem!");
    // Editar a mensagem
    message.edit("Esta mensagem buscada foi editada");
  });
```
