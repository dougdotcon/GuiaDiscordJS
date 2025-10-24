# Instalando e Usando um Editor Adequado

Vamos tirar um momento para apreciar o fato de que o melhor código não é apenas código que _funciona_, mas também código que é _legível_. E código _legível_ já é mais fácil de resolver problemas. Para simplificar a vida de qualquer código, um bom editor é fundamental. Um bom editor dirá onde estão seus erros, validará seu código, dará melhores práticas, e alguns até executarão seu código para você.

Neste tutorial vamos olhar para um desses editores: Atom. Agora, não tenho preconceito pessoal a favor ou contra Atom, mas o editor, e alguma ajuda de pessoas que o usam, estavam prontamente disponíveis, então este é o que estou mostrando.

Outras alternativas seriam VS Code e Sublime Text 3. Você terá que procurar instruções específicas para esses editores por conta própria.

## Instalando o Atom

Longe de mim realmente mostrar como instalar coisas no seu computador - vou fingir por um momento que você sabe o que está fazendo e dar o esboço:

1. Vá para o [site Atom.io](https://atom.io/).
2. Clique no grande botão vermelho **Download Windows Installer** (estatisticamente, você está no Windows).
3. Uma vez que o .exe seja baixado, execute-o e instale-o.

Atom começa com uma interface simples com uma boa tela de boas-vindas e um guia, então se você quiser, vá em frente e leia um pouco.

## Abrindo seu projeto

Ao contrário de alguns editores mais básicos, Atom tem a capacidade de abrir uma _pasta_ de projeto para que você não precise continuar abrindo arquivos individualmente.

* Vá para **Arquivo**, **Abrir Pasta** (CTRL+SHIFT+O)
* Navegue até a localização do seu arquivo principal do bot (index.js, app.js, ou qualquer outro)
* Clique em **Selecionar Pasta** (_Nota: seus arquivos não aparecerão aqui, apenas a estrutura de pastas. Não se preocupe, eles não foram para lugar nenhum_)

À esquerda do editor você terá todos os arquivos. Basta clicar duas vezes em qualquer um deles para abri-lo. Para limpar as coisas, você pode fechar a Tela de Boas-vindas e tal.

{% hint style="info" %}
Já, você pode ver que este código parece super limpo e colorido. Além disso, o tema escuro não machuca os olhos, então isso é uma vantagem.
{% endhint %}

## Instalando ESLint

Um 'Linter' é um plugin ou app que verifica seu código para dizer onde estão os erros, e também ajuda na formatação do código com indentação e estilos adequados.

{% hint style="info" %}
Obviamente há um debate sobre qual linter você deve usar. ESLint? JSHint? Alguma outra biblioteca obscura? Vou usar ESLint porque... sei como usá-lo. Não estou endossando de forma alguma!
{% endhint %}

ESLint funciona em 2 partes. A primeira é o plugin para Atom, que é instalado através da linha de comando:

* Pressione Windows+R no seu teclado
* Digite `cmd` e pressione Enter
* Digite o comando `apm install linter-eslint` e pressione Enter
* Isso pode levar alguns segundos a um minuto para completar, seja paciente.
* Mantenha o Prompt de Comando aberto.

Em seguida, precisamos instalar o módulo eslint npm, que é o que realmente verifica seu código. `linter-eslint` apenas integra esses resultados no Atom.

Na linha de comando, digite `npm i -g eslint` e pressione Enter. Uma vez completo (mostrará uma lista de eslint e dependências instaladas).

Você precisará reiniciar o Atom para que as mudanças tenham efeito.

## Configurando opções ESLint

ESLint, por padrão, esperará código _muito_ rigoroso de você e reclamará por uma série de coisas. Se você usar o tipo errado de aspas (simples vs duplas), ou se usar 4 em vez de 2 espaços, alarmes soarão.

As opções ESLint podem ser globais ou locais ao seu projeto. Vamos fazer uma local como exemplo.

* Crie um novo arquivo no Atom
* Salve-o como `.eslintrc.json` na pasta do seu projeto (onde está seu app.js/index.js)
* Copie o código abaixo dentro do arquivo e salve novamente:

```javascript
{
    "env": {
        "es6": true,
        "node": true
    },
    "extends": "eslint:recommended",
    "parserOptions": {
        "sourceType": "module"
    },
    "rules": {
        "no-console": "off",
        "indent": [
            "error",
            2
        ],
        "linebreak-style": [
            "error",
            "windows"
        ],
        "quotes": [
            "warn",
            "double"
        ],
        "semi": [
            "warn",
            "always"
        ]
    }
}
```

Não se intimide com isso: é apenas uma pequena configuração em formato JSON. Não vou entrar em detalhes do que cada opção faz, elas estão todas explicadas [na documentação](http://eslint.org/docs/rules/), mas estas são as regras que estou usando. Basicamente:

* Forçar linter a suportar node
* Forçar linter a suportar código ES6 `<3`
* Indentar com 2 espaços (e não uma tab de 4 espaços, ugh)
* Fazer quebras de linha estilos Windows
* Avisar em aspas simples, preferir aspas duplas
* Avisar quando o ponto e vírgula está ausente
