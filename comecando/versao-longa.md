# Começando - Versão Longa

Então, você quer escrever um bot e conhece um pouco de JavaScript, ou talvez até node.js. Você quer fazer coisas legais como um bot de música, comandos de tag, buscas aleatórias de imagens, toda a parafernália. Bem, você está no lugar certo!

{% hint style="info" %}
**Esta é a versão longa** com muita conversa inútil, piadas e explicações.  
Aqui está a [versão resumida (TL;DR)](getting-started-tl-dr.md)
{% endhint %}

Este tutorial vai te guiar pelos primeiros passos de criar um bot, configurá-lo, fazê-lo funcionar e adicionar alguns comandos a ele.

## Passo 1: Criando seu Aplicativo e Conta do Bot

O primeiro passo para criar um bot é criar seu próprio _aplicativo_ Discord. O bot usará a API do Discord, que requer a criação de uma conta para fins de autenticação. Não se preocupe, é super simples.

### Criando a Conta do Aplicativo

Para criar o aplicativo, vá para a [página de aplicativos do Discord](https://discord.com/developers/applications/). Assumindo que você está logado (se não estiver, faça isso agora), você chegará a uma página que se parece com isto:

![A página de aplicativos](../.gitbook/assets/gs-application-page.png)

Clique em (você adivinhou!) **Novo Aplicativo**. Isso traz o seguinte modal, no qual você deve simplesmente inserir um nome para o _aplicativo_ (este será o nome de usuário inicial do bot). Clique em **Criar** que criará o aplicativo em si.

![Modal de novo aplicativo](../.gitbook/assets/gs-new-application.png)

O **ID do Aplicativo** na página será o ID de usuário do seu bot. A descrição do aplicativo é usada na seção `Sobre mim` dos bots. Então sinta-se à vontade para adicionar uma descrição do seu bot em menos de 190 caracteres.

{% hint style=" tips %} Embora a página indique claramente um máximo de 400, apenas 190 serão exibidos na seção `Sobre mim`.{% endhint %}

![O aplicativo criado](../.gitbook/assets/gs-created-application.png)

### Criar a Conta do Bot

Depois de criar o aplicativo, precisamos criar o **Usuário Bot**. Vá para a seção **Bot** à esquerda e você será recebido com a seguinte tela.

![Construir um bot](../.gitbook/assets/gs-making-bot.png)

Finalmente clique em **Adicionar Bot**, depois em **Sim, faça isso** para criar seu bot.

![Tem certeza?](../.gitbook/assets/gs-add-bot-modal.png)

Há algumas coisas que você pode mudar aqui e mais importante o token.

![Fazendo o bot](../.gitbook/assets/gs-bot-created.png)

* `Ícone` para mudar o avatar do bot (também pode ser feito com discord.js)
* `Nome de Usuário` para mudar o nome de usuário do seu bot no Discord (isso também pode ser feito através de código).
* `Token` Este é o token do seu bot, que será usado ao conectar ao discord. Veja [a seção abaixo](getting-started-long-version.md#getting-your-bot-token) para detalhes.
* `Bot Público` Isso alterna a capacidade de outros usuários adicionarem seu bot ao servidor deles. Você pode desligar isso durante o desenvolvimento para evitar que usuários aleatórios o convidem.
* `Requer Oauth2 Code Grant` Não marque isso. Apenas não. Não é útil para você e causará problemas se você ligar.
* `Privileged Gateway Intents` Agora isso é importante, se seu bot está verificando dados de presença ou baixando a lista de membros, você precisará alternar um ou ambos destes, por enquanto não são necessários. Mas por favor note, se seu bot atingir 100 servidores você precisará estar na whitelist e verificado para usar estes.

### Adicionar seu Bot a um Servidor

Ok então, isso pode ser um pouco cedo para fazer isso, mas não importa muito - mesmo que você não tenha escrito uma única linha de código para seu bot, você já pode "convidar" ele para um servidor. Para adicionar um bot, você precisa de permissões _Gerenciar Servidor_ ou _Administrador_ nesse servidor. Esta é a **única** maneira de adicionar um bot, ele não pode usar links de convite ou quaisquer outros métodos.

Para gerar o link, clique em **OAuth2** na página do app (está acima de `Bot`), e role para baixo até **Escopos**. Marque o escopo `bot` para gerar um link, se você está planejando adicionar comandos slash certifique-se de clicar em `applications.commands` também.

Normalmente, bots são convidados com as _permissões_ específicas que são dadas ao cargo do bot que não podem ser removidas a menos que você expulse e reconvide o bot. Isso é opcional, mas você pode definir essas permissões na página **Bot**, rolando para baixo até a seção **Permissões do Bot**. Marque quaisquer permissões que seu bot requer. Isso modifica o link de convite acima, que você pode então compartilhar.

Depois que você tiver o link, pode copiá-lo para uma janela do navegador e visitá-lo. Quando você faz isso, mostra uma janela deixando você escolher o servidor onde adicionar o bot, simplesmente selecione o servidor e clique em **Autorizar**.

![Convidando o bot](../.gitbook/assets/gs-invite-bot.png)

{% hint style="info" %}
Você precisa estar logado no Discord no navegador com sua conta para ver uma lista de servidores. Você só pode adicionar um bot a servidores onde você tem permissões **Gerenciar Servidor** ou **Administrador**.
{% endhint %}

### Obtendo seu Token do Bot

{% hint style="danger" %}
Ok então, **grande aviso chamativo**, **PRESTE ATENÇÃO**. Esta próxima parte é realmente, realmente importante: O **token** do seu bot deve ser **SECRETO**. É a maneira pela qual seu bot se autentica com o servidor Discord da mesma forma que você faz login no Discord com um nome de usuário e senha. **Revelar seu token é como colocar sua senha na internet**, e qualquer pessoa que conseguir este token pode usar a conexão do **seu** bot para fazer coisas. Como deletar todas as mensagens do seu servidor e banir todos. Se seu token chegar à internet, **altere-o imediatamente**. Isso inclui colocá-lo no pastebin/hastebin, tê-lo em um repositório público do github, exibir uma captura de tela dele, qualquer coisa. **ENTENDEU? ÓTIMO!**, o Github fez parceria com o Discord para invalidar seu token se for encontrado dentro do seu repositório de código e te enviar uma mensagem via uma mensagem `System` no Discord.
{% endhint %}

Com esse aviso fora do caminho, vamos para o próximo passo. O Token, como acabei de mencionar, é a maneira pela qual o bot se autentica. Para obtê-lo, vá para a seção **Bot** da página do app, depois clique em **Copiar** para copiá-lo para a área de transferência. Você também pode _visualizar_ seu token aqui se desejar. Não esquecendo aquela sempre importante chave `Regenerar` se seu token for comprometido:

![NUNCA COMPARTILHE SEU TOKEN! Isso não pode ser exagerado.](../.gitbook/assets/gs-copy-token.png)

## Passo 2: Preparando seu Ambiente de Programação

Isso pode parecer óbvio, mas vou dizer mesmo assim: Você não pode simplesmente encher o notepad.exe com código de bot e esperar que funcione. Para usar discord.js você precisará de algumas coisas instaladas. No mínimo:

* Obtenha Node.js versão 16.6 ou superior (versões anteriores não são suportadas). [Download para Windows](https://nodejs.org/en/download/) ou se você está em uma distro linux, via [gerenciador de pacotes](https://nodejs.org/en/download/package-manager/).
* Obtenha um editor de código real. Não use notepad ou notepad++, eles não são suficientes. [VS Code](https://www.visualstudio.com/en-us/products/code-vs.aspx) , [Sublime Text 3](https://www.sublimetext.com/3) e [Atom](https://atom.io/) são frequentemente recomendados.

Depois que você tiver o software necessário, o próximo passo é preparar um _espaço_ para seu código. Por favor não coloque seus arquivos na sua área de trabalho é... insalubre. Se você tem mais de um disco rígido ou partição, você poderia criar um lugar especial para seu projeto de desenvolvimento. O meu, por exemplo, é `D:\develop\` , e meu bot é `D:\develop\guide-bot\` . Depois que você criou uma pasta, abra seu CLI (interface de linha de comando) nessa pasta. Usuários Linux, vocês sabem como. Usuários Windows, aqui está um truque: `SHIFT + Botão Direito` na pasta, depois escolha o comando "secreto" **Abrir janela do PowerShell aqui**. Mágica!

E agora pronto para o próximo passo!

## Instalando Discord.js

Então você tem seu CLI pronto para ir, em uma pasta vazia, e você só quer começar a programar. Ok, espere um último segundo: vamos instalar discord.js. Mas primeiro vamos inicializar esta pasta com NPM, que garantirá que qualquer módulo instalado estará aqui, e em nenhum outro lugar. Simplesmente execute `npm init -y` e depois pressione Enter. Um novo arquivo é criado chamado `package.json`, [clique aqui](https://docs.npmjs.com/files/package.json) para mais informações sobre isso.

E agora instalamos Discord.js através do NPM, o Gerenciador de Pacotes Node:

`npm i discord.js` _No momento da escrita deste texto v13 ainda não foi lançado._

![Instalando os pacotes](../.gitbook/assets/gs-installing-discordjs.gif)

Isso levará alguns segundos e exibirá muitas coisas na tela. A menos que você tenha uma mensagem vermelha grande dizendo que não funcionou, ou pacote não encontrado, ou qualquer coisa, você está pronto para ir. Se você olhar sua pasta, notará que há uma nova pasta criada aqui: `node_modules` . Isso contém todos os pacotes instalados para seu projeto.

## Fazendo seu Primeiro Bot Funcionar

{% hint style="info" %}
Eu honestamente considero que se você não entende o código que está prestes a ver, programar um bot pode não ser para você. Se você não entende a seguinte amostra, por favor vá para [CodeAcademy](https://www.codecademy.com/learn/javascript) e aprenda Javascript primeiro. Eu imploro: pare, largue e role.
{% endhint %}

Ok finalmente, estamos prontos para começar a programar. \o/ Vamos dar uma olhada no mais básico dos exemplos, o bot ping-pong. Aqui está o código completo:

```javascript
const { Client, Intents } = require("discord.js");
// O Client e Intents são desestruturados do discord.js, já que ele exporta um objeto por padrão. Leia sobre desestruturação aqui https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment
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

{% hint style="info" %}
A variável `client` aqui é usada como um exemplo para representar a classe [&lt;Client&gt;](https://discord.js.org/#/docs/main/stable/class/Client). Algumas pessoas chamam de `bot`, mas você tecnicamente pode chamar do que quiser. Eu recomendo ficar com `client` no entanto!
{% endhint %}

Ok vamos só... realmente fazer esse cara funcionar, porque isso é literalmente **um bot funcional**. Então vamos fazê-lo rodar!

1. Copie esse código e cole em seu editor.
2. Substitua a string na função `client.login()` com _seu_ token
3. Salve o arquivo como `index.js`.
4. No CLI (que ainda deve estar na pasta do seu projeto) digite o seguinte comando: `node index.js`

Se tudo correu bem (espero que sim) seu bot agora está conectado ao seu servidor, está na sua lista de usuários, e pronto para responder todos os seus comandos... Bem, pelo menos, _um_ comando: `ping`. Em seu estado atual, o bot responderá "pong!" a qualquer mensagem que começar com, _exatamente_, `ping`. Vamos demonstrar!

![Ping?, Pong!](../.gitbook/assets/gs-ping-pong.png)

Sucesso! Você agora tem um bot rodando! Como você provavelmente percebe agora eu poderia provavelmente tagarelar daqui, mostrando um monte de coisas. Mas o escopo deste tutorial está completo, então vou calar a boca agora! Tchau!

## O Próximo Passo?

Agora que você tem um bot básico e funcional, é hora de começar a adicionar novos recursos! Dirija-se a [Seu Primeiro Bot](../primeiro-bot/seu-primeiro-bot.md) para continuar sua jornada adicionando novos comandos e recursos!

## Adendo: Obtendo Ajuda e Suporte

Antes de começar a obter suporte de servidores Discord para ajudá-lo com seu bot, eu fortemente aconselho dar uma olhada nos seguintes recursos muito úteis.

* [Documentação do Discord.js](http://discord.js.org) : Pelo amor de tudo que é (in)sagrado, **leia a documentação**. Sim, será estranho no início se você não está acostumado com "documentação de desenvolvedor", mas contém muita informação sobre cada recurso da API. Combine isso com os exemplos acima para ver a API em contexto.
* [An Idiot's Guide](https://www.youtube.com/c/AnIdiotsGuide) é outro ótimo canal com mais material. Os guias do York são ótimos e ele continua atualizando-os.
* [Evie.Codes no YouTube](https://www.youtube.com/channel/UCvQubaJPD0D-PSokbd5DAiw): Se você prefere vídeo a palavras, a série no YouTube da Evie (que é boa, embora não seja mais mantida com novos vídeos!) te leva a começar com bots.
* [Servidor Oficial do An Idiot's Guide](https://discord.gg/vXVxsAjSMF): O servidor oficial do An Idiot's Guide. Cheio de usuários amigáveis e úteis!
* [Servidor Oficial do Discord.js](https://discord.gg/djs): O servidor oficial tem várias pessoas competentes para ajudá-lo, e a equipe de desenvolvimento também está lá!
