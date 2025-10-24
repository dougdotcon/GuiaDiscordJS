# Criando um Comando Eval

## O que é eval exatamente?

Em JavaScript (e node), `eval()` é uma função que avalia qualquer string _como código javascript_ e realmente a executa. Significa, se eu `eval(2+2)`, eval retornará `4`. Se eu avalio `client.guilds.cache.size`, retornará quantos servidores o bot está atualmente. E se eu avalio `client.guilds.cache.map(g=>g.name).join('\n')` então retornará uma lista de nomes de servidores separados por uma quebra de linha. Legal, certo?

## Mas eval é perigoso

Vou dizer isso de uma maneira que provavelmente é mortificante de entender: _Dar a alguém acesso a_ `eval()` _é literalmente como sentá-los **no seu computador**, com **acesso total de admin**, e então **sair da sala**._ `eval()` em javascript de navegador é trivial e não perigoso - você está executando no seu próprio navegador, qualquer coisa que você estragar vai estar no seu próprio PC, não nos servidores web.

Mas `eval()` em **node** é realmente, realmente perigoso e poderoso. Porque pode executar qualquer coisa que **você** executa como bot, e também pode executar código que você não está **esperando** executar, se alguém mais tiver acesso a ele. **Node.js** tem acesso ao seu **disco rígido**. A coisa toda. Cada pedaço dele. Para entender o que isso significa, olhe para o seguinte comando: `rm -rf / --no-preserve-root`. Você sabe o que este comando faz? **Ele deleta todos os discos rígidos do seu servidor**. Quero dizer, só funciona no Linux, mas a maioria dos sistemas VPS e a maioria dos provedores de hospedagem estão em sistemas baseados em UNIX.

Outra coisa que está no seu disco rígido são senhas. Tem um arquivo de configuração com sua senha do banco de dados? Posso conseguir isso. `config.json` com seu token e outras chaves de API nele? Posso pegar isso facilmente. Puxa, posso simplesmente pegar seu token direto do objeto `client` se souber como.

## Protegendo seu eval

Então primeiro, precisamos entender a regra #1 ao usar comandos `eval`:

{% hint style="danger" %}
_**NUNCA JAMAIS DÊ PERMISSÕES DE EVAL A NINGUÉM MAIS**_
{% endhint %}

Não me importo se é um dono de servidor, alguém com quem você tem conversado por meses, você **não pode** confiar em ninguém com eval. Há apenas uma exceção a esta regra: Alguém que você conhece **na vida real** que você pode socar na cara quando eles realmente destruírem metade do seu servidor ou banirem por engano todos em todos os servidores em que seu bot está. Eval ignora qualquer permissão baseada em comando que você possa ter, ignora todas as verificações de segurança. Eval é todo-poderoso.
{% endhint %}

Então como você protege isso? Simples: apenas permita uso do seu próprio ID de usuário. Então por exemplo meu ID de usuário é `139412744439988224` então verifico se o ID do autor da mensagem é o meu, que adicionamos na nossa configuração no início:

```javascript
{
  // O resto da config
  "ownerID": "139412744439988224"
}
```

No código para o bot:

```javascript
// Se o ID do autor da mensagem não for igual
// ao nosso ownerID, saia daí!
if (message.author.id !== config.ownerID) return;
```

É simples assim proteger o comando diretamente dentro da sua condição ou arquivo ou o que for. É claro, se você tem algum tipo de manipulador de comandos há provavelmente uma maneira de restringir a um ID também. Isso não é específico do discord.js: sempre há uma maneira de fazer isso. Se não houver (se um manipulador de comandos não deixar você restringir por ID), então **encontre uma nova biblioteca!**.

## Comando Eval Simples

Então agora você foi completamente informado sobre os perigos do Eval, vamos dar uma olhada em como implementar um comando eval simples.

Primeiro, porém, fortemente sugiro usar a seguinte função (coloque-a fora de qualquer manipulador de evento/funções que você tem, para que seja acessível em qualquer lugar). Esta função impede o uso de menções reais dentro da linha de retorno adicionando um caractere de largura zero entre o `@` e o primeiro caractere da menção - bloqueando a menção de acontecer.

{% hint style="info" %}
O seguinte trecho limpo é _**OBRIGATÓRIO**_ para fazer o eval funcionar.
{% endhint %}

```javascript
// Esta função limpa e prepara o
// resultado da entrada do nosso comando eval para enviar
// ao canal
const clean = async (text) => {
// Se nossa entrada é uma promise, aguarde antes de continuar
if (text && text.constructor.name == "Promise")
  text = await text;

// Se a resposta não é uma string, `util.inspect()`
// é usado para 'stringificar' o código de uma maneira segura que
// não vai dar erro em objetos com referências circulares
// (como Coleções, por exemplo)
if (typeof text !== "string")
  text = require("util").inspect(text, { depth: 1 });

// Substituir símbolos com alternativas de código de caractere
text = text
  .replace(/`/g, "`" + String.fromCharCode(8203))
  .replace(/@/g, "@" + String.fromCharCode(8203));

// Enviar o resultado limpo
return text;
}
```

Ok, então vamos chegar ao ponto: O comando eval real. Aqui está em toda sua glória, assumindo que você seguiu este guia completamente:

```javascript
// Este comando vai dentro do nosso manipulador de evento de mensagem
client.on("messageCreate", async (message) => {

  // Obter nossos argumentos de entrada
  const args = message.content.split(" ").slice(1);

  // O comando eval real
  if (message.content.startsWith(`${config.prefix}eval`)) {

    // Se o ID do autor da mensagem não for igual
    // ao nosso ownerID, saia daí!
    if (message.author.id !== config.ownerID)
      return;

    // No caso de algo falhar, precisamos capturar erros
    // em um bloco try/catch
    try {
      // Avaliar (executar) nossa entrada
      const evaled = eval(args.join(" "));

      // Colocar nosso resultado eval através da função
      // que definimos acima
      const cleaned = await clean(evaled);

      // Responder no canal com nosso resultado
      message.channel.send(`\`\`\`js\n${cleaned}\n\`\`\``);
    } catch (err) {
      // Responder no canal com nosso erro
      message.channel.send(`\`ERRO\` \`\`\`xl\n${cleaned}\n\`\`\``);
    }

    // Fim do nosso comando
  }

  // Fim do nosso manipulador de evento de mensagem
});
```

É isso. Esse é o comando. Note algumas coisas:

* Seu IDE ou editor pode gritar no `eval(code)` por ser `inseguro`. Veja o resto desta página sobre POR QUE diz isso. Não posso dizer que você não foi avisado!
* Se a resposta tiver mais de 2000 caracteres isso dará erro quando tentar enviar os resultados. Isso é devido ao limite de caracteres de mensagens enviadas a canais. Você precisará lidar com dividir isso e enviar os resultados maiores se quiser ser capaz de enviar resultados maiores.

{% hint style="info" %}
O comando eval acima NÃO VAI censurar/remover seu token do cliente se for retornado como parte da saída do resultado. Para censurar seu token das saídas de resultado, adicione o seguinte código.
{% endhint %}

Ajuste nossa função `clean()` para receber o cliente como entrada assim.

```javascript
const clean = async (client, text) => { 
  // O resto do código
}
```

Certifique-se de fornecer o cliente como entrada quando chamamos a função mais tarde no código.

```javascript
    // ...

    try {
      // Colocar nosso resultado eval através da função que definimos acima
      const cleaned = await clean(client, evaled);
    }

    // ...
```

```javascript
const clean = async (client, text) => { 
  // ...

  // Você precisará colocar isso dentro da função clean
  // antes que o resultado seja retornado.
  text = text.replaceAll(client.token, "[REDACTED]");

  // ...
}
```

## Lembrete Final

{% hint style="danger" %}
**EU NÃO SOU RESPONSÁVEL SE VOCÊ ESTRAGAR, E NEM SÃO NENHUM DOS USUÁRIOS E DESENVOLVEDORES DO DISCORD.JS**
{% endhint %}

Espero que os avisos tenham sido claros o suficiente para ajudá-lo a entender os perigos... mas a ideia de eval ainda é atraente o suficiente que você vai usá-lo para si mesmo de qualquer forma!
