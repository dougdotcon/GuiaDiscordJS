---
description: 'O guia não oficial para iniciantes em Discord.js, escrito por idiotas para iniciantes.'
---

# 🎉 Bem-vindo ao Guia Discord.js Bot

<div align="center">

### 📚 O guia completo para criar bots incríveis no Discord usando Discord.js v13+

[![Status da Tradução](https://img.shields.io/badge/Tradução-PT--BR-success)](checklist.md)
[![Discord.js](https://img.shields.io/badge/Discord.js-v13-blue)](https://discord.js.org)
[![Licença](https://img.shields.io/badge/Licença-MIT-green)](LICENSE)

</div>

---

## 📖 O que é este projeto?

Este é um **guia completo e traduzido para Português Brasileiro** que ensina como criar bots para Discord usando a biblioteca Discord.js. Diferente da documentação oficial que pode ser intimidadora para iniciantes, este guia oferece explicações passo a passo com exemplos práticos e código funcional.

### Para que serve?

Este guia foi criado para **simplificar o aprendizado** de desenvolvimento de bots Discord. Ele foi escrito por desenvolvedores experientes que entenderam a dificuldade de começar na biblioteca Discord.js e criaram este recurso para ajudar iniciantes a:

- ✨ **Aprender** os conceitos fundamentais do Discord.js
- 🚀 **Criar** seu primeiro bot funcional
- 🎯 **Implementar** funcionalidades avançadas como sistemas de pontos, starboards, logs, etc.
- 🛠️ **Desenvolver** habilidades de programação JavaScript/Node.js na prática
- 📚 **Entender** a estrutura e funcionamento da API do Discord

### O que você pode fazer com este guia?

Ao seguir este guia, você será capaz de criar bots Discord com as seguintes funcionalidades:

**🔧 Funcionalidades Básicas:**
- Criar comandos customizados com prefixos
- Gerenciar mensagens e canais
- Detectar eventos do servidor (entrada de membros, mensagens deletadas, etc.)
- Usar emojis personalizados e do Discord
- Criar embeds ricos e visualmente atraentes

**⚙️ Funcionalidades Intermediárias:**
- Sistema de pontos e níveis com banco de dados SQLite
- Sistema de starboard (mensagens destacadas)
- Rastreamento de convites (ver quem convidou quem)
- Logs de auditoria e moderação
- Comandos com cooldown e proteção de acesso
- Manipulador de comandos modular e organizado

**🚀 Funcionalidades Avançadas:**
- Webhooks para notificações e integrações
- Sharding para bots grandes (milhares de servidores)
- Gerenciamento completo de cargos e permissões
- Comando eval para desenvolvedores
- Integração com APIs externas (Cleverbot, etc.)
- Configuração por servidor

**🛠️ Ferramentas de Desenvolvimento:**
- Uso de Git e GitHub para controle de versão
- Variáveis de ambiente para segurança
- Async/await para programação assíncrona
- Editor adequado para produtividade
- Deploy em serviços de hospedagem

---

## 🎯 Para quem é este guia?

Este guia é perfeito para:

- 👨‍💻 **Iniciantes** em programação que querem aprender JavaScript de forma prática
- 🤖 **Desenvolvedores** que querem criar bots para seus servidores Discord
- 📝 **Estudantes** aprendendo desenvolvimento web e Node.js
- 🔧 **Autodidatas** que preferem aprender através de exemplos práticos
- 🇧🇷 **Comunidade brasileira** que prefere conteúdo em português

**Requisitos:** Conhecimento básico de JavaScript é recomendado (variáveis, funções, objetos). Se você não conhece JavaScript, recomendamos estudar os fundamentos primeiro.

---

## 🚀 Começando

Novo no Discord.js? Recomendamos começar pela seção [Começando](comecando/README.md) onde você encontrará:

- ✅ Instalação do Node.js e configuração do ambiente
- ✅ Criação do bot no Discord Developer Portal
- ✅ Configuração do primeiro bot funcional
- ✅ Guias rápidos (TL;DR) e versões detalhadas
- ✅ Entendimento dos conceitos básicos

---

## 📚 Índice de Conteúdo

### 🎯 Fundamentos

**[Primeiro Bot](primeiro-bot/README.md)**  
Crie seu primeiro bot funcional com comandos personalizados e configuração completa.

**[Eventos e Manipuladores](entendendo/eventos-e-manipuladores.md)**  
Entenda como o Discord.js funciona com eventos assíncronos e manipulação de ações.

**[Coleções](entendendo/colecoes.md)**  
Aprenda a trabalhar com dados da API do Discord de forma eficiente.

### 💻 Guias de Programação

**[Sistema de Pontos](guias-de-programacao/sistema-pontos-sqlite.md)**  
Implemente um sistema completo de XP/Níveis com banco de dados SQLite.

**[Starboard](guias-de-programacao/criando-starboard.md)**  
Crie um sistema de favoritos onde mensagens são destacadas automaticamente.

**[Rastreamento de Convites](guias-de-programacao/rastreando-convites.md)**  
Monitore qual membro convidou cada pessoa que entra no servidor.

**[Logs de Auditoria](guias-de-programacao/usando-logs-auditoria.md)**  
Identifique quem executou ações de moderação no servidor.

**[Emojis](guias-de-programacao/usando-emojis.md)**  
Use emojis personalizados e Unicode em seus comandos e mensagens.

### 🌐 Funcionalidades Avançadas

**[Webhooks](webhooks-discord/README.md)**  
Configure notificações e integrações com serviços externos.

**[Sharding](entendendo/sharding.md)**  
Escale seu bot para suportar milhares de servidores.

**[Permissões](entendendo/cargos.md)**  
Gerencie cargos e permissões de forma programática.

### 📝 Exemplos Práticos

**[Comando Eval](exemplos/criando-comando-eval.md)**  
Implemente um console de desenvolvedor para testar código.

**[Exemplos Diversos](exemplos/exemplos-diversos.md)**  
Cooldowns, purga de mensagens, detectores de palavrões, etc.

### 🛠️ Ferramentas e Configuração

**[Variáveis de Ambiente](outros-guias/variaveis-ambiente.md)**  
Proteja tokens e configurações sensíveis com arquivos .env.

**[Git e GitHub](outros-guias/usando-git.md)**  
Use controle de versão para gerenciar seu código.

**[Editor Adequado](outros-guias/instalando-editor-adequado.md)**  
Configure seu ambiente de desenvolvimento para máxima produtividade.

**[Async/Await](outros-guias/async-await.md)**  
Domine programação assíncrona moderna em JavaScript.

---

## 🆘 Obtenha Suporte

### Discord Servers

<div align="center">

👥 **[Servidor An Idiot's Guide](https://discord.gg/bRCvFy9)**  
Dúvidas sobre este guia? Entre no nosso servidor!

🤖 **[Servidor Oficial Discord.js](https://discord.gg/djs)**  
Suporte oficial da biblioteca Discord.js

</div>

### Outros Recursos

- 📖 [Documentação Oficial do Discord.js](https://discord.js.org)
- 💡 [FAQ - Perguntas Frequentes](perguntas-frequentes.md)
- ⚠️ [Erros Comuns e Soluções](erros-comuns.md)

---

## 📊 Status da Tradução

Este guia foi **completamente traduzido** para **Português Brasileiro (PT-BR)**. 

✅ **39 arquivos traduzidos**  
📈 **100% completo**  
🎯 **Nomes de pastas e arquivos traduzidos**  
🔗 **Todos os links internos atualizados**

Verifique o [Checklist de Tradução](checklist.md) para ver o progresso detalhado.

---

## 🌟 Contribuindo

Contribuições são bem-vindas! Este guia é mantido pela comunidade e novos colaboradores são sempre apreciados. Se você encontrar erros, tiver sugestões ou quiser melhorar algo, sinta-se à vontade para contribuir.

---

## 🙏 Agradecimentos

Agradecemos a todos os colaboradores que tornaram este guia possível e continuam ajudando a mantê-lo atualizado e acessível. Mantenedores originais: York#0001 e Discordaholic#0001 ("Evie.Codes").

---

## 📄 Licença

Este guia é distribuído sob a licença MIT. Veja o arquivo LICENSE para mais detalhes.

---

<div align="center">

**Feito com ❤️ pela comunidade Discord.js**

[Iniciar Tutorial](comecando/README.md) • [Ver FAQ](perguntas-frequentes.md) • [Erros Comuns](erros-comuns.md)

</div>
