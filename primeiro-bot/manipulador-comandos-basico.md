# Um Manipulador de Comandos Básico

Um _Manipulador de Comandos_ é essencialmente uma maneira de separar seus comandos em arquivos diferentes, em vez de ter um monte de condições `if/else` dentro do seu código (ou um `switch/case` se você estiver sendo chique).

Neste caso, o código mostra como separar cada comando em seu próprio arquivo. Isso significa que cada comando pode ser _editado_ separadamente e também _recarregado_ sem a necessidade de reiniciar seu bot. Sim, de verdade!

{% hint style="info" %}
Quer uma versão melhor e atualizada deste código? Estamos mantendo este manipulador de comandos no nível da comunidade. [Guide Bot está no Github](https://github.com/AnIdiotsGuide/guidebot/) e não apenas você pode usar o código, mas também pode contribuir se se sentir proficiente o suficiente!
{% endhint %}

## O que você precisa saber

Para escrever e usar corretamente um manipulador de comandos, sugiro que você se familiarize com algumas coisas.

* Confira [Introdução a Módulos](https://js.evie.dev/modules) para informações sobre módulos (que usaremos para cada comando)
* Entenda como [Eventos](../entendendo/eventos-e-manipuladores.md) funcionam e como cada evento tem argumentos diferentes fornecidos.
* Tenha uma boa compreensão de, pelo menos, [Comandos com Argumentos](comando-com-argumentos.md), que usaremos como base para a maior parte do nosso código.

## Mudanças no Arquivo Principal

Como estamos criando um arquivo (módulo) separado para cada evento e cada comando, nosso arquivo principal (app.js, ou index.js, ou o que você chamar) mudará drasticamente de uma lista de comandos para um arquivo simples que carrega outros arquivos.

Dois loops principais são necessários para executar este plano mestre. Primeiro, aquele que carregará todos os arquivos de `events`. Cada evento precisará ter um arquivo nessa pasta, nomeado _exatamente_ como o próprio evento. Então para `messageCreate` queremos `./events/messageCreate.js`, para `guildBanAdd` queremos `./events/guildBanAdd.js`, etc.

```javascript
// Ler os Arquivos no Diretório de Eventos e filtrar arquivos que terminam com .js
const files = fs.readdirSync("./events").filter(file => file.endsWith(".js"));
// Loop sobre cada arquivo
for (const file of files) {
  // Dividir o arquivo em sua extensão e obter o nome do evento
  const eventName = file.split(".")[0];
  // Requerir o arquivo
  const event = require(`./events/${file}`);
    // receita super-secreta para chamar eventos com todos os seus argumentos corretos *depois* da variável `client`.
    // sem entrar em muitos detalhes, isso significa que cada evento será chamado com o argumento client,
    // seguido de seus argumentos "normais", como message, member, etc etc.
    // Esta linha é incrível a propósito. Só estou dizendo.
  client.on(eventName, event.bind(null, client));
}
```

O segundo loop será para os próprios comandos. Por algumas razões, queremos colocar os comandos dentro de uma estrutura à qual possamos nos referir mais tarde - usaremos uma Coleção Discord:

```javascript
client.commands = new Discord.Collection();
// Ler o Diretório de Comandos e filtrar os arquivos que terminam com .js
const commands = fs.readdirSync("./commands").filter(file => file.endsWith(".js"));
// Loop sobre os arquivos de Comando
for (const file of commands) {
  // Obter o nome do comando dividindo o arquivo
  const commandName = file.split(".")[0];
  // Requerir o arquivo
  const command = require(`./commands/${file}`);

  console.log(`Tentando carregar comando ${commandName}`);
  // Definir o comando em uma coleção
  client.commands.set(command.name, command);
}
```

Ok então com isso dito, nosso arquivo principal agora se parece com isto (quão _limpo_ isso é, realmente?):

```javascript
const { Client, Intents } = require("discord.js");
const fs = require("fs");

const client = new Client({
  intents: [Intents.FLAGS.GUILDS, Intents.FLAGS.GUILD_MESSAGES]
});
const config = require("./config.json");
// Também precisamos garantir que estamos anexando a config ao CLIENT para que seja acessível em todos os lugares!
client.config = config;
client.commands = new Discord.Collection();

const events = fs.readdirSync("./events").filter(file => file.endsWith(".js"));
for (const file of events) {
  const eventName = file.split(".")[0];
  const event = require(`./events/${file}`);
  client.on(eventName, event.bind(null, client));
}

const commands = fs.readdirSync("./commands").filter(file => file.endsWith(".js"));
for (const file of commands) {
  const commandName = file.split(".")[0];
  const command = require(`./commands/${file}`);

  console.log(`Tentando carregar comando ${commandName}`);
  client.commands.set(command.name, command);
}

client.login(config.token);
```

## Nosso primeiro Evento: Mensagem

O evento `messageCreate` é obviamente o mais importante, pois receberá todas as mensagens enviadas ao bot. Crie o arquivo `./events/messageCreate.js` (certifique-se de que está escrito _exatamente_ assim) e olhe para este trecho de código:

```javascript
module.exports = (client, message) => {
  // Ignorar todos os bots
  if (message.author.bot) return;

  // Ignorar mensagens que não começam com o prefixo (em config.json)
  if (message.content.indexOf(client.config.prefix) !== 0) return;

  // Nossa definição padrão de argumento/nome de comando.
  const args = message.content.slice(client.config.prefix.length).trim().split(/ +/g);
  const command = args.shift().toLowerCase();

  // Pegar os dados do comando do Enmap client.commands
  const cmd = client.commands.get(command);

  // Se esse comando não existir, sair silenciosamente e não fazer nada
  if (!cmd) return;

  // Executar o comando
  cmd.run(client, message, args);
};
```

Há mais coisas que poderíamos fazer aqui, como obter configurações por servidor ou verificar permissões antes de executar o comando, etc. Por um desejo de manter esta página simples, evitei todo esse código extra, mas você ainda pode encontrá-lo no [repositório GuideBot](https://github.com/AnIdiotsGuide/guidebot/)!

## Comandos de exemplo

Este seria o conteúdo do arquivo `./commands/ping.js`, que é chamado com `!ping` (assumindo `!` como prefixo)

```javascript
exports.run = (client, message, args) => {
    message.channel.send("pong!").catch(console.error);
}
```

Outro exemplo seria o comando mais complexo `./commands/kick.js`, chamado usando `!kick @user`

```javascript
exports.run = (client, message, [mention, ...reason]) => {
  const modRole = message.guild.roles.cache.find(role => role.name === "Mods");
  if (!modRole)
    return console.log("O cargo Mods não existe");

  if (!message.member.roles.cache.has(modRole.id))
    return message.reply("Você não pode usar este comando.");

  if (message.mentions.members.size === 0)
    return message.reply("Por favor mencione um usuário para expulsar");

  if (!message.guild.me.hasPermission("KICK_MEMBERS"))
    return message.reply("Eu não tenho a permissão `KICK_MEMBERS`");

  const kickMember = message.mentions.members.first();

  kickMember.kick(reason.join(" ")).then(member => {
    message.reply(`${member.user.username} foi expulso com sucesso.`);
  });
};
```

Observe a estrutura na primeira linha. `exports.run` é o "nome da função" que é exportado, com 3 argumentos: `client` (o cliente), `message` (a variável message do manipulador) e `args`. Aqui, `args` é substituído por desestruturação chique que captura o `reason` (o resto da mensagem após a menção) em um array. Veja [Comandos com Argumentos](comando-com-argumentos.md) para detalhes.

## Outros Eventos

Eventos são tratados quase exatamente da mesma maneira, exceto que o número de argumentos depende de qual evento é. Por exemplo, o evento `ready`:

```javascript
module.exports = (client) => {
  console.log(`Pronto para servir em ${client.channels.cache.size} canais em ${client.guilds.cache.size} servidores, para um total de ${client.users.cache.size} usuários.`);
}
```

Note que o evento `ready` normalmente não tem argumentos, é apenas `()`. Mas como estamos em módulos separados, é necessário "passar" a variável `client` para ele ou não seria acessível. É para isso que serve nosso `bind` chique no arquivo principal!

Aqui está outro exemplo com o evento `guildMemberAdd`:

```javascript
const { Permissions } = require("discord.js");

module.exports = (client, member) => {
  const defaultChannel = member.guild.channels.cache.find(channel => channel.permissionsFor(guild.me).has(Permissions.FLAGS.SEND_MESSAGES));
  defaultChannel.send(`Bem-vindo ${member.user} a este servidor.`).catch(console.error);
}
```

Agora temos `client` e também `member` que é o argumento fornecido _por_ o evento `guildMemberAdd`.

## BÔNUS: O comando "reload"

Por causa da maneira como `require()` funciona no node, se você modificar qualquer um dos arquivos de comando em `./commands`, as mudanças não são refletidas imediatamente quando você chama esse comando novamente - porque `require()` _cacheia_ o arquivo na memória em vez de lê-lo toda vez. Embora isso seja ótimo para eficiência, significa que precisamos limpar essa versão em cache se mudarmos comandos.

O comando _Reload_ faz exatamente isso, simplesmente deleta o cache para que na próxima vez que esse comando específico seja executado, ele atualizará seu código do arquivo.

```javascript
exports.run = (client, message, args) => {
  if (!args || args.length < 1) return message.reply("Deve fornecer um nome de comando para recarregar.");
  const commandName = args[0];
  // Verificar se o comando existe e é válido
  if (!client.commands.has(commandName)) {
    return message.reply("Esse comando não existe");
  }
  // o caminho é relativo à *pasta atual*, então apenas ./filename.js
  delete require.cache[require.resolve(`./${commandName}.js`)];
  // Também precisamos deletar e recarregar o comando do Enmap client.commands
  client.commands.delete(commandName);
  const props = require(`./${commandName}.js`);
  client.commands.set(commandName, props);
  message.reply(`O comando ${commandName} foi recarregado`);
};
```

Lembre-se de que tudo isso é apenas uma versão bastante básica do manipulador de comandos GuideBot que também tem permissões, níveis, configurações por servidor e uma porção de comandos e eventos de exemplo! [Dirija-se ao Github](https://github.com/AnIdiotsGuide/guidebot/) para ver o manipulador completo.

Próximo passo é tornar este manipulador de comandos básico _melhor_ com a adição de [comandos slash](slash-commands/md).
