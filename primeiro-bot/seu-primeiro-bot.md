# Seu Primeiro Bot

Este capítulo assume que você seguiu o capítulo Começando e seu código do bot compila. Além disso, tenho que repetir: se você não entende o código que está prestes a ver, programar um bot pode não ser para você. Vá para [CodeAcademy](https://www.codecademy.com/learn/javascript) e aprenda Javascript.

Neste capítulo vou guiá-lo através do desenvolvimento de um bot simples com alguns comandos úteis. Começaremos com o exemplo que criamos no primeiro capítulo:

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

## Introduzindo Eventos

Antes de mergulharmos em mais código, precisamos primeiro entender o que é um _Evento_.

Isto é um evento:

```javascript
client.on("messageCreate", (message) => {
  // Este código executa quando o evento é disparado
});
```

Isto é, especificamente, um evento em _discord.js_ mas é similar a como outras APIs lidam com eventos. Este evento dispara _toda vez que o bot vê uma mensagem_. Isso inclui todos os canais aos quais o bot tem acesso, bem como qualquer mensagem direta ou privada que receba. Se alguém enviar 5 mensagens em um canal, este evento dispara 5 vezes.

Por que isso é importante? Bem, se você pretende usar seu bot em um servidor grande, ou se você quer que ele esteja em múltiplos servidores, isso se torna um grande número de eventos disparando a cada momento. Não quero entrar em muita conversa sobre otimização, mas para um único ponto: **use uma única função de evento para cada evento**.

Discord.js contém um grande número de eventos que podem disparar sob certas situações. Por exemplo, o evento `ready` dispara quando o bot entra online. O evento `guildMemberAdd` dispara quando um novo usuário entra em um servidor compartilhado com o bot. Para uma lista completa de eventos, veja [Eventos na documentação](https://discord.js.org/#/docs/main/stable/class/Client?scrollTo=e-applicationCommandCreate). Voltaremos a alguns desses mais tarde neste capítulo.

## Adicionando um segundo comando

Uma das primeiras coisas úteis que você pode querer aprender é como adicionar um segundo comando ao seu bot. Embora existam maneiras _melhores_ do que o que estou prestes a mostrar, por enquanto isso será suficiente.

{% hint style="info" %}
De agora em diante vou omitir o código que requer e inicia o discord.js e me concentrar em partes específicas do código.
{% endhint %}

```javascript
client.on("messageCreate", (message) => {
  if (message.content.startsWith("ping")) {
    message.channel.send("pong!");
  } else

  if (message.content.startsWith("foo")) {
    message.channel.send("bar!");
  }
});
```

Salve seu código e reinicie seu bot. Para fazer isso, use `CTRL+C` na linha de comando e execute novamente `node index.js`. Sim, há maneiras melhores de recarregar o código, como você verá mais tarde neste livro.

Você pode testar seu novo comando dizendo `foo` em um canal que você compartilha com o bot. Você também pode confirmar que `ping` ainda retorna `pong`!

## Usando um Prefixo

Você pode ter notado que muitos bots respondem a comandos que têm um prefixo. Isso pode ser um ponto de exclamação (!), um ponto (.), um ponto de interrogação(?), ou outro caractere mas com a introdução de comandos slash é fortemente aconselhado contra usar `/`. Mas isso é útil por duas coisas.

Primeiro, se você não usar um prefixo único e tiver mais de um bot em um servidor, ambos responderão aos mesmos comandos. Em servidores de desenvolvedores, digitar `!help` leva a uma enxurrada de respostas e mensagens privadas que é algo a evitar.

Segundo, no exemplo acima respondemos quando a mensagem _começa com_ os 3 caracteres, `foo`. Em seu estado atual, isso significa que a seguinte frase acionará a resposta do bot: **fool, you have not heard the last of me!**. Sim, esse é um exemplo estranho, mas ainda é válido - diga isso no canal do seu bot e ele responderá.

Para contornar isso, vamos usar um prefixo, que armazenaremos em uma variável. Dessa forma obtemos o prefixo bem como a capacidade de mudá-lo para todos os comandos em um só lugar. Aqui está um código de exemplo que faz isso:

```javascript
// Definir o prefixo
const prefix = "!";
client.on("messageCreate", (message) => {
  // Sair e parar se não estiver lá
  if (!message.content.startsWith(prefix)) return;

  // As aspas invertidas são Literais de Template introduzidos em Javascript em ES6 ou ES2015, como substituição para Concatenação de Strings Leia sobre eles aqui https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals
  if (message.content.startsWith(`${prefix}ping`)) {
    message.channel.send("pong!");
  } else

  if (message.content.startsWith(`${prefix}foo`)) {
    message.channel.send("bar!");
  }
});
```

As mudanças no código ainda são simples. Vamos passá-las:

* `const prefix = "!";` define o prefixo como o ponto de exclamação. Você pode mudá-lo para outra coisa, é claro.
* A linha `if (!msg.content.startsWith(prefix)) return;` é um pequeno pedaço de otimização que lê: "Se a mensagem não começar com meu prefixo, pare o que está fazendo". Isso impede que o resto da função execute, tornando seu bot mais rápido e responsivo.
* Os comandos mudaram para usar este prefixo, onde `startsWith(`\`${prefix}ping\`)` só seria acionado quando a mensagem começar com `!ping`.

O segundo ponto é tão importante quanto ter um único manipulador de evento `message`. Digamos que o bot receba cem mensagens a cada minuto (não é muito exagero em bots populares). Se a função não quebrar no início, você está processando essas cem mensagens em cada uma de suas condições de comando. Se, por outro lado, você quebrar quando o prefixo não estiver presente, está economizando todos esses ciclos de processador para coisas melhores. Se comandos são 1% de suas mensagens, você está economizando 99% de poder de processamento...

{% hint style="info" %}
OK, desculpe, estou mentindo um pouco. Não é 99%, isso é um exagero. É, no entanto, verdade que você economiza uma tonelada em poder de processador e RAM.
{% endhint %}

## Prevenindo Botception

Estamos praticamente terminados com o bot básico. Há uma última coisa sobre a qual quero falar: bots respondendo uns aos outros. Vamos fingir por um momento que você tem dois bots em seu servidor e cada um pode responder ao mesmo comando prefixado, `!help`. Mas quando esse comando é chamado, ele responde: `!help commands: Type !help followed by one of the following to see details: ping , foo`.

Agora, uma pessoa digita `!help` em um canal, e ambos os bots respondem. Mas, eles também verão o **outro** bot dizendo `!help commands: [...]`, verão isso como uma solicitação de ajuda, responderão uns aos outros... em um loop infinito. Para evitar que isso aconteça, podemos adicionar uma segunda condição dentro do nosso manipulador de evento `message`, logo abaixo daquela que verifica o prefixo:

```javascript
const prefix = "!";
client.on("messageCreate", (message) => {
  // nossa nova verificação:
  if (!message.content.startsWith(prefix) || message.author.bot) return;
  // [resto do código]
});
```

Essa condição contém um operador _OU_ ( || ), que lê como o seguinte:

{% hint style="info" %}
Se não houver prefixo ou o autor desta mensagem for um bot, pare de processar. Isso inclui este bot, ele mesmo.
{% endhint %}

E agora, temos um bot que responde apenas a 2 comandos e não desperdiça nenhum poder tentando descobrir qualquer outra coisa. Este é um bot básico completo? Claro! Então vamos terminar esta página aqui e vamos dar uma olhada em algum conceito novo na próxima.

O código completo do bot seria agora:

```javascript
const { Client, Intents } = require("discord.js");
const client = new Client({
  intents: [Intents.FLAGS.GUILDS, Intents.FLAGS.GUILD_MESSAGES]
});

// Definir o prefixo
let prefix = "!";
client.on("messageCreate", (message) => {
  // Sair e parar se o prefixo não estiver lá ou se o usuário for um bot
  if (!message.content.startsWith(prefix) || message.author.bot) return;

  if (message.content.startsWith(`${prefix}ping`)) {
    message.channel.send("pong!");
  } else

  if (message.content.startsWith(`${prefix}foo`)) {
    message.channel.send("bar!");
  }
});

client.login("SuperSecretBotTokenHere");
```

Toda vez que vejo aquele `SuperSecretBotTokenHere`, eu me encolho um pouco. Veja, não é uma boa prática ter tokens e coisas de autenticação em seu código, realmente deveria estar em um arquivo separado! Vá para [Adicionando um Arquivo de Configuração](adding-a-config-file.md) e vamos fazer isso.
