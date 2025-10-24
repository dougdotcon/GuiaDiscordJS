---
description: >-
  Starboards: Sua fonte premium de memes, mensagens fora de contexto, e citações
  engraçadas. Vamos fazer um, certo!
---

# Criando um Starboard

Este é um recurso muito aguardado, solicitado por muitas pessoas.

Vamos começar falando sobre o que é um starboard. Este é um exemplo tirado do Servidor Oficial do Discord.js.

![Starboard](../.gitbook/assets/starboard.png)

Um starboard é um recurso popular em bots que serve como um canal de mensagens que os usuários do servidor acham engraçadas, estúpidas, ou ambas! Para fazer um starboard funcionando, precisamos monitorar uma reação sendo adicionada a uma mensagem, e faremos isso com os eventos `messageReactionAdd` e `messageReactionRemove`.

{% hint style="info" %}
Antes de começarmos, há uma coisa que você precisa saber. Este tutorial é apenas imediatamente compatível com Guidebot Class. A menos que você esteja usando isso, precisará modificar o código para suas necessidades!
{% endhint %}

Então, vamos começar!

Neste bloco, apenas fazemos uma configuração simples para mais tarde. Para facilidade, pessoalmente defino o objeto de mensagem como `message`, mas isso é completamente opcional. Em seguida, pegamos a chave starboardChannel das configurações dos servidores.

{% hint style="info" %}
Lembrete: GuideBot e GuideBot-Class usam [Enmap](https://www.npmjs.com/package/enmap) para configurações!
{% endhint %}

Então, executamos algumas verificações na reação e na mensagem. Primeiro, verificamos se a reação **NÃO** é o emoji de estrela unicode. Em seguida, executamos duas verificações na mensagem, verificando para ver se o usuário que adicionou a reação foi o autor da mensagem, se o usuário que enviou a mensagem é a pessoa que reagiu a ela, e se o autor da mensagem é um bot. Se nenhuma dessas verificações retornar verdadeiro, estamos bons para continuar.

{% hint style="info" %}
Agora, é muito importante que você tenha uma chave starboardChannel nas configurações do servidor antes de tentar usar isso. Se você está usando Guidebot, pode simplesmente executar `conf add starboardChannel starboard` e aplicar a mudança para todos os servidores!
{% endhint %}

```javascript
module.exports = class {
  constructor(client) {
    this.client = client;
  }

  // É aqui que toda a ação acontece. 
  async run(reaction, user) {
    const message = reaction.message;
     // Esta é a primeira verificação onde verificamos para ver se a reação não é o emoji de estrela unicode.
    if (reaction.emoji.name !== "⭐") return;
     // Aqui verificamos para ver se a pessoa que reagiu é a pessoa que enviou a mensagem original.
    if (message.author.id === user.id) return message.channel.send(`${user}, você não pode dar estrela nas suas próprias mensagens.`);
    // Esta é nossa verificação final, verificando para ver se a mensagem foi enviada por um bot.
    if (message.author.bot) return message.channel.send(`${user}, você não pode dar estrela em mensagens de bot.`);
    // Aqui obtemos o canal do starboard das configurações dos servidores. 
    const { starboardChannel } = this.client.settings.get(message.guild.id); 
    // Aqui vamos encontrar o canal
    const starChannel = message.guild.channels.cache.find(channel => channel.name === starboardChannel)
    // Se não houver canal do starboard, paramos o evento de executar mais longe, e dizemos que eles não têm um canal do starboard.
    if (!starChannel) return message.channel.send(`Parece que você não tem um canal \`${starboardChannel}\`.`); 
  }
}
```

Agora, este próximo bloco pode parecer realmente complexo, mas realmente não é. Vamos quebrá-lo.

Primeiro, declaramos 6 variáveis aqui, fetch, stars, star, foundStar, image, e embed. `fetch` buscará as últimas 100 mensagens no canal do starboard, `stars` tenta encontrar um embed cujo rodapé termina com o id da mensagem.

Em seguida, começaremos uma declaração if se já houver um embed no canal do starboard para esta mensagem, e também definimos nossas 4 variáveis finais. `star` é um regex simples para verificar quantas estrelas o embed já tem, `foundStar` nos permite usar a cor do embed pré-existente, `image` verifica se há algo anexado à mensagem, e `embed` é nosso novo embed!

Também usamos uma função que ainda não foi discutida, e essa é `this.extension`. Ela verifica a mensagem para ver se tem alguma imagem. Você verá isso depois, já que não é _muito_ importante.

Eu disse que não era tão complicado. Vamos continuar.

```javascript
// Aqui buscamos 100 mensagens do canal do starboard.
const fetch = await starChannel.messages.fetch({ limit: 100 }); 
// Verificamos as mensagens dentro do objeto fetch para ver se a mensagem que foi reagida já é uma mensagem no starboard,
const stars = fetch.find(m => m.embeds[0].footer.text.startsWith("⭐") && m.embeds[0].footer.text.endsWith(message.id)); 
// Agora configuramos uma declaração if se a mensagem for encontrada dentro do starboard.
if (stars) {
  // Regex para verificar quantas estrelas o embed tem.
  const star = /^\⭐\s([0-9]{1,3})\s\|\s([0-9]{17,20})/.exec(stars.embeds[0].footer.text);
  // Uma variável que nos permite usar a cor do embed pré-existente.
  const foundStar = stars.embeds[0];
  // Usamos a função this.extension para ver se há algo anexado à mensagem.
  const image = message.attachments.size > 0 ? await this.extension(reaction, message.attachments.array()[0].url) : ""; 
  const embed = new MessageEmbed()
    .setColor(foundStar.color)
    .setDescription(foundStar.description)
    .setAuthor(message.author.tag, message.author.displayAvatarURL)
    .setTimestamp()
    .setFooter(`⭐ ${parseInt(star[1])+1} | ${message.id}`)
    .setImage(image);
  // Buscamos o ID da mensagem já no starboard.
  const starMsg = await starChannel.messages.fetch(stars.id);
  // E agora editamos a mensagem com o novo embed!
  await starMsg.edit({ embeds: [embed] }); 
}
```

{% hint style="info" %}
Confuso com o que /^\⭐\s([0-9]{1,3})\s\|\s([0-9]{17,20})/ é? É Regex ou Expressões Regulares, que é usado para fazer parse e encontrar texto. [Leia mais aqui](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions)
{% endhint %}

Agora, se você fosse apenas usar o código acima, seu starboard só funcionaria se já houvesse uma mensagem no canal do starboard. Vamos cuidar disso.

Aqui adicionamos uma declaração if que imita e é colocada após o bloco anterior, mas desta vez definindo manualmente a cor do embed, e também definindo manualmente a quantidade de estrelas que o embed terá.

```javascript
// Agora usamos uma declaração if se uma mensagem não for encontrada no starboard para a mensagem.
if (!stars) {
  // Usamos a função this.extension para ver se há algo anexado à mensagem.
  const image = message.attachments.size > 0 ? await this.extension(reaction, message.attachments.array()[0].url) : ""; 
  // Se a mensagem estiver vazia, não permitimos que o usuário dê estrela na mensagem.
  if (image === "" && message.cleanContent.length < 1) return message.channel.send(`${user}, você não pode dar estrela em uma mensagem vazia.`); 
  const embed = new MessageEmbed()
    // Definimos a cor para um amarelo agradável aqui.
    .setColor(15844367)
    // Aqui usamos cleanContent, que substitui todas as menções na mensagem por seus
    // texto equivalente. Por exemplo, um ping @everyone apenas exibirá como @everyone, sem marcar você!
    // Na data desta edição (09/06/18) embeds não mencionam ainda.
    // Mas nada está impedindo o Discord de habilitar menções de embeds em uma atualização futura.
    .setDescription(message.cleanContent) 
    .setAuthor(message.author.tag, message.author.displayAvatarURL)
    .setTimestamp(new Date())
    .setFooter(`⭐ 1 | ${message.id}`)
    .setImage(image);
  await starChannel.send({ embeds: [embed] });
}
```

Agora, se você tem estado seguindo exatamente, você tem um starboard funcionando! Mas se você gosta de um TL;DR, aqui está.

```javascript
module.exports = class {
  constructor(client) {
    this.client = client;
  }

  async run(reaction, user) {
    const message = reaction.message;
    if (reaction.emoji.name !== "⭐") return;
    if (message.author.id === user.id) return message.channel.send(`${user}, você não pode dar estrela nas suas próprias mensagens.`);
    if (message.author.bot) return message.channel.send(`${user}, você não pode dar estrela em mensagens de bot.`);
    const { starboardChannel } = this.client.settings.get(message.guild.id);
    const starChannel = message.guild.channels.cache.find(channel => channel.name === starboardChannel)
    if (!starChannel) return message.channel.send(`Parece que você não tem um canal \`${starboardChannel}\`.`); 
    const fetchedMessages = await starChannel.messages.fetch({ limit: 100 });
    const stars = fetchedMessages.find(m => m.embeds[0].footer.text.startsWith("⭐") && m.embeds[0].footer.text.endsWith(message.id));
    if (stars) {
      const star = /^\⭐\s([0-9]{1,3})\s\|\s([0-9]{17,20})/.exec(stars.embeds[0].footer.text);
      const foundStar = stars.embeds[0];
      const image = message.attachments.size > 0 ? await this.extension(reaction, message.attachments.array()[0].url) : "";
      const embed = new MessageEmbed()
        .setColor(foundStar.color)
        .setDescription(foundStar.description)
        .setAuthor(message.author.tag, message.author.displayAvatarURL)
        .setTimestamp()
        .setFooter(`⭐ ${parseInt(star[1])+1} | ${message.id}`)
        .setImage(image);
      const starMsg = await starChannel.messages.fetch(stars.id);
      await starMsg.edit({ embeds: [embed] });
    }
    if (!stars) {
      const image = message.attachments.size > 0 ? await this.extension(reaction, message.attachments.array()[0].url) : "";
      if (image === "" && message.cleanContent.length < 1) return message.channel.send(`${user}, você não pode dar estrela em uma mensagem vazia.`);
      const embed = new MessageEmbed()
        .setColor(15844367)
        .setDescription(message.cleanContent)
        .setAuthor(message.author.tag, message.author.displayAvatarURL)
        .setTimestamp(new Date())
        .setFooter(`⭐ 1 | ${message.id}`)
        .setImage(image);
      await starChannel.send({ embeds: [embed] });
    }
  }

  // Aqui adicionamos a função this.extension para verificar se há algo anexado à mensagem.
  extension(reaction, attachment) {
    const imageLink = attachment.split(".");
    const typeOfImage = imageLink[imageLink.length - 1];
    const image = /(jpg|jpeg|png|gif)/gi.test(typeOfImage);
    if (!image) return "";
    return attachment;
  }
};
```

Então, todo esse código acima é apenas para se uma reação for adicionada. Agora, vamos lidar com se uma reação for removida.

Aqui, imitamos muito de perto o evento messageReactionAdd, apenas desta vez vamos **subtrair** 1 da quantidade total de estrelas que estão no embed.'

Há apenas uma pequena diferença entre o evento `messageReactionAdd` e o evento `messageReactionRemove`, e essa é que o evento `messageReactionAdd` dispara quando uma reação é adicionada, e o evento `messageReactionRemove` dispara quando uma reação é removida.

```javascript
// Aqui precisamos verificar se o usuário que removeu a reação não é o autor da mensagem, já que a estrela então apenas removeria uma estrela como fizemos return quando ele adicionou
if (message.author.id === user.id) return;
if (stars) {
  const star = /^\⭐\s([0-9]{1,3})\s\|\s([0-9]{17,20})/.exec(stars.embeds[0].footer.text);
  const foundStar = stars.embeds[0];
  const image = message.attachments.size > 0 ? await this.extension(reaction, message.attachments.array()[0].url) : "";
  const embed = new MessageEmbed()
    .setColor(foundStar.color)
    .setDescription(foundStar.description)
    .setAuthor(message.author.tag, message.author.displayAvatarURL)
    .setTimestamp()
    .setFooter(`⭐ ${parseInt(star[1])-1} | ${message.id}`)
    .setImage(image);
  const starMsg = await starChannel.messages.fetch(stars.id);
  await starMsg.edit({ embeds: [embed] });
  // Aqui queremos verificar se a mensagem agora tem 0 Estrelas
  if (parseInt(star[1]) - 1 == 0) return setTimeout(() => starMsg.delete(), 1000);
}
```

Aqui está o bloco de código finalizado para o evento `messageReactionRemove`.

```javascript
module.exports = class {
  constructor(client) {
    this.client = client;
  }

  async run(reaction, user) {
    const { message } = reaction;
    if (message.author.id === user.id) return;
    if (reaction.emoji.name !== "⭐") return;
    const { starboardChannel } = this.client.settings.get(message.guild.id);
    const starChannel = message.guild.channels.cache.find(channel => channel.name == starboardChannel)
    if (!starChannel) return message.channel.send(`Parece que você não tem um canal \`${starboardChannel}\`.`); 
    const fetchedMessages = await starChannel.messages.fetch({ limit: 100 });
    const stars = fetchedMessages.find(m => m.embeds[0].footer.text.startsWith("⭐") && m.embeds[0].footer.text.endsWith(reaction.message.id));
    if (stars) {
      const star = /^\⭐\s([0-9]{1,3})\s\|\s([0-9]{17,20})/.exec(stars.embeds[0].footer.text);
      const foundStar = stars.embeds[0];
      const image = message.attachments.size > 0 ? await this.extension(reaction, message.attachments.array()[0].url) : "";
      const embed = new MessageEmbed()
        .setColor(foundStar.color)
        .setDescription(foundStar.description)
        .setAuthor(message.author.tag, message.author.displayAvatarURL)
        .setTimestamp()
        .setFooter(`⭐ ${parseInt(star[1])-1} | ${message.id}`)
        .setImage(image);
      const starMsg = await starChannel.messages.fetch(stars.id);
      await starMsg.edit({ embeds: [embed] });
      if (parseInt(star[1]) - 1 == 0) return setTimeout(() => starMsg.delete(), 1000);
    }
  }

  // Agora, pode parecer estranho que usamos isso no evento messageReactionRemove, mas ainda precisamos verificar se há uma imagem para que possamos defini-la, se necessário.
  extension(reaction, attachment) {
    const imageLink = attachment.split(".");
    const typeOfImage = imageLink[imageLink.length - 1];
    const image = /(jpg|jpeg|png|gif)/gi.test(typeOfImage);
    if (!image) return "";
    return attachment;
  };
};
```
