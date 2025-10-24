# Cargos e Permissões

Cargos são um recurso poderoso no Discord, e admitidamente têm sido uma das partes mais difíceis de dominar no discord.js. Este passo a passo visa explicar como cargos e permissões funcionam. Também vamos explorar como usar cargos para proteger seus comandos.

## Hierarquia de Cargos

Vamos começar com uma visão geral básica da hierarquia de cargos no Discord.

... ou na verdade não, eles já explicam melhor do que eu me importo: [Gerenciamento de Cargos 101](https://support.discord.com/hc/en-us/articles/214836687-Role-Management-101). Leia sobre isso, depois volte aqui. Vou esperar. (Sim eu sei que é cafona, então me processe).

## Código de Cargo

Vamos direto ao ponto. Você quer saber como usar cargos e permissões em seu bot.

### Obter Cargo por Nome ou ID

Esta é a parte "fácil" assim que você realmente se acostuma com isso. É apenas como obter qualquer outro elemento de Coleção, mas aqui está um lembrete de qualquer forma!

```javascript
// obter cargo por ID
let myRole = message.guild.roles.cache.get("264410914592129025");

// obter cargo por nome
let myRole = message.guild.roles.cache.find(role => role.name === "Moderators");
```

{% hint style="info" %}
Para obter o ID de um cargo, você pode mencioná-lo com um `\` antes dele, como `\@rolename`, ou copiá-lo do menu de cargos. Se você mencioná-lo, o ID são os números entre os `<>`. Para obter o ID de um cargo sem mencioná-lo, habilite o modo desenvolvedor na seção Aparência das suas configurações de usuário, depois vá para o menu de cargos nas configurações do servidor e clique com o botão direito no cargo do qual você quer o ID, depois clique em "Copiar ID".
{% endhint %}

### Verificar se um membro tem um cargo

Em um manipulador de `messageCreate`, você tem acesso a verificar a classe GuildMember do autor da mensagem:

```javascript
// assumindo que role.id é um ID real de um cargo válido:
if (message.member.roles.cache.has(role.id)) {
  console.log("Oba, o autor da mensagem tem o cargo!");
}

else {
  console.log("Não, não tem, nada.");
}
```

```javascript
// Verificar se eles têm um de muitos cargos
if (message.member.roles.cache.some(r=>["Dev", "Mod", "Server Staff", "Proficient"].includes(r.name)) ) {
  // tem um dos cargos
}

else {
  // não tem nenhum dos cargos
}
```

{% hint style="info" %}
Para pegar membros e usuários de diferentes maneiras veja a [Página FAQ](../frequently-asked-questions.md).
{% endhint %}

### Obter todos os membros que têm um cargo

```javascript
let roleID = "264410914592129025";
let membersWithRole = message.guild.roles.cache.get(roleID).members;
console.log(`Consegui ${membersWithRole.size} membros com esse cargo.`);
```

### Adicionar um membro a um cargo

Ok, agora que você tem cargos, provavelmente quer adicionar um membro a um cargo. Simples o suficiente! Discord.js fornece 2 métodos úteis para adicionar e remover um cargo. Vamos olhar para eles!

```javascript
let role = message.guild.roles.cache.find(r => r.name === "Team Mystic");

// Vamos fingir que você mencionou o usuário para o qual quer adicionar um cargo (!addrole @user Nome do Cargo):
let member = message.mentions.members.first();

// ou a pessoa que fez o comando: let member = message.member;

// Adicionar o cargo!
member.roles.add(role).catch(console.error);

// Remover um cargo!
member.roles.remove(role).catch(console.error);
```

Ok, sinto que tenho que adicionar um _pouco_ de precisão aqui na implementação:

* Você **não pode** adicionar ou remover um cargo que é mais alto que o do bot. Isso deveria ser óbvio.
* O bot requer permissões `MANAGE_ROLES` para isso. Você pode verificar isso usando o código mais abaixo nesta página.
* Por causa dos limites de taxa globais, você não pode fazer 2 "ações" de cargo imediatamente uma após a outra. A primeira ação funcionará, a segunda não. Você pode contornar isso usando `<GuildMember>.roles.set([array, de, cargos])`. Isso sobrescreverá todos os cargos existentes e apenas aplicará os que estão no array então tenha cuidado com isso.

## Código de Permissão

### Verificar permissão específica de um membro em um canal

Para verificar uma única sobrescrita de permissão em um canal:

```javascript
// Obtendo todas as permissões para um membro em um canal.
let perms = message.channel.permissionsFor(message.member);

// Verifica permissões de Gerenciar Mensagens.
let can_manage_messages = message.channel.permissionsFor(message.member).has("MANAGE_MESSAGES", false);

// Ver permissões como um objeto (útil para debug ou eval)
message.channel.permissionsFor(message.member).serialize(false)
```

{% hint style="info" %} Passamos `false` para o parâmetro checkAdmin porque sobrescritas de canal Administrador não concedem implicitamente nenhuma permissão, ao contrário de Cargos ou quando você é o Dono do Servidor. (A API permitirá que você crie uma sobrescrita com Administrador, e até mesmo dirá ao D.JS que uma sobrescrita de canal teve permissões de Administrador definidas. Desenvolvedores do Discord declararam que isso é [comportamento intencional](https://github.com/discord/discord-api-docs/issues/640).)
{% endhint %}

### Obter todas as permissões de um membro em um servidor

Apenas tão fácil, uau!

```javascript
let perms = message.member.permissions;

// Verificar se um membro tem uma permissão específica no servidor!
let has_kick = perms.has("KICK_MEMBERS");
```

fácil, certo?

Agora vá programar!

## ADENDO: Nomes de Permissão

Clique [aqui](https://discord.js.org/#/docs/main/master/class/Permissions?scrollTo=s-FLAGS) para a lista completa de nomes de permissão internos, usados para `.has(name)` nos exemplos acima
