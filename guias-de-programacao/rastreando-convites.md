---
description: >-
  Usando um cache simples para rastrear convites e saber qual convite foi usado quando um
  novo membro entra em um servidor.
---

# Rastreando Convites Usados

Uma coisa relativamente frequente que as pessoas adorariam saber é "qual convite alguém usou para entrar". Infelizmente, a API do Discord não fornece informações sobre o convite usado para entrar em um servidor.

Para contornar isso, precisamos fazer duas etapas separadas. A primeira é buscar todos os convites para cada servidor e armazená-los em um [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) temporário. A segunda é buscar um convite do servidor sempre que alguém entra, e encontrar qual tem um novo uso nele. Felizmente, isso é bem simples!

{% hint style="info" %}
Para este código funcionar, você precisa ter o intent privilegiado `GUILD_MEMBERS` habilitado na Página de Aplicação do seu Bot. Você também precisa se certificar de inicializar que a propriedade Intents nas Opções do Cliente tenha este intent mencionado. Esses dois passos vitais lhe darão acesso aos eventos privilegiados `guildMemberAdd` e `guildMemberRemove`.
{% endhint %}

Embora o abaixo seja uma aproximação justa de rastreamento de convites, ainda não é perfeito. Há 2 coisas que não rastreia:

* Convites temporários de uso único (ao clicar com o botão direito em alguém, e fazer Convite Para => Servidor). Esses existem apenas por alguns momentos e não podem ser rastreados de forma alguma.

## Armazenando Convites em Cache

A primeira parte é buscar todos os convites e mantê-los em cache em um Map. Isso é feito no evento `ready`. Primeiro, no entanto, devemos garantir que o cache seja inicializado _fora_ do evento ready.

```javascript
// Inicializar o cache de convites
const invites = new Map();

// Um método muito útil para criar um atraso sem bloquear todo o script.
const wait = require("timers/promises").setTimeout;

client.on("ready", async () => {
  // "ready" não está realmente pronto. Precisamos esperar um pouco.
  await wait(1000);

  // Loop sobre todos os servidores
  client.guilds.cache.forEach(async (guild) => {
    // Buscar todos os Convites do Servidor
    const firstInvites = await guild.invites.fetch();
    // Definir a chave como ID do Servidor, e criar um map que tem o código do convite, e o número de usos
    invites.set(guild.id, new Map(firstInvites.map((invite) => [invite.code, invite.uses])));
  });
});
```

Prosseguindo para a segunda parte, precisamos ouvir os eventos `inviteCreate` e `inviteDelete` fornecidos pela API, e atualizar nosso cache de acordo.

```javascript
client.on("inviteDelete", (invite) => {
  // Deletar o Convite do Cache
  invites.get(invite.guild.id).delete(invite.code);
});

client.on("inviteCreate", (invite) => {
  // Atualizar cache em novos convites
  invites.get(invite.guild.id).set(invite.code, invite.uses);
});

```

## Armazenando em Cache Servidores que foram adicionados/removidos

Como já temos os Servidores e seus Convites em cache, precisamos verificar se fomos adicionados ou removidos de algum dos Servidores. Se formos adicionados, precisamos buscar e armazenar em cache todos os convites. Se formos removidos, podemos simplesmente deletar os dados do nosso cache. É simples assim

```javascript
client.on("guildCreate", (guild) => {
  // Fomos adicionados a um novo Servidor. Vamos buscar todos os convites, e salvá-lo no nosso cache
  guild.invites.fetch().then(guildInvites => {
    // Isso é o mesmo que o evento ready
    invites.set(guild.id, new Map(guildInvites.map((invite) => [invite.code, invite.uses])));
  })
});

client.on("guildDelete", (guild) => {
  // Fomos removidos de um Servidor. Vamos deletar todos os seus convites
  invites.delete(guild.id);
});
```

## Capturando Novos Membros

Então agora que temos nosso objeto `invites`, estamos prontos para ouvir o evento `guildMemberAdd`. Quando um novo membro entra, precisamos buscar todos os convites do servidor mais uma vez. Então, procuramos através dos nossos convites _em cache_ e vemos qual foi usado, comparando o uso atual do convite com os em cache.

```javascript
client.on("guildMemberAdd", member => {
  // Para comparar, precisamos carregar a lista de convites atual.
  member.guild.invites.fetch().then(newInvites => {
    // Estes são os convites *existentes* para o servidor.
    const oldInvites = invites.get(member.guild.id);
    // Procurar através dos convites, encontrar aquele para o qual os usos aumentaram.
    const invite = newInvites.find(i => i.uses > oldInvites.get(i.code));
    // Isso é apenas para simplificar a mensagem sendo enviada abaixo (inviter não tem uma propriedade tag)
    const inviter = client.users.cache.get(invite.inviter.id);
    // Obter o canal de log (mude para seu gosto)
    const logChannel = member.guild.channels.cache.find(channel => channel.name === "join-logs");
    // Uma mensagem bem básica com as informações que precisamos. 
    inviter
      ? logChannel.send(`${member.user.tag} entrou usando o código de convite ${invite.code} de ${inviter.tag}. Convite foi usado ${invite.uses} vezes desde sua criação.`)
      : logChannel.send(`${member.user.tag} entrou mas não consegui encontrar através de qual convite.`);
  });
});
```

E... bem, isso é basicamente isso. Mas....

## Há um Problema

Então aqui está o problema. Cada vez que você busca convites, está atingindo a API do Discord com uma solicitação de informações. Embora isso não seja um problema para bots pequenos, pode ser conforme o bot cresce. Não estou dizendo que usar este código te baniria da API - não há problema inerente em consultar a API se não for abuso. No entanto, há algumas questões técnicas com desempenho especialmente.

Quanto mais servidores você tem, mais convites existem em cada servidor, mais dados você está recebendo da API. Lembre-se, por causa da maneira como o evento `ready` funciona, você precisa _esperar_ um pouco antes de executar o método fetch, e quanto mais servidores você tem, mais tempo precisa esperar. Durante este tempo, entradas de membros não registrarão corretamente porque o cache não existe.

Além disso, o próprio cache cresce em tamanho, então você está usando mais memória. Além disso, leva mais tempo para ordenar através dos convites quanto mais convites existem. Pode fazer o bot parecer ser mais lento para responder a membros entrando.

Então para concluir, o código acima funciona perfeitamente bem e ele _não deveria_ te causar problemas com o Discord, mas eu não recomendaria implementar isso em bots maiores, especialmente se você está preocupado com memória e desempenho.
