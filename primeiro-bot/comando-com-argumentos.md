# Comando com Argumentos

## Criando um array de argumentos

A primeira coisa que precisamos fazer para usar argumentos é separá-los. Um comando com argumentos normalmente se pareceria com isto: `!mycommand arg1 arg2 arg3`

Nisso, precisamos fazer 3 coisas:

* Remover o prefixo
* Pegar a parte do _comando_ (`mycommand`)
* Pegar o _array_ de _argumentos_ que será:

`["arg1", "arg2", "arg3"]`

Na grande maioria do código que vi, os argumentos são _divididos_ no início do código, e cada comando juntará o array de argumentos conforme necessário dentro do código do comando.

Na minha experiência, a melhor (e mais eficiente) maneira de separar todas essas coisas são as seguintes 2 linhas de código:

```javascript
const args = message.content.slice(prefix.length).trim().split(/ +/g);
const command = args.shift().toLowerCase();
```

Vamos quebrar isso no que isso _realmente_ faz, linha por linha.

* `.slice(prefix.length)` remove o prefixo como `!` ou `+` do conteúdo da mensagem, deixando `mycommand arg1 arg2 arg3`.
* `.trim()` garante que não há espaços extras antes/depois do texto.
* `.split(/ +/g)` divide a string por _um ou muitos espaços_. Por que não apenas por espaço? Porque às vezes especialmente no mobile, você pode ter um espaço extra antes ou depois de menções, ou simplesmente um espaço extra por engano. Isso significa que `mycommand    arg1  arg2        arg3` funcionará tão bem quanto se tivessem apenas 1 espaço.

Na segunda linha:

* `args.shift()` onde `shift()` **remove um elemento do array** e o retorna. Isso nos dá `mycommand` que é retornado, e o array `args` se torna apenas `['arg1', 'arg2','arg3']`
* `.toLowerCase()` para que nosso comando esteja sempre em minúsculas, significando que `!Ping`, `!ping` e `!PiNg` funcionarão.

## Usando a variável `command` corretamente

Então agora que temos nossa variável `command`, não precisamos mais usar `if (message.content.startsWith(prefix+'command'))` para cada comando. Podemos simplificar isso olhando apenas para a própria variável `command`. Por exemplo, estes 2 comandos muito básicos:

```javascript
if (command === 'ping') {
  message.channel.send('Pong!');
} else

if (command === 'blah') {
  message.channel.send('Meh.');
}
```

Agora isso não está simplesmente tão bonito e limpo? Eu amo isso!

Alternativamente, você também pode usar isso em um bloco de comando switch/case (eu não gosto, mas cada um com o seu, certo?):

```javascript
switch (command) {
  case "ping" :
    message.channel.send('Pong!');
    break;
  case "blah" :
    message.channel.send('Meh.');
    break;
}
```

Aqui está um exemplo completo que é muito frequentemente o manipulador de comandos que usei como base para construir:

```javascript
client.on("messageCreate", message => {
  if (message.author.bot) return;
  // É aqui que vamos colocar nosso código.
  if (message.content.indexOf(config.prefix) !== 0) return;

  const args = message.content.slice(config.prefix.length).trim().split(/ +/g);
  const command = args.shift().toLowerCase();

  if (command === 'ping') {
    message.channel.send('Pong!');
  } else

  if (command === 'blah') {
    message.channel.send('Meh.');
  }
});
```

## Trabalhando com os argumentos

Ok vamos chegar à essência desta página: realmente usar o array `args` em alguns exemplos de comandos.

O primeiro é um comando perfeitamente inútil, mas pela minha vida, não consigo pensar em um comando realmente simples usando apenas argumentos de uma palavra. Então aqui está um comando ASL ridículo:

```javascript
if (command === "asl") {
  let age = args[0]; // Lembre-se que arrays são baseados em 0!.
  let sex = args[1];
  let location = args[2];
  message.reply(`Olá ${message.author.username}, vejo que você é um ${age} anos de idade ${sex} de ${location}. Quer namorar?`);
}
```

E se você quiser ser **realmente** chique com ECMAScript 6, aqui está um incrível:

```javascript
if (command === "asl") {
  let [age, sex, location] = args;
  message.reply(`Olá ${message.author.username}, vejo que você é um ${age} anos de idade ${sex} de ${location}. Quer namorar?`);
}
```

{% hint style="info" %}
Isso é chamado de [Desestruturação](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) e é incrível!
{% endhint %}

## Pegando Menções

Outra maneira de usar argumentos, quando o comando deve ter como alvo um usuário específico (ou usuários), é usar _Menções_. Por exemplo, expulsar shit-posters irritantes com `!kick @Xx_SniperBitch_xX @UselessIdiot` pode ser feito com facilidade, em vez de tentar pegar o ID ou o nome deles.

No contexto do manipulador de evento `message`, todas as menções em uma mensagem fazem parte do objeto `message.mentions`. Este objeto então contém múltiplas [Coleções](../understanding/collections.md) de diferentes tipos de menção. Aqui estão os vários tipos de menção disponíveis:

* `message.mentions.members` contém todas as @menções como objetos [GuildMember](https://discord.js.org/#/docs/main/stable/class/GuildMember).
* `message.mentions.users` contém todas as @menções como objetos [User](https://discord.js.org/#/docs/main/stable/class/User).
* `message.mentions.roles` contém todas as menções de @cargo como objetos [Role](https://discord.js.org/#/docs/main/stable/class/Role).
* `message.mentions.channels` contém todas as menções de #canal como objetos [TextChannel](https://discord.js.org/#/docs/main/stable/class/TextChannel) ou [VoiceChannel](https://discord.js.org/#/docs/main/stable/class/VoiceChannel).

Cada um desses são coleções, então qualquer método de coleção pode ser usado neles. O método mais comum para usar em menções é .first() que obtém a primeira menção, já que frequentemente há apenas uma delas.

Vamos construir um comando `kick` rápido e sujo, então. Sem tratamento de erros ou verificações de mod - direto ao ponto! (_Cul Sec_, como os franceses diriam):

```javascript
// Expulsar um único usuário na menção
if (command === "kick") {
  let member = message.mentions.members.first();
  member.kick();
}
```

Isso seria chamado com, por exemplo, `!kick @AnnoyingUser23`

## Argumentos de comprimento variável

Vamos tornar o comando kick acima um pouco melhor. Como o Discord agora suporta _razões_ de expulsão nos Logs de Auditoria, o comando `kick()` do Discord.js também suporta um argumento `reason` opcional. Mas, como a razão pode ter múltiplas palavras, precisamos _juntar_ todas essas palavras.

Então vamos fazer isso agora, com o que já aprendemos, e um pouco extra:

```javascript
if (command === "kick") {
  let member = message.mentions.members.first();
  let reason = args.slice(1).join(" ");
  member.kick(reason);
}
```

Então, a razão é obtida removendo os primeiros elementos (a menção, que parece `<@1234567489213>`) e juntando novamente o resto dos elementos do array com um espaço.

Para usar este comando, um usuário faria algo como: `!kick @SuperGamerDude Troll Óbvio, shit-posting`.

Aqui está outro exemplo, com um comando super simples, o comando `say`. Ele faz o bot dizer o que você acabou de enviar e depois deleta sua mensagem:

```javascript
if (command === "say"){
  let text = args.join(" ");
  message.delete();
  message.channel.send(text);
}
```

{% hint style="info" %}
Se você está pensando, "E se eu tiver mais de um argumento com espaços?", sim, esse é um problema mais difícil. Idealmente, se você precisar de mais de um argumento com espaços nele, não use espaços para dividir os argumentos. Por exemplo, `!newtag First Var Second Var Third Var` não funcionará. Mas `!newtag First Var;Second Var;Third Var;` poderia funcionar removendo o comando, dividindo por `;` e depois dividindo por espaço. Não para os fracos de coração!
{% endhint %}

## Indo um passo além

Agora, definitivamente sempre há espaço para alguma otimização e código melhor. Neste ponto, "analisar argumentos" se torna algo que você pode perceber que é necessário para _todos_ os seus comandos, e escrever "(command === 'thing')" para cada comando é chato e entediante. Então, como seu próximo passo, considere dar uma olhada em fazer [Um Manipulador de Comandos Básico](manipulador-comandos-basico.md). Isso **grandemente** simplifica a criação de novos comandos.
