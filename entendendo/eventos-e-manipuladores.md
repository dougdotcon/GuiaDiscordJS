# Eventos e Manipuladores

Já exploramos um manipulador de eventos em [seu primeiro bot](../primeiro-bot/seu-primeiro-bot.md), o manipulador de `message`. Agora vamos dar uma olhada em alguns dos manipuladores mais importantes que você usará, junto com um exemplo.

{% hint style="danger" %}
_**NÃO ANINHE EVENTOS**_ Um ponto importante: Não aninhe nenhum evento (ou seja, "coloque um dentro de outro"). Nunca. Eventos devem estar no nível "raiz" do seu código, _ao lado_ do manipulador de `message` e não dentro dele.
{% endhint %}

## O evento `ready` e sua importância

Ah, programação assíncrona. Tão incrível. Tão difícil de entender quando você primeiro encontra. A realidade do discord.js e muitas, muitas outras bibliotecas que você encontrará, é que o código não é executado uma linha por vez, uma após a outra.

Deveria ter sido óbvio com o uso de `client.on("message")` que dispara para cada mensagem. Para explicar como o evento `ready` é importante, vamos olhar o seguinte código:

```javascript
const { Client, Intents } = require("discord.js");
const client = new Client({
  intents: [Intents.FLAGS.GUILDS, Intents.FLAGS.GUILD_MESSAGES]
});

client.user.setActivity("Online!");

client.login("SuperSecretBotTokenHere");
```

Este código não funcionará, porque `client` não está imediatamente disponível após ser inicializado. `client.user` será undefined neste caso, mesmo se invertêssemos as linhas setActivity e login. Isso porque leva uma pequena quantidade de tempo para o discord.js carregar seus servidores, usuários, canais e toda essa parafernália. Quanto mais servidores o bot estiver, mais tempo leva.

Para garantir que `client` e todas as suas "coisas" estejam prontas, podemos usar o evento `ready`. Qualquer código que você queira executar na inicialização que requer acesso ao objeto `client`, precisará estar neste evento.

Aqui está um exemplo simples de usar o manipulador de evento `ready`:

```javascript
client.on("ready", () => {
  client.user.setActivity(`em ${client.guilds.cache.size} servidores`);
  console.log(`Pronto para servir em ${client.guilds.cache.size} servidores, para ${client.users.cache.size} usuários.`);
});
```

## Detectando Novos Membros

Outro evento útil uma vez que você habilitou o intent privilegiado e adicionou `GUILD_MEMBERS` ao seu array de intents é [`guildMemberAdd`](https://discord.js.org/#/docs/main/master/class/Client?scrollTo=e-guildMemberAdd). Que dispara sempre que alguém entra em qualquer um dos servidores em que o bot está. Você verá isso em servidores menores: um bot dá boas-vindas a cada novo membro no canal \#welcome. O seguinte código faz isso.

```javascript
client.on("guildMemberAdd", (member) => {
  console.log(`Novo Usuário "${member.user.username}" entrou em "${member.guild.name}"` );
  member.guild.channels.cache.find(c => c.name === "welcome").send(`"${member.user.username}" entrou neste servidor`);
});
```

Os objetos disponíveis para cada evento são importantes: eles só estão disponíveis dentro desses contextos. Chamar `message` do `guildMemberAdd` não funcionaria - não está no contexto. `client` está sempre disponível dentro de todos os seus callbacks, é claro.

## Erros, Avisos e Mensagens de Debug

Sim, bots falham às vezes. E sim, a biblioteca também pode! Há um pequeno truque que podemos usar, no entanto, para prevenir travamentos completos às vezes: Capturando o evento `error`.

O seguinte pequeno trecho de código (que pode estar em qualquer lugar do seu arquivo) capturará todas as mensagens de saída do discord.js. Isso inclui todos os erros, avisos e mensagens de debug.

{% hint style="danger" %}
**NOTA:** O evento debug **VAI** exibir seu token parcialmente, então tenha cuidado ao entregar um log de debug.
{% endhint %}

```javascript
  client.on("error", (e) => console.error(e));
  client.on("warn", (e) => console.warn(e));
  client.on("debug", (e) => console.info(e));
```

## Testando Eventos

Então agora você está se perguntando, como testo esses eventos? Tenho que entrar em um servidor com uma conta alternativa para testar o evento guildMemberAdd? Isso não é, tipo, super chato?

Na verdade, há uma maneira fácil de testar quase qualquer evento. Sem entrar em muitos detalhes, `client`, seu Cliente Discord, estende algo chamado de `EventHandler`. Qualquer vez que você vir `client.on("something")` significa que você está manipulando um `evento` chamado `"something"`. Mas EventHandler tem outra função além de `on`. Tem `emit`. Emit é a contrapartida de `on`. Quando você `emit` um evento, ele é manipulado pelo callback para esse evento em `on`.

Então o que isso _significa_??? Significa que se _você_ emitir um evento, seu código pode capturá-lo. Eu sei eu sei estou divagando sem te dar um exemplo e você está aqui por exemplos. Aqui está um:

```javascript
client.emit("guildMemberAdd", message.member);
```

Isso emite o evento que normalmente dispara quando um novo membro entra em um servidor. Então está _fingindo_ que este membro específico entrou novamente no servidor mesmo que não tenha. Isso obviamente funciona para qualquer evento, mas você tem que fornecer os argumentos adequados para ele. Como `guildMemberAdd` requer apenas um membro, qualquer membro servirá (veja [FAQ](../perguntas-frequentes.md) para saber como obter outro membro). Posso disparar o evento `ready` novamente usando `client.emit("ready")` (o evento ready não recebe nenhum parâmetro).

E outros eventos? Vamos ver. `guildBanAdd` recebe 2 parâmetros: `guild` e `user`, para simular que um usuário foi banido. Então, você poderia `client.emit("guildBanAdd", message.guild, message.author)` para simular banir a pessoa enviando uma mensagem. Novamente, obter essas coisas (Servidores e Usuários) está no FAQ.

Você pode fazer tudo isso em um comando "test", ou pode fazer o que eu faço: usar `eval`. [Confira o comando Eval](../exemplos/criando-comando-eval.md) quando estiver pronto para ir por esse caminho.
