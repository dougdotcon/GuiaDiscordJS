# Sharding

**Sharding** é o método pelo qual o código de um bot é "dividido" em múltiplas _instâncias_ de si mesmo. Quando um bot é sharded, cada shard manipula apenas uma certa porcentagem de todos os servidores em que o bot está.

Há dificuldades adicionais ao shardar um bot que adicionam complexidade ao seu código (uma das razões pelas quais você não deveria shardar muito cedo). Você não precisa se preocupar com sharding até que seu bot atinja cerca de 2.400 servidores. VOCÊ DEVE SHARDAR até o momento em que atingir 2.500 servidores, no entanto.

## Estilos de Sharding

Há dois estilos de sharding que discutiremos: [sharding interno](sharding.md#internal-sharding) e [sharding tradicional](sharding.md#traditional-sharding). Cada um desses estilos de sharding possui benefícios dependendo da sua situação.

## Sharding Interno

`internal` sharding é o método pelo qual o código de um bot cria múltiplas conexões de shard para a API do Discord _dentro de um único processo_. Isso significa que todos os servidores, canais e usuários em um shard estarão disponíveis para outro shard via uma chamada direta (ex. `client.guilds.cache.get('GUILD_ID')`). Devido ao grande tamanho de memória que o processo único do bot crescerá usando este estilo de sharding, não é ideal para bots com muitos servidores.

### Cliente Shardado Internamente

Se você gostaria de usar isso, ajuste as opções do Cliente no seu arquivo principal do bot onde você define seu cliente assim:

```javascript
/*
    O seguinte código vai no seu arquivo principal do bot.
*/

// Incluir ShardingManager do discord.js
const { Client } = require("discord.js");

// Quando definimos nosso cliente, incluímos a propriedade "shardCount"
// e definimos como 'auto' para permitir que o cliente crie automaticamente
// o número correto de shards.
// Se você gostaria de ter um número diferente de shards, você pode
// também definir isso como um número.
const client = new Client({ shardCount: "auto" });
```

## Sharding Tradicional

`traditional` sharding é o método pelo qual o código de um bot cria processos filhos individuais via um processo gerenciador de shard principal, cada processo filho sendo um shard do bot. Ao usar este estilo de sharding, servidores, canais e usuários em um shard _não estarão_ disponíveis para outro via chamada direta (ex. `client.guilds.cache.get('GUILD_ID')`) porque cada shard está em um processo separado.

Este estilo de sharding é ideal para bots maiores, ou bots que precisam ser escaláveis para permitir crescimento futuro. O resto desta página discutirá [como fazer uso de sharding tradicional](sharding.md#example-sharding-manager-code) e [como compartilhar informações entre shards](sharding.md#sharing-information-between-shards).

Para aprender como fazer uso disso, continue lendo!

### Alertas de Sharding Tradicional

* Coleções não armazenam em cache dados de todos os shards, então você não pode pegar dados de um servidor em outro shard facilmente.
* Para fazer qualquer coisa entre shards você precisa se preocupar em usar [`fetchClientValues`](sharding.md#fetchclientvalues) e [`broadcastEval`](sharding.md#broadcasteval) (Exemplos e explicação abaixo).

  E novamente:

* Bots shardados tradicionalmente frequentemente ganham aumentos de desempenho muito marginais e podem até usar _mais memória_ devido ao uso de mais processos node.
* Se você está usando qualquer tipo de banco de dados ou conexão, múltiplos shards podem causar problemas com múltiplos processos conectando a um único ponto final.

### Código Exemplo do Gerenciador de Sharding

```javascript
/*
    O seguinte código vai em seu próprio arquivo, e você executa este arquivo
    em vez do seu arquivo principal do bot.
*/

// Incluir ShardingManager do discord.js
const { ShardingManager } = require("discord.js");

// Criar sua instância do ShardingManager
const manager = new ShardingManager("./SEU_NOME_DO_ARQUIVO_DO_BOT.js", {
    // para opções do ShardingManager veja:
    // https://discord.js.org/#/docs/main/stable/class/ShardingManager
    totalShards: "auto",
    token: "SEU_TOKEN_VAI_AQUI"
});

// Emitido quando um shard é criado
manager.on("shardCreate", (shard) => console.log(`Shard ${shard.id} lançado`));

// Criar seus shards
manager.spawn();
```

## Compartilhando Informações Entre Shards

Lembre-se de como estávamos falando sobre sharding ser um método de "dividir" o bot em múltiplas instâncias de si mesmo? Por causa disso, coisas como adicionar seus servidores totais ou obter um servidor específico não são tão simples quanto eram antes. Devemos agora usar [`fetchClientValues`](sharding.md#fetchclientvalues) ou [`broadcastEval`](sharding.md#broadcasteval) para obter informações de shards.

Essas duas funções são seu recurso essencial para obter qualquer informação de outros shards, então familiarize-se com elas!

### FetchClientValues

[`fetchClientValues`](https://discord.js.org/#/docs/main/stable/class/ShardClientUtil?scrollTo=fetchClientValues) obtém propriedades do Cliente de todos os shards. Isso é o que você deve usar quando gostaria de obter qualquer uma das propriedades aninhadas do Cliente, como `guilds.cache.size` ou `uptime`. É útil para obter coisas como tamanhos de Coleção, propriedades básicas do cliente, e informações não processadas sobre o cliente.

Exemplo:

```javascript
/*
    Exemplo de resultado de fetchClientValues() em um bot com 4.300 servidores divididos em
    4 shards.
    Assuma que isso está sendo executado no shard 0, o primeiro shard.
*/

// Se apenas obtivermos nosso client.guilds.cache.size, retornará
// apenas o número de servidores no shard em que isso está sendo executado.
console.log(client.guilds.cache.size);
// 1050

// Se gostaríamos de obter nosso client.guilds.cache.size de todos
// nossos shards, devemos fazer uso de fetchClientValues().
const res = await client.shard.fetchClientValues("guilds.cache.size");
console.log(res);
//     [
//        1050,    // shard 0
//        1100,    // shard 1
//        1075,    // shard 2
//        1075     // shard 3
//    ]
```

Digamos que você quer fazer algo como obter sua contagem total de servidores - Em um ambiente não shardado, isso seria tão simples quanto obter o `client.guilds.cache.size`. No entanto, neste caso `client.guilds.cache.size` não retornará os servidores totais em que seu bot está. Em vez disso retorna apenas o número total de servidores _neste shard_, como na primeira parte do exemplo acima.

Aqui está um exemplo de uma função que usa `fetchClientValues()` para primeiro obter, depois adicionar o número total de servidores de _todos os shards_ (ou seja, a contagem total de servidores do seu bot):

```javascript
/*
    Exemplo por ZiNc#2032

      versão discord.js 12.x
      client = new discordjs.Client()

      retorna número
*/

const getServerCount = async () => {
    // obter tamanho da coleção de servidores de todos os shards
    const req = await client.shard.fetchClientValues("guilds.cache.size");

    // retornar o valor adicionado
    return req.reduce((p, n) => p + n, 0);
}
```

{% hint style="info" %}
`fetchClientValues()` não permite que você faça uso de métodos javascript ou métodos do cliente para obter ou processar informações antes de retorná-las. Apenas permite que você obtenha informações de propriedades do cliente.
{% endhint %}

### BroadcastEval

[`broadcastEval`](https://discord.js.org/#/docs/main/stable/class/ShardClientUtil?scrollTo=broadcastEval) avalia a entrada no contexto de cada Cliente(s) do shard. Isso é o que você deve usar quando quer executar um método ou processar dados em um shard e retornar o resultado. É útil para obter informações que não estão disponíveis através de propriedades do cliente e devem ser recuperadas através do uso de métodos.

Exemplo:

```javascript
/*
    Exemplo de resultado de broadcastEval() em um bot com 4 servidores divididos em
    2 shards.
    Assuma que isso está sendo executado no shard 0, o primeiro shard.
*/

// Se apenas mapearmos nossos members.cache.size dos servidores, retornará
// apenas o members.cache.size mapeado do shard em que isso está sendo executado.
console.log(client.guilds.cache.map((guild) => guild.members.cache.size));
//        [
//            30,
//            25
//        ],

// Se gostaríamos de mapear nossos members.cache.size dos servidores em todos
// nossos shards, devemos fazer uso de broadcastEval().
// Lembre-se, isso executa no contexto do cliente, então nos referimos ao
// Cliente usando "this".
const res = await client.shard.broadcastEval((c) => c.guilds.cache.map((guild) => 
    guild.members.cache.size));
console.log(res);
//     [
//        [    // shard 0
//            30,
//            25
//        ],
//        [    // shard 1
//            55,
//            10
//        ]
//     ]
```

Digamos que você quer obter um servidor do seu cliente. Em um ambiente não shardado, você simplesmente usaria `client.guilds.cache.get('ID')` ou algo assim e então continuaria com seu código. Neste caso, no entanto, é possível que o servidor que você está tentando obter _não esteja presente no shard_. Para obter o servidor para uso, você então precisaria buscá-lo de qualquer shard em que esteja presente usando `broadcastEval()`.

Aqui está um exemplo de uma função que usa `broadcastEval()` para obter um único servidor não importa em qual shard esteja presente:

```javascript
/*
      Exemplo por ZiNc#2032

    NOTA: Propriedades do servidor buscado como "Guild.members.cache" e
    "Guild.roles.cache" não serão Managers ou Collections; essas
    propriedades serão arrays de IDs de snowflake.

      versão discord.js 12.x
      client = new discordjs.Client()

      retorna Guild
*/

const getServer = async (guildID) => {
    // tentar obter servidor de todos os shards
    const req = await client.shard.broadcastEval((c, id) => c.guilds.cache.get(id), { 
        context: guildID
    });

    // retornar Guild ou null se não encontrado
    return req.find(res => !!res) || null;
}
```

{% hint style="info" %}
`broadcastEval()` retornará apenas objetos javascript básicos, e não retornará estruturas do Discord.js como Collections, objetos Guild, objetos Member, e similares. Você precisará interagir diretamente com a API ou construir essas estruturas você mesmo após obter uma resposta do seu `broadcastEval()`.
{% endhint %}

Exemplo de um objeto Guild retornado por [`broadcastEval`](sharding.md#broadcasteval):

```javascript
const res = await client.shard.broadcastEval((c) => c.guilds.cache.map((guild) => 
    guild.members.cache.size));
console.log(res);
//     [
//        [    // qualquer shard que tenha o servidor
//          {
//            members: [
//              '123456789123456789', '123456789123456789', '123456789123456789',
//              '123456789123456789', '123456789123456789', '123456789123456789',
//              '123456789123456789', '123456789123456789', '123456789123456789',
//              '123456789123456789', '123456789123456789', '123456789123456789',
//              '123456789123456789', '123456789123456789', '123456789123456789'
//            ],
//            channels: [
//              '123456789123456789', '123456789123456789', '123456789123456789',
//              '123456789123456789', '123456789123456789', '123456789123456789',
//              '123456789123456789', '123456789123456789', '123456789123456789',
//              '123456789123456789', '123456789123456789', '123456789123456789',
//              '123456789123456789', '123456789123456789', '123456789123456789'
//            ],
//            roles: [
//              '123456789123456789', '123456789123456789', '123456789123456789',
//              '123456789123456789', '123456789123456789', '123456789123456789'
//            ],
//            emojis: [
//              '123456789123456789', '123456789123456789', '123456789123456789',
//              '123456789123456789', '123456789123456789', '123456789123456789'
//            ],
//            afkChannelID: null,
//            afkTimeout: 300,
//            applicationID: null,
//            banner: null,
//            bannerURL: null
//            createdTimestamp: 1234567891234,
//            defaultMessageNotifications: 'MENTIONS',
//            deleted: false,
//            description: null,
//            explicitContentFilter: 'MEMBERS_WITHOUT_ROLES',
//            features: [ 'ANIMATED_ICON', 'INVITE_SPLASH' ],
//            icon: 'ICON_ID',
//            iconURL: 'ICON_URL',
//            id: '123456789123456789',
//            joinedTimestamp: 1234567891234,
//            large: true,
//            memberCount: 6000,
//            mfaLevel: 1,
//            name: 'Server_Name',
//            nameAcronym: 'E',
//            ownerID: '123456789123456789',
//            premiumSubscriptionCount: 3,
//            premiumTier: 1,
//            publicUpdatesChannelID: null,
//            region: 'us-east',
//            rulesChannelID: null,
//            shardID: 0,
//            splash: 'SPLASH_ID',
//            splashURL: 'SPLASH_URL',
//            systemChannelFlags: 1,
//            systemChannelID: '123456789123456789',
//            vanityURLCode: null,
//            verificationLevel: 'MEDIUM',
//          }
//        ],
//
//      ...null // todas as outras respostas de shard
//     ]
```
