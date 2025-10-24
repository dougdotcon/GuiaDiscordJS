# Usando Git para Compartilhar e Atualizar Código

Você já chegou a um ponto onde está editando código, removendo e adicionando e mudando coisas e de repente percebe, _Merda, deletei esta parte e preciso reescrever ou reutilizar agora. Droga!_

Você já desejou que houvesse uma maneira mais simples de transferir seu código para sua hospedagem, em vez de ter que conectar a um FTP, ou zipar seu arquivo, fazer upload para seu host e descompactar... você sabe a rotina, certo? Ugh!

Você já desejou poder compartilhar facilmente seu código e ter outras pessoas ajudando você em alguns pedaços, tornando seu bot melhor?

Se você respondeu **Sim** a qualquer uma dessas perguntas, menino eu tenho um produto para _você_! É chamado `git` e é uma ferramenta bastante mágica.

## Pré-requisitos e Software

Para aproveitar ao máximo o `git` você precisa primeiro ter _pelo menos_ completado o walkthrough [Adicionando um arquivo config.json](../primeiro-bot/seu-primeiro-bot.md#adding-a-configjson-file-to-your-bot). Isso significa que pelo menos, seu token está localizado em um arquivo separado, e não dentro de seus arquivos javascript. Ter o prefixo e o ID do dono nesse arquivo de configuração separado significa que você obtém a vantagem de teste fácil: Mude o prefixo e o token, e você tem um bot secundário com o qual pode testar seu código, wooh!

O que mais você precisa? O próprio `git`, é claro! Para Windows obtenha [Git SCM](https://git-scm.com/download/win), no Linux execute `sudo apt-get install git-all` ou `sudo yum install git-all` dependendo do método de instalação da sua distro. Usuários Mac também têm um instalador [Git SCM](http://git-scm.com/download/mac).

Mais alguma coisa? Eeeeh, não! Isso é tudo, realmente. Vamos para o uso!

## Inicialização e `.gitignore`

Então o primeiro passo de bebê no mundo do git é inicializar sua pasta de projeto como um repositório git. Para fazer isso, você precisa navegar até essa pasta na sua linha de comando. Ou usar um atalho do OS:

* Usuários Mac podem usar [este truque do lifehacker.com](http://lifehacker.com/launch-an-os-x-terminal-window-from-a-specific-folder-1466745514).
* Usuários Windows, lembre-se do truque mágico: SHIFT+Botão Direito na sua pasta de projeto, depois escolha **Abrir janela de comando aqui**.
* A maioria das distros Linux tem uma opção **Abrir no Terminal**. Mas você usa Linux, consegue descobrir, certo?

Uma vez que você tenha seu prompt aberto nessa pasta, vá em frente e execute `git init`. Não tem opções ou perguntas - apenas inicializa a pasta.

### Ignorando Arquivos

No git, um dos arquivos mais importantes é `.gitignore` que, como o nome implicaria, ignora certos arquivos. Isso é bastante crítico com apps node.js e nosso bot: você definitivamente quer e precisa ignorar tanto a pasta `node_modules` quanto o arquivo `config.json`. O primeiro porque você não quer milhares de arquivos sendo enviados (e eles não funcionam em outros sistemas de qualquer forma), o último porque você não quer que seu token acabe em um repositório `git` público!

Há arquivos `.gitignore` pré-construídos que você pode pegar da internet, mas para o propósito deste exercício você só precisa de 2 linhas nesse arquivo:

```text
node_modules
config.json
```

Mas como você cria isso? Usuários Linux e Mac, provavelmente podem criar o arquivo diretamente com facilidade. Usuários Windows, você tem que fazer um pequeno desvio - Windows não gosta de arquivos com apenas uma extensão e sem nome de arquivo. Não se preocupe, mas é simples. Comece criando um novo arquivo chamado, por exemplo, `gitignore.txt` e coloque as 2 linhas acima. Então, no seu console, execute `rename gitignore.txt .gitignore` e isso o renomeará como esperado.

## Seu primeiro comando `git`

E agora estamos prontos para começar a `git gud` (não não digite isso é um meme!)

O primeiro comando que você pode tentar é `git status`. Isso mostrará uma lista de todos os seus arquivos e em todas as subpastas e indicará-os como **arquivos não rastreados**, significando que não estão sendo rastreados (salvos) pelo git.

Vamos resolver isso com nosso segundo comando, que essencialmente diz ao git para rastrear todos esses arquivos e começar a fazer sua mágica neles: `git add .` onde `.` é apenas um atalho para 'todos os arquivos incluindo subpastas'.

E agora é hora de se _comprometer_ com a tarefa. _Fazer commit_ basicamente significa que você está criando um snapshot do seu projeto, que é salvo para sempre em seu histórico. Um commit pode ser tão pequeno quanto uma pequena correção de bug de uma linha ou reescrever uma função ou módulo completo. Fazer commit sempre será com uma mensagem que descreve a mudança, então commits menores podem ser mais fáceis de gerenciar e rastrear no futuro. Então vamos lá:

`git commit -m 'Commit Inicial do Projeto'`

Isso produzirá um novo commit com algumas estatísticas incluindo o número de linhas alteradas e a lista de arquivos alterados. Ótimo! Agora temos um commit. História engraçada: você pode continuar fazendo commits de mudanças localmente, e fazer todas as grandes coisas que `git` faz sem nunca fazer push para um repositório remoto. Mas é para isso que estamos aqui, certo? Então vamos em frente!

## Criando uma conta GitHub/GitLab/BitBucket

Com `git` você tem muitas escolhas quando se trata de hospedar seus projetos online. Fazer push para um repositório online lhe dá segurança em backups - todo o seu histórico de commits está disponível, e nada é perdido aconteça o que acontecer. Também lhe dá a capacidade de compartilhar seus projetos publicamente ou privadamente com outras pessoas, e tê-las contribuindo para seu projeto se você quiser.

Os 3 principais serviços de hospedagem `git` que são gratuitos são GitHub, GitLab e BitBucket. Vou usar GitHub neste caso, mas a maioria dos passos que darei aqui são os mesmos para todos os 3 serviços. Você só precisa de uma conta e está pronto para começar!

Então vamos em frente e preparar o GitHub. No [GitHub.com](https://github.com/), crie uma conta e depois crie um **Novo Repositório**. Dê-lhe um nome que identifique seu projeto - não precisa ser idêntico ao nome da sua pasta, e você pode mudá-lo depois se quiser. Não adicione um readme por enquanto.

Na página que exibe após criar o repositório você realmente obtém as instruções que precisa fazer, sob **push an existing repository from the command line**:

```text
git remote add origin git@github.com:seu-usuario/seu-nome-de-repo.git
git push -u origin master
```

Obviamente, mude `seu-usuario` e `seu-nome-de-repo` para as strings apropriadas. A primeira linha _liga_ seu repositório local ao site GitHub. A segunda linha realmente pega todo o seu histórico de commits local, e o _empurra_ diretamente para o github. Note que esta ação pedirá para você inserir seu nome de usuário e senha do github.

'Espera, realmente? É só isso?' você pergunta. Sim, seu código agora está no github. Atualize a página do github e você verá todo o seu código lá - sem o `node_modules` e `config.json` se você seguiu corretamente!

## Fazendo push de mais atualizações

Ligar sua conta não te salva de comandos futuros. Na verdade, toda vez que você faz uma mudança e quer empurrá-la para o repositório você tem que usar aquele comando `push` novamente. Note que você não _precisa_ fazer push de cada commit individualmente - você pode trabalhar por um dia e fazer múltiplos commits, e empurrá-los todos de uma vez no final do dia.

Um pequeno atalho que você pode usar é `add` e `commit` ao mesmo tempo. Isso só funciona se você tiver arquivos alterados mas não criado novos (para isso apenas faça `git add .` separadamente). Então se você terminou apenas pequenas mudanças, pode usar isso:

```text
git commit -am 'Corrigido problema de permissão no comando 8ball'
```

E não se esqueça de `git push origin master` novamente!

## Levando seu código para outra máquina

Então, digamos que você tenha uma hospedagem VPS em algum lugar. Ou um Raspberry Pi. Ou qualquer outra máquina. Você quer levar esse código seu para ela. Você primeiro precisa seguir os passos de `instalação` acima para obter `git` instalado na máquina, e então precisa _clonar_ o repositório do git remoto:

```text
git clone git@github.com:seu-usuario/seu-nome-de-repo.git
```

É isso! Bem, você precisa inserir seu nome de usuário e senha novamente, mas uma vez feito, esse código está disponível!

Não se esqueça que qualquer arquivo que foi adicionado ao seu arquivo `.gitignore` não está presente aqui, então você precisará fazer 2 coisas:

1. Criar seu `config.json` novamente, seja com o mesmo token e prefixo, ou diferentes.
2. Instalar seus módulos node novamente. Se você executou corretamente `npm init` e instalou todos os seus módulos com o argumento `-S` ou `--save`, você só precisa fazer `npm install`. Caso contrário, você precisa instalá-los todos novamente, por exemplo usando `npm install discord.js`

Agora, lembre-se que esses arquivos não estão sincronizados então não serão sobrescritos ou modificados sempre que você fizer push ou pull de e para o repo.

Falando nisso, qualquer vez que você fez `push` para o GitHub, pode obter uma cópia atualizada do código simplesmente executando `git pull origin master`.

Você terá que reiniciar seu bot para que as mudanças tenham efeito, é claro.

## Algumas Últimas Dicas

O processo não é unidirecional. Se você modificar arquivos em _qualquer_ computador pode `git push origin master` e `git pull origin master` no outro. Tenha cuidado ao editar em ambos os locais, no entanto, pois isso pode causar conflitos que são irritantes de resolver (e estão além do escopo deste guia)

SE você alguma vez tiver mudanças 'temporárias' e quiser sobrescrevê-las - como uma correção rápida no seu VPS que você também corrigiu localmente - pode evitar todo o processo de 'conflito' _Resetando_ um repositório para seu último `pull`. Simplesmente execute `git reset --hard HEAD` para apagar quaisquer mudanças locais, depois execute `git pull origin master` para pegar as mudanças mais recentes.

Você pode conectar a múltiplos repositórios remotos executando o comando `remote add` acima. Você precisará de um nome diferente, por exemplo em vez de `origin` você pode chamar um remoto `gitlab` e então qualquer comando deve refletir isso, como `git pull gitlab master`.

Há **muito** mais no `git` do que o que foi mostrado aqui (e estou ciente de que mesmo isso já é uma porção de informações). Você pode criar e mesclar `branches`, `reverter` para commits anteriores, fazer e aceitar `PR`s... [Há muito mais](https://www.google.com/search?q=git+tutorials) que você pode aprender!

Mas por agora... acho que esta parede maciça de texto é suficiente.
