# Erros Comuns

## Não é possível encontrar o módulo `discord.js`

### Problema:

Você não instalou o Discord.js ou instalou na pasta errada.

### Solução:

* Certifique-se de estar na pasta **correta** onde estão os arquivos do seu bot
* Pressione SHIFT+Botão Direito na pasta e selecione **Abrir janela de comando aqui**
* Execute `npm init -y` e pressione Enter até o assistente concluir
* Execute `npm i discord.js` novamente para instalar o Discord.

## `Error: getaddrinfo ENOTFOUND gateway.discord.gg gateway.discord.gg:443`

### Problema:

Sua internet caiu.

### Solução:

Obtenha uma internet melhor e mais estável, ou hospede seu bot em um VPS.

## Fim Inesperado de Entrada

### Problema:

```text
});
^
SyntaxError: Unexpected end of input
```

### Solução:

Seu código tem um erro em algum lugar. Isso é _impossível_ de diagnosticar sem o código **completo**, já que o erro pode estar em qualquer lugar (na verdade, a pilha de erros geralmente diz que está no final do seu código).

O seguinte truque é uma mão na roda, então preste atenção: Seu editor de código está tentando ajudá-lo. Qualquer editor que você esteja usando (exceto notepad++.exe. Não use notepad++!), clicar em qualquer caractere especial como parênteses, colchetes, chaves, aspas duplas e simples, destacará automaticamente o que corresponde a ele. A captura de tela abaixo mostra isso: Cliquei na chave na parte inferior, ela me mostra a de cima destacando-a. Aprenda isso e como diferentes funções e manipuladores de eventos "se parecem".

![Eles têm amigos](.gitbook/assets/editorhelp.png)

Você pode conferir [Instalando e Usando um Editor Adequado](outros-guias/instalando-editor-adequado.md) para ajudar pelo menos a saber que há erros _antes_ de executar o código do seu bot.
