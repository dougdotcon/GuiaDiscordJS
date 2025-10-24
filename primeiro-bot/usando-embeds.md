# Usando Embeds em Mensagens

{% hint style="warning" %}
Embeds podem parecer bonitos, mas podem ser desabilitados através de permissões e preferências do usuário, e não ficarão iguais no mobile - especialmente os complexos. É fortemente recomendado _não_ usá-los a menos que você tenha um fallback apenas de texto. Sim, eles são bonitos, mas não os use se você não _precisar_!
{% endhint %}

## Embeds

Aqui estão algumas regras para embeds:

* Todo campo é opcional
* Pelo menos um campo deve estar presente
* Nenhum campo pode estar vazio, null ou undefined.

Essas não são apenas diretrizes, são regras, e quebrar essas regras significa que seu embed não será enviado - retornará `Bad Request`.

Há 2 maneiras de fazer embeds. A maneira mais limpa é usando o construtor `MessageEmbed` e a segunda é escrevendo o objeto em si, vamos cobrir o construtor `MessageEmbed` _primeiro_.

## Construtor MessageEmbed

As mesmas regras se aplicam para `MessageEmbed` como se aplicam para os baseados em objeto. Na verdade, o construtor é apenas um atalho para obter o mesmo objeto e oferece nem mais, nem menos funcionalidade.

```javascript
const embed = new Discord.MessageEmbed()
  /*
   * Alternativamente, use "#3498DB", [52, 152, 219] ou um número inteiro.
   */
  .setColor(0x3498DB)
  .setAuthor("Nome do Autor, pode conter 256 caracteres", "https://i.imgur.com/lm8s41J.png")
  .setTitle("Este é seu título, pode conter 256 caracteres")
  .setURL("https://discord.js.org/#/docs/main/stable/class/MessageEmbed")
  .setDescription("Este é o corpo principal do texto, pode conter 4096 caracteres.")
  .setImage("http://i.imgur.com/yVpymuV.png")
  .setThumbnail("http://i.imgur.com/p2qNFag.png")
  .addField("Este é um título de campo único, pode conter 256 caracteres", "Este é um valor de campo, pode conter 1024 caracteres.")
  /*
   * Campos inline podem não ser exibidos como inline se a miniatura e/ou imagem for muito grande.
   */
  .addFields(
    { name: "Campos inline", value: "Eles podem ter diferentes campos com pequenos títulos, e você pode inline-los.", inline: true },
    { name: "Links mascarados", value: "Você pode colocar [links mascarados](https://discord.js.org/#/docs/main/master/class/MessageEmbed) dentro de embeds ricos.", inline: true },
    { name: "Markdown", value: "Você pode colocar todo o *usual* **__Markdown__** dentro deles.", inline: true }
  )
  /*
   * Campo em branco, útil para criar algum espaço.
   */
  .addField("\u200b", "\u200b")
  /*
   * Recebe um objeto Date, por padrão a data atual.
   */
  .setTimestamp()
  .setFooter("Este é o texto do rodapé, pode conter 2048 caracteres", "http://i.imgur.com/w1vhFSR.png");
  /*
   * Com o Discord agora permitindo que mensagens contenham até 10 embeds, precisamos colocá-lo em um array.
   */
  message.channel.send({ embeds: [embed] });
```

Que produz o seguinte:

![O exemplo de embed](../.gitbook/assets/first-bot-embed-example.png)

Agora esse código não parece limpo e incrível? Bem, vamos olhar a versão de objeto.

```javascript
  message.channel.send({ embeds: [{
    color: 3447003,
    author: {
      name: "Nome do Autor, pode conter 256 caracteres",
      icon_url: "https://i.imgur.com/lm8s41J.png"
    },
    thumbnail: {
      url: "http://i.imgur.com/p2qNFag.png"
    },
    image: {
      url: "http://i.imgur.com/yVpymuV.png"
    },
    title: "Este é seu título, pode conter 256 caracteres",
    url: "https://discord.js.org/#/docs/main/master/class/MessageEmbed",
    description: "Este é o corpo principal do texto, pode conter 2048 caracteres.",
    fields: [{
      name: "Este é um título de campo único, pode conter 256 caracteres",
      value: "Este é um valor de campo, pode conter 1024 caracteres.",
      inline: false
    },
    {
      name: "Campos inline",
      value: "Eles podem ter diferentes campos com pequenos títulos, e você pode inline-los.",
      inline: true
    },
    {
      name: "Links mascarados",
      value: "Você pode colocar [links mascarados](https://discord.js.org/#/docs/main/master/class/MessageEmbed) dentro de embeds ricos.",
      inline: true
    },
    {
      name: "Markdown",
      value: "Você pode colocar todo o *usual* **__Markdown__** dentro deles.",
      inline: true
    },
    {
      name: "\u200b",
      value:"\u200b"
    }],
    timestamp: new Date(),
    footer: {
      icon_url: "http://i.imgur.com/w1vhFSR.png",
      text: "Este é o texto do rodapé, pode conter 2048 caracteres"
    }
  }]});
```

Como você pode ver, é um pouco grande em comparação, mas ambos produzem o mesmo embed, então não há razão real para mostrá-lo, parabéns você sabe como usar embeds em mensagens normais agora!

Não é diferente se você estiver usando um webhook ou uma resposta de interação.
