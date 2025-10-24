# Começando - Resumo

{% hint style="info" %}
**Esta é a versão resumida**. Se você deseja uma versão longa com mais explicações, por favor veja [este guia](getting-started-long-version.md)
{% endhint %}

## Criar Aplicativo e Conta do Bot

* Vá para a [Página de Aplicativos do Discord.com](https://discord.com/developers/applications/me)
* Crie um **Novo Aplicativo** e dê um nome
* Clique em **Bot**, **Adicionar Bot** e finalmente clique em **Sim, faça isso**
* Visite `https://discord.com/oauth2/authorize?client_id=APP_ID&scope=bot` , substituindo **APP\_ID** pelo **ID do Aplicativo** da página do app, para adicionar o bot ao seu servidor (ou peça a um administrador do servidor para fazer isso por você). Se você também quer comandos slash, adicione `%20applications.commands` ao final da URL acima.
* Copie o **Token Secreto** do seu bot e guarde para depois

## Software Pré-requisito

Dependendo do sistema operacional que você está usando, a instalação será um pouco diferente.

* nodejs (Versão 16.6 ou superior necessária, veja [Windows](https://nodejs.org/en/download/) ou [Linux](https://nodejs.org/en/download/package-manager/))

Depois de ter tudo isso instalado, crie uma pasta para seu projeto e instale o discord.js:

`mkdir mybot` `cd mybot` `npm i discord.js`

## Código de Exemplo

O seguinte é um bot simples ping/pong. Salve como um arquivo de texto (ex. `index.js`), substituindo a string na última linha pelo token secreto do bot que você obteve anteriormente:

```javascript
const { Client, Intents } = require("discord.js");
// Como o discord.js exporta um objeto por padrão, podemos desestruturá-lo. Leia mais aqui https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment
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

## Lançando o Bot

No seu prompt de comando, de dentro da pasta onde `index.js` está localizado, execute com:

`node index.js`

Se nenhum erro for mostrado, o bot deve entrar no(s) servidor(es) em que você o adicionou.

## Recursos

* [Documentação do Discord.js](http://discord.js.org) : Pelo amor de tudo que é (in)sagrado, **leia a documentação**. Sim, será estranho no início se você não está acostumado com "documentação de desenvolvedor", mas contém muita informação sobre cada recurso da API. Combine isso com os exemplos acima para ver a API em contexto.
* [An Idiot's Guide](https://www.youtube.com/c/AnIdiotsGuide) é outro ótimo canal com mais material. Os guias do York são ótimos e ele continua atualizando-os.
* [Evie.Codes no YouTube](https://www.youtube.com/channel/UCvQubaJPD0D-PSokbd5DAiw): Se você prefere vídeo a palavras, a série no YouTube da Evie (que é boa, embora não seja mais mantida com novos vídeos!) te leva a começar com bots.
* [Servidor Oficial do An Idiot's Guide](https://discord.gg/vXVxsAjSMF): O servidor oficial do An Idiot's Guide. Cheio de usuários amigáveis e úteis!
* [Servidor Oficial do Discord.js](https://discord.gg/bRCvFy9): O servidor oficial tem várias pessoas competentes para ajudá-lo, e a equipe de desenvolvimento também está lá!
