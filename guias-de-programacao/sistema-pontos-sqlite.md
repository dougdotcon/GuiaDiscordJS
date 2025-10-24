# Sistema de Pontos Baseado em SQLite

Esse é o foco deste guia: vamos criar o sistema de pontos com SQLite. O núcleo deste sistema é usar o pacote `better-sqlite3` que você pode obter de [npmjs.com](https://npmjs.com/package/better-sqlite3).

## Instalação

{% hint style="warning" %}
**Pré-Requisitos**: `better-sqlite3`, similarmente a muitos módulos, é compilado usando `node-gyp` que tem 2 requisitos muito importantes: Python 2.7 e as Ferramentas de Build C++. Para windows, abra um prompt de comando Elevado (Administrador) e execute o seguinte PRIMEIRO, antes de instalar better-sqlite3: `npm i --vs2015 -g windows-build-tools`. Para linux, você precisa de `sudo apt-get install build-essential` e precisa descobrir como instalar Python 2.7 (NÃO Python 3!) no seu sistema.
{% endhint %}

Para este guia funcionar, você primeiro precisa se certificar de ter os módulos adequados instalados. Vamos assumir que você já tem `discord.js` instalado, e vamos direto para instalar o sqlite e sua dependência node-gyp:

```text
npm i node-gyp better-sqlite3
```

## Configurando a Tabela

Você tem SQLite instalado, agora o que queremos em nossa tabela? Vou te dizer!

Para este sistema de pontos de exemplo queremos o ID do usuário, pontos e nível para estar em conformidade com os Termos de Serviço do Discord Developer. Não vou entrar muito fundo na gíria em torno de SQL e SQLite, mas as tabelas são feitas de linhas e colunas de dados. Entendeu? Bom, continuando!

Nosso ponto de partida é um manipulador de mensagens muito básico com comandos pré-existentes - como o que vemos na página [Comando com Argumentos](../primeiro-bot/comando-com-argumentos.md) deste guia. O código é assim:

```javascript
const { Client, Intents } = require("discord.js");
const client = new Client({
  intents: [Intents.FLAGS.GUILDS, Intents.FLAGS.GUILD_MESSAGES]
});
const config = require("./config.json"); // Contém o prefixo, e token!

client.on("ready", () => {
  console.log("Estou pronto!");
});

client.on("messageCreate", message => {
  if (message.author.bot) return;
  // É aqui que vamos colocar nosso código.
  if (message.content.indexOf(config.prefix) !== 0) return;

  const args = message.content.slice(config.prefix.length).trim().split(/ +/g);
  const command = args.shift().toLowerCase();

  // Código específico de comando aqui!
});

client.login(config.token);
```

Agora que temos isso devemos `require` sqlite e fazer uso dele, coloque o seguinte abaixo de `const client`:

```javascript
const SQLite = require("better-sqlite3");
const sql = new SQLite("./scores.sqlite");
```

Temos uma pequena ressalva - realmente não queremos reagir em Mensagens Diretas, então todo o nosso código estará em um bloco que verifica isso. Não queremos apenas ignorar DMs porque nosso próprio bot pode ter comandos DM!

```javascript
client.on("messageCreate", message => {
  if (message.author.bot) return;
  if (message.guild) {
    // É aqui que vamos colocar nosso código.
  }
  // Resto do manipulador de mensagens
});
```

Seu código deve parecer assim agora:

```javascript
const { Client } = require("discord.js");
const client = new Client();
const config = require("./config.json");
const SQLite = require("better-sqlite3");
const sql = new SQLite("./scores.sqlite");

client.on("ready", () => {
  console.log("Estou pronto!");
});

client.on("messageCreate", message => {
  if (message.author.bot) return;
    if (message.guild) {
    // É aqui que vamos colocar nosso código.
  }
  if (message.content.indexOf(config.prefix) !== 0) return;

  const args = message.content.slice(config.prefix.length).trim().split(/ +/g);
  const command = args.shift().toLowerCase();

  // Código específico de comando aqui!
});

client.login(config.token);
```

## Vestindo a Tabela

Uma coisa importante com SQLite é que ele só criará tabelas se pedirmos para isso. Isso significa, temos que nos certificar de que a tabela existe. Faremos isso no nosso evento `ready`, então ele executará apenas uma vez quando iniciarmos o bot. Agora, estou fazendo _um pouco de mágica_ aqui, que inclui definir alguns toggles de banco de dados que tornam o SQLite mais rápido. Se você quiser procurar "pragma synchronous" e "pragma journal mode wal", sinta-se à vontade para aprender o que são, mas essas são boas configurações de _produção_ para ter.

E, outro pedaço de mágica aqui, é que podemos _preparar_ algumas declarações antecipadamente, e simplesmente _executá-las_ com valores específicos mais tarde. Este é um conceito mais avançado de SQL, mas deve ser fácil de seguir mesmo se você não estiver familiarizado com isso. Então aqui está o código para o evento ready:

```javascript
client.on("ready", () => {
  // Verificar se a tabela "points" existe.
  const table = sql.prepare("SELECT count(*) FROM sqlite_master WHERE type='table' AND name = 'scores';").get();
  if (!table['count(*)']) {
    // Se a tabela não estiver lá, criá-la e configurar o banco de dados corretamente.
    sql.prepare("CREATE TABLE scores (id TEXT PRIMARY KEY, user TEXT, guild TEXT, points INTEGER, level INTEGER);").run();
    // Garantir que a linha "id" seja sempre única e indexada.
    sql.prepare("CREATE UNIQUE INDEX idx_scores_id ON scores (id);").run();
    sql.pragma("synchronous = 1");
    sql.pragma("journal_mode = wal");
  }

  // E então temos duas declarações preparadas para obter e definir os dados de pontuação.
  client.getScore = sql.prepare("SELECT * FROM scores WHERE user = ? AND guild = ?");
  client.setScore = sql.prepare("INSERT OR REPLACE INTO scores (id, user, guild, points, level) VALUES (@id, @user, @guild, @points, @level);");
});
```

## Você ganha pontos, e você ganha pontos e TODO MUNDO GANHA PONTOS.

Agora podemos ir em frente e começar a usar o banco de dados para recuperar e armazenar dados de pontos. Faremos isso dentro do Manipulador de Mensagens, e nosso primeiro passo é tentar recuperar os pontos existentes para um usuário dentro da tabela de pontos, que seria assim:

```javascript
let score = client.getScore.get(message.author.id, message.guild.id);
```

Importante, se o bot nunca viu este usuário antes eles não terão nenhum dado, o que significa que temos que "definir" seus valores iniciais. Isso pode ser feito com uma condição simples:

```javascript
if (!score) {
  score = {
    id: `${message.guild.id}-${message.author.id}`,
    user: message.author.id,
    guild: message.guild.id,
    points: 0,
    level: 1
  }
}
```

## Mantendo Pontuação

Agora que temos nosso valor "scores" inicial, podemos fazer duas coisas: primeiro, incrementar os pontos. E segundo, calcular o nível do usuário.

```javascript
// Incrementar a pontuação
score.points++;

// Calcular o nível atual através de MATEMÁTICA OMG AJUDA.
const curLevel = Math.floor(0.1 * Math.sqrt(score.points));

// Verificar se o usuário subiu de nível, e deixá-los saber se tiverem:
if (score.level < curLevel) {
  // Subiu de nível!
  score.level++;
  message.reply(`Você subiu para o nível **${curLevel}**! Não é legal?`);
}
```

E finalmente, precisamos salvar tudo isso de volta no banco de dados. SQLite tem um grande recurso "secreto" chamado "INSERT OR REPLACE" e já criamos uma declaração preparada para isso, chamada `client.setScore`. Isso basicamente _atualizará uma linha existente com o mesmo_ `id`_, ou criará uma nova linha se o_ `id` _não for encontrado_. Isso explica por que temos o campo `id` lá, caso você estivesse se perguntando.

```javascript
// Isso parece super simples porque está chamando a declaração preparada!
client.setScore.run(score);
```

Vamos juntar tudo. Seu código deve parecer assim agora.

```javascript
const { Client, MessageEmbed } = require("discord.js");
const client = new Client();
const config = require("./config.json");
const SQLite = require("better-sqlite3");
const sql = new SQLite("./scores.sqlite");

client.on("ready", () => {
  // Verificar se a tabela "points" existe.
  const table = sql.prepare("SELECT count(*) FROM sqlite_master WHERE type='table' AND name = 'scores';").get();
  if (!table['count(*)']) {
    // Se a tabela não estiver lá, criá-la e configurar o banco de dados corretamente.
    sql.prepare("CREATE TABLE scores (id TEXT PRIMARY KEY, user TEXT, guild TEXT, points INTEGER, level INTEGER);").run();
    // Garantir que a linha "id" seja sempre única e indexada.
    sql.prepare("CREATE UNIQUE INDEX idx_scores_id ON scores (id);").run();
    sql.pragma("synchronous = 1");
    sql.pragma("journal_mode = wal");
  }

  // E então temos duas declarações preparadas para obter e definir os dados de pontuação.
  client.getScore = sql.prepare("SELECT * FROM scores WHERE user = ? AND guild = ?");
  client.setScore = sql.prepare("INSERT OR REPLACE INTO scores (id, user, guild, points, level) VALUES (@id, @user, @guild, @points, @level);");
});

client.on("messageCreate", message => {
  if (message.author.bot) return;
  let score;
  if (message.guild) {
    score = client.getScore.get(message.author.id, message.guild.id);
    if (!score) {
      score = { id: `${message.guild.id}-${message.author.id}`, user: message.author.id, guild: message.guild.id, points: 0, level: 1 }
    }
    score.points++;
    const curLevel = Math.floor(0.1 * Math.sqrt(score.points));
    if (score.level < curLevel) {
      score.level++;
      message.reply(`Você subiu para o nível **${curLevel}**! Não é legal?`);
    }
    client.setScore.run(score);
  }
  if (message.content.indexOf(config.prefix) !== 0) return;

  const args = message.content.slice(config.prefix.length).trim().split(/ +/g);
  const command = args.shift().toLowerCase();

  // Código específico de comando aqui!
});

client.login(config.token);
```

## Deixar um usuário visualizar seu nível e pontos

Agora que temos o núcleo deste código feito, precisamos adicionar alguns comandos, que podemos adicionar abaixo de todo o nosso código no manipulador de mensagens. Já separamos os argumentos e comandos, então isso será bem fácil, especialmente já que já carregamos a `score`, calculamos os pontos, e o nível!

```javascript
if (command === "points") {
  return message.reply(`Você atualmente tem ${score.points} pontos e está no nível ${score.level}!`);
}
```

### Adendo: Leaderboard e comandos Give!

Aqui estão alguns comandos rápidos e fáceis que você pode usar, assumindo que o código acima é usado e isso ainda está acontecendo no mesmo arquivo:

```javascript
// Você pode modificar o código abaixo para remover pontos do usuário mencionado também!
if (command === "give") {
  // Limitado ao dono do servidor - ajuste à sua própria preferência!
  if (!message.author.id === message.guild.ownerId) return message.reply("Você não é meu chefe, você não pode fazer isso!");

  const user = message.mentions.users.first() || client.users.cache.get(args[0]);
  if (!user) return message.reply("Você deve mencionar alguém ou dar o ID deles!");

  const pointsToAdd = parseInt(args[1], 10);
  if (!pointsToAdd) return message.reply("Você não me disse quantos pontos dar...");

  // Obter seus pontos atuais.
  let userScore = client.getScore.get(user.id, message.guild.id);

  // É possível dar pontos a um usuário que não vimos, então precisamos iniciar padrões aqui também!
  if (!userScore) {
    userScore = { id: `${message.guild.id}-${user.id}`, user: user.id, guild: message.guild.id, points: 0, level: 1 }
  }
  userScore.points += pointsToAdd;

  // Também queremos atualizar o nível deles (mas não os notificaremos se mudar)
  let userLevel = Math.floor(0.1 * Math.sqrt(score.points));
  userScore.level = userLevel;

  // E salvamos!
  client.setScore.run(userScore);

  return message.channel.send(`${user.tag} recebeu ${pointsToAdd} pontos e agora está em ${userScore.points} pontos.`);
}

if (command === "leaderboard") {
  const top10 = sql.prepare("SELECT * FROM scores WHERE guild = ? ORDER BY points DESC LIMIT 10;").all(message.guild.id);

    // Agora balance e mostre! (como um embed legal também!)
  const embed = new MessageEmbed()
    .setTitle("Leaderboard")
    .setAuthor(client.user.username, client.user.avatarURL())
    .setDescription("Nossos top 10 líderes de pontos!")
    .setColor(0x00AE86);

  for (const data of top10) {
    embed.addFields({ name: client.users.cache.get(data.user).tag, value: `${data.points} pontos (nível ${data.level})` });
  }
  return message.channel.send({ embed: [embed] });
}
```
