# Async / Await

Quando uma função async é chamada, ela retorna uma [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise). Quando a função async retorna um valor, a Promise será resolvida com o valor retornado. Quando a função async lança uma exceção ou algum valor, a Promise será rejeitada com o valor lançado.

Uma função async pode conter uma expressão [await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await), que pausa a execução da função async e espera pela resolução da Promise passada, e então retoma a execução da função async e retorna o valor resolvido.

### Promise

Mas, o que são Promises? O Objeto Promise é usado para computações/chamadas assíncronas, mas ao contrário de funções, elas não retornam o valor imediatamente, pois têm três estados:

* **Pending**: estado inicial, não cumprida ou rejeitada.
* **Fulfilled**: significando que a operação foi concluída com sucesso.
* **Rejected**: significando que a operação falhou.

Uma função retorna uma Promise quando você chama um método assíncrono, significa, como explicado acima, que o valor **final** não está disponível quando a função foi chamada, mas quando o objeto Promise resolve.

Um exemplo no mundo real seria:

**Você pega uma garrafa de água, abre, vira e esvazia. Então descarta.**

O problema acima é, quando você pega a garrafa de água, pode abri-la imediatamente (função sync), mas quando você a vira e esvazia, você tem que **aguardar** até que a garrafa fique vazia. Isso é uma **Função Async**.

**AoDude\#8676** propôs o seguinte exemplo:

```javascript
const perrier = require("PerrierBrandWater");
const bottle = new perrier.BottleOfWater();

bottle.open(); // operação sync
bottle.turnOverAndDrain() // operação async
    .then(emptyBottle => emptyBottle.dispose())
    .catch((err) => {
        console.error(err);
        runForYourLives();
    });

const runForYourLives = () => process.exit();
```

No exemplo acima, você abre a garrafa (operação sync, pode fazer isso na execução do código), então chama `turnOverAndDrain`, que é uma operação async. Você não sabe se a operação será executada com sucesso (a água é completamente esvaziada) ou falhará (algo aconteceu e a água não pôde ser completamente esvaziada).

Quando você chama uma função assíncrona, em [**ES6**](https://www.ecma-international.org/ecma-262/6.0/), pode usar as palavras-chave `then` e `catch`. Em algum ponto elas fazem alguma lógica.

Você abre a garrafa, vira e esvazia, **então** descarta. Mas se algo falhou, há um `erro`, você **captura** ele, exibe um erro no console e corre para salvar suas vidas.

Dentro de `then` e `catch` há funções, dentro de then, usamos a variável **emptyBottle**, que é o valor que a função **turnOverAndDrain** retorna quando resolve. E dentro de catch, **err** é o objeto de erro que a função retorna quando falha.

Não entendeu? Há um exemplo com Discord.JS:

Alguma prática, o método [client.users.fetch](https://discord.js.org/#/docs/main/stable/class/UserManager?scrollTo=fetch) retorna `Promise<User>`. Significa, quando você chama esse método, retornará uma **Promise**, resolvendo com um objeto **User** (mas também pode lançar um erro).

```javascript
client.users.fetch(id)
    .then((User) => {
        // Fazer algo com o objeto User
    })
    .catch((err) => {
        // Fazer algo com o objeto Error, por exemplo, console.error(err);
    })
```

O código acima requer um UserID, mas você não tem o objeto User já que vai recuperar informações do Discord para obter o objeto User, isso leva um tempo, uma vez que você obtém os dados, Discord.JS resolverá o método, executando a função dentro de `then`, passando o objeto `User`, descrito na documentação.

### Uso de Async/Await

Uma vez que sabemos como usar o `Objeto Promise` e sabemos como trabalhar com ele, é hora de aprender como usar Promises ES8, com `async`/`await`.

```javascript
async () => {
    const User = await client.users.fetch(id);
    // Fazer algo com o objeto User
}
```

**ESPERA O QUÊ? ISSO É TUDO?** Sim, é. No código acima, você está definindo a constante `User` como o resultado da Promise, daí a palavra-chave `await`. Neste contexto, seu código (quando executa), chama o método `client.users.fetch()`, mas vai parar ali, uma vez que a promise resolve, o valor retornado (Objeto User) é atribuído à constante User.

**Espera, temos a substituição para** `then`**, mas e se o método falhar?** Uma vantagem do Async/Await ES8 é que você pode chamar múltiplas AsyncFunctions e capturá-las todas de uma vez. Como no seguinte exemplo:

```javascript
async () => {
    try {
        const User = await client.users.fetch(id);
        const member = await guild.members.fetch(User);
        const role = guild.roles.cache.find(r => r.name === "Idiot Subscribers");
        await member.roles.add(role);
        await channel.send("Sucesso!");
    } catch (e) {
        console.error(e);
    }
}
```

No exemplo acima, você busca um usuário, uma vez que você tem o objeto User, declara como a constante `User`, então você busca um membro com o `User`, se for encontrado, obtém um cargo (método sync, não retorna um `Objeto Promise`, então você não precisa da palavra-chave `await`), adiciona o cargo ao membro e envia uma mensagem ao canal.

Se **UMA** das promises falhar, o código para de executar o resto dos métodos dentro do bloco de `try` e executará o bloco dentro de `catch`, com o objeto Error, e enviará um erro ao console.

O exemplo executa uma **Promise Chain** (executa promises, uma após a anterior), você não quer vê-las com async/await ES6. Sério. É algo que chamamos de **Callback Hell** (muitos níveis de indentação, código muito difícil de seguir, daí muito mais difícil de trabalhar...).

## Importante!

Para usar a palavra-chave `await`, você **DEVE** ter escrito a palavra-chave `async` na função cujo bloco contém o código. Por exemplo:

```javascript
const EditMessage = async (id, content) => {
    const Message = await channel.messages.fetch(id); // Async
    return Message.edit(content);
}
```

```javascript
async function EditMessage(id, content) {
    const Message = await channel.messages.fetch(id); // Async
    return Message.edit(content);
}
```

No entanto, se você tem uma função dentro de outra, por exemplo:

```javascript
const EditMessage = async (id, content) => {
    const Message = await channel.messages.fetch(id);
    setTimeout(() => {
        await Message.edit(content);
        Message.channel.send("Editado!");
    }, 5000);
}
```

Isso lançará um erro:

```javascript
        await Message.edit(content);
              ^^^^^^^
SyntaxError: Unexpected identifier
```

Este exemplo falhou porque a função dentro de `setTimeout` não tem a palavra-chave `async`.

## Documentação

Os seguintes links são do MDN (Mozilla Developer Network), eles fornecem sintaxe, descrição e exemplos.

* [JavaScript Reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference)
* [AsyncFunction (Statement)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
* [AsyncFunction (Operator)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/async_function)
* [await (Operator)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await)
* [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
