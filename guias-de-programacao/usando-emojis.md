# Usando Emojis

Aqui está um fato interessante que você pode não saber sobre bots no Discord: Eles têm acesso a cada "emoji personalizado" de cada servidor em que estão - de graça. Isso mesmo, você tem um recurso do Nitro, grátis no seu bot, agora mesmo! Nesta página vamos dar uma olhada em como aproveitar esses emojis, como acessá-los e como exibi-los.

## O que é um Emoji?

Vamos começar desmontando exatamente o que é um Emoji, como eles são configurados e como são acessados. Então aqui, temos um emoji:

![ayy](https://cdn.discordapp.com/emojis/305818615712579584.png)

Quando quero escrever este emoji no meu chat, simplesmente digito `:ayy:` e ele se transforma no acima (menor, é claro, mas ainda assim). Mas nos bastidores, 2 coisas acontecem para este emoji aparecer:

* Discord procura o emoji na minha lista, encontra aquele com o nome `ayy` e procura seu ID.
* Então envia o código _real_ do emoji para o servidor, que se parece com isso: `<:ayy:305818615712579584>`. Este é o código que compõe o emoji.
* Quando um cliente recebe o acima, ele procura a URL do Emoji de seu ID, para obter a localização da imagem. Neste caso, é: `https://cdn.discordapp.com/emojis/305818615712579584.png`.
* Como você pode ver o ID é a única coisa que realmente importa na URL. Este ID é único para cada emoji.

## Como o Discord.js armazena emojis?

Há dois lugares onde você pode pegar emojis usando discord.js: no cliente, e nos servidores. `client.emojis.cache` é uma coleção de cada emoji que o cliente tem acesso, e `guild.emojis.cache` é uma coleção dos emojis de um servidor específico.

Se você aprendeu algo de [Entendendo Coleções](../entendendo/colecoes.md), você já pode saber como obter algo por ID de uma coleção:

```javascript
const ayy = client.emojis.cache.get("305818615712579584");
```

Você também pode saber como usar `find` para obter algo com outra propriedade - então aqui, posso obter `ayy` através de seu nome:

```javascript
const ayy = client.emojis.cache.find(emoji => emoji.name === "ayy");
```

## Exibindo Emoji no chat

Mas como alguém exibe esse emoji no chat? Bem, assim como usuários e cargos, emojis têm um método especial `.toString()` que os converte para o formato apropriado. Então, `ayy.toString()` realmente exibirá o `<:ayy:305818615712579584>` que vimos acima, que o cliente transforma em um emoji adequado.

Você também pode aproveitar concatenação e literais de template para simplificar a tarefa, já que eles farão a conversão automaticamente para você:

```javascript
if (message.content === "ayy") {
   const ayy = client.emojis.cache.find(emoji => emoji.name === "ayy");
   message.reply(`${ayy} LMAO`);
}
```

Se você quisesse listar todos os emojis em um servidor, uma operação simples de map na coleção deve dar resultados adequados:

```javascript
if (message.content === "listemojis") {
  const emojiList = message.guild.emojis.cache.map(emoji => emoji.toString()).join(" ");
  message.channel.send(emojiList);
}
```

Neste exemplo, você pode listar todos os emojis personalizados com (emoji.id, emoji.image e emoji.name).

```javascript
if (message.content === "listemojis") {
   const emojiList = message.guild.emojis.cache.map((e, x) => `${x} = ${e} | ${e.name}`).join("\n");
   message.channel.send(emojiList);
}

exemplo: 
450661466287112204 = :image: | name
```

## Reagindo com Emojis

Você também pode usar emojis personalizados como reações a mensagens, usando `message.react(emoji)`. No caso de emojis personalizados, você deve usar o ID do emoji, então você poderia fazer algo como `message.react(ayy.id)` ou `message.react("305818615712579584")` para adicionar o emoji `ayy` como uma reação.

## Mas e os Emojis Unicode?

Não se esqueça que há uma coleção muito extensa de emojis que estão integrados ao Discord que você pode ter acesso. Discord usa Twemoji, fornecido pelo Twitter. Você pode usar esses emojis para reagir a mensagens diretamente.

A maneira que o Discord espera esses emojis no entanto é que eles tenham que ser o caractere _unicode_, não o "texto". Significa, você não pode simplesmente fazer `message.send(":poop:")` e esperar ver 💩 aparecer. Você realmente precisa obter o valor unicode. Como você faz isso? Apenas escape o emoji no chat: `\:poop:` mostrará como 💩. Você pode copiar/colar isso dentro do código do seu bot tanto em uma string de mensagem, ou como uma reação de emoji como `message.react("💩")`.
