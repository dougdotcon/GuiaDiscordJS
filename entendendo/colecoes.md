# Coleções

Nesta página vamos explorar Coleções e como usá-las para obter dados de várias partes da API.

Uma **Coleção** é uma _classe utilitária_ que armazena dados. Coleções são a estrutura de dados Map() do Javascript com métodos utilitários adicionais. Isso é usado em todo o discord.js em vez de Arrays para qualquer coisa que tenha um ID, para melhor desempenho significativo e facilidade de uso.

Exemplos de Coleções incluem:

* `client.users.cache`, `client.guilds.cache`, `client.channels.cache`
* `guild.channels.cache`, `guild.members.cache`
* logs de mensagens (no callback de `messages.fetch()`)
* `client.emojis.cache`

## Obtendo por ID

Muito simplesmente, para obter qualquer coisa por ID você pode usar `Collection.get(id)`. Por exemplo, obter um canal pode ser `client.channels.cache.get("81385020756865024")`. Obter um usuário também é trivial: `client.users.cache.get("139412744439988224")`

## Encontrando por chave

Se você não tem o ID mas apenas alguma outra propriedade, pode usar `find()` para buscar por propriedade:

`let guild = client.guilds.cache.find(guild => guild.name === "discord.js - imagine a bot");`

O _primeiro_ resultado que retorna `true` dentro da função será retornado. A ideia genérica disso é:

`let result = <Collection>.find(item => item.property === "um valor")`

Você também pode estar olhando para outros dados, propriedades não no nível superior, etc. Sua imaginação é o limite.

Quer um ótimo exemplo? Aqui está obtendo o primeiro cargo que corresponde a um de 4 nomes de cargo:

```javascript
const acceptedRoles = ["Mod", "Moderator", "Staff", "Mod Staff"];
const modRole = member.roles.cache.find(role => acceptedRoles.includes(role.name));
if (!modRole) return "Nenhum cargo encontrado";
```

{% hint style="info" %}
Não precisa retornar o cargo real? `.some()` pode ser o que você precisa. É mais rápido que find, mas só retornará um booleano true/false se encontrar algo:
{% endhint %}

```javascript
const hasModRole = member.roles.cache.some(role => acceptedRoles.includes(role.name));
// hasModRole é booleano.
```

## Filtragem personalizada

_Coleções_ também têm uma maneira personalizada de filtrar seu conteúdo com uma função anônima:

`let large_guilds = client.guilds.cache.filter(g => g.memberCount > 100);`

`filter()` retorna uma nova coleção contendo apenas itens onde o filtro retornou `true`, neste caso servidores com mais de 100 membros.

## Mapeando Campos

Uma coisa ótima que você pode fazer com uma coleção é obter dados específicos dela com `map()`, que é útil ao listar coisas. `<Collection>.map()` recebe uma função que retorna uma string. Seu resultado é um array de todas as strings retornadas por cada item. Aqui está um exemplo: vamos obter uma lista completa de todos os servidores em que um bot está, por nome:

```javascript
const guildNames = client.guilds.cache.map(g => g.name).join("\n")
```

Como `.join()` é um método de array, que liga todas as entradas juntas, obtemos uma boa lista de todos os servidores, com uma quebra de linha entre cada um. Legal!

Também podemos obter uma string mais personalizada. Vamos fingir que a propriedade `user.tag` não existe, e queríamos obter todos os user#discriminator em nosso bot. Aqui está como faríamos (usando literais de template incríveis):

```javascript
const tags = client.users.cache.map(u => `${u.username}#${u.discriminator}`).join(", ");
```

## Combinando e Encadeando

Em muitos casos você definitivamente pode encadear métodos juntos para código realmente limpo. Por exemplo, esta é uma lista delimitada por vírgulas de todos os servidores pequenos em um bot:

```javascript
const smallGuilds = client.guilds.cache.filter(g => g.memberCount < 10).map(g => g.name).join("\n");
```

## Mais Dados!

Para ver **todos** os Métodos de Coleção do Discord.js, por favor [consulte a documentação](https://discord.js.org/#/docs/collection/main/general/welcome). Como Collection estende Map(), você também precisará consultar [esta página incrível do mdn](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Map) que descreve os métodos nativos - mais notavelmente `.forEach()`, `.has()`, etc.
