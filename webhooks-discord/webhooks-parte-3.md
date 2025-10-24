# Webhooks (Parte 3)

Nos últimos dois capítulos cobrimos como criar e usar os webhooks via código.

Neste capítulo vamos cobrir serviços de terceiros como [Zapier](https://zapier.com/), [IFTT](https://ifttt.com/) e até mesmo [ShareX](https://getsharex.com/). Se você é membro do meu servidor você saberá que tem um canal \#video-guides onde um webhook posta meus vídeos quando eu os libero, essa ponte youtube-to-discord é fornecida pelo Zapier (Mais sobre isso depois).

Mas antes de pularmos para o Zapier, vamos cobrir algo que eu deveria ter coberto no primeiro capítulo.

A [documentação oficial](https://discord.com/developers/docs/intro) para os [endpoints de webhook](https://discord.com/developers/docs/resources/webhook) do Discord lista 3 "tipos", você tem seu [Webhook Padrão](https://discord.com/developers/docs/resources/webhook#execute-webhook), você tem um [Webhook Compatível com Slack](https://discord.com/developers/docs/resources/webhook#execute-slackcompatible-webhook) e um [Webhook Compatível com Github](https://discord.com/developers/docs/resources/webhook#execute-githubcompatible-webhook)

![Webhook Padrão](../.gitbook/assets/wh07.png)

O webhook padrão (acima) basicamente age como uma mensagem normal que você enviaria com seu bot, mas você pode definir o avatar, nome de usuário, etc, o que significa que você não precisa defini-los quando cria o webhook.

![Webhook Compatível com Slack](../.gitbook/assets/wh08.png)

O webhook compatível com slack (acima) é muito chique em comparação, eles são basicamente `MessageEmbeds` que você vê muitos bots de moderação usarem, você pode ter miniaturas, imagens, e mais (veja a [Documentação do Slack](https://api.slack.com/incoming-webhooks) para mais informações).

![Webhook Compatível com Github](../.gitbook/assets/wh09.png)

O webhook compatível com github (acima) é como o webhook compatível com slack, mas é limitado em sua customização, você só pode "customizar" quais informações ele envia do Github e não pode estilizá-lo.

Agora estabelecemos os tipos de webhook que o Discord pode lidar, vamos focar no webhook regular, e no Webhook Compatível com Slack.

Agora não há muito que possamos fazer com serviços de terceiros em relação ao webhook padrão, mas um amigo meu me informou, que você pode usar os webhooks do discord com [ShareX](https://getsharex.com/).

## ShareX

Agora, baixe e instale [ShareX](https://getsharex.com/) e abra o programa e clique no menu suspenso para `Destinations` e clique em [Configurações de Destino](https://raw.githubusercontent.com/AnIdiotsGuide/discordjs-bot-guide/master/.gitbook/assets/wh10.png) então role até o fundo para [Uploaders Personalizados](https://raw.githubusercontent.com/AnIdiotsGuide/discordjs-bot-guide/master/.gitbook/assets/wh11.png) e clique nele.

No campo de texto logo abaixo de "Uploaders" coloque `Discord`.

{% hint style="info" %}
Observe os botões `Add`, `Remove` e `Update` um pouco abaixo desse campo, usaremos eles depois.
{% endhint %}

Esse será o nome para este uploader personalizado. Depois disso, certifique-se de que o `Tipo de solicitação` está definido como `POST`, e onde diz `URL de solicitação`, é onde você quer colocar a URL do webhook que obteve do seu canal do discord.

Abaixo desse campo de texto há outro com o rótulo `Nome do formulário de arquivo`, apenas coloque `file` lá.

![Tipo de solicitação, url e nome do arquivo do formulário.](../.gitbook/assets/wh12.png)

Estamos quase terminando! Agora no canto inferior direito você encontrará uma seção com três campos, `URL`, `URL de Miniatura` e `URL de Exclusão`, o único que queremos é o campo `URL`.

Coloque isso lá. `$json:attachments[0].url$` e agora vamos salvar e testar!

Logo abaixo da caixa de texto onde você digitou `Discord` para este uploader personalizado, clique no botão `Add` e salvará as configurações para você, agora para testar.

No lado esquerdo, há uma série de menus suspensos, clique no menu suspenso para o `Image uploader` e selecione Discord e dê um clique nesse botão brilhante `Test`.

Se tudo correu bem, você deve ter uma resposta na caixa de texto do canto inferior direito, deve parecer algo assim;

![Uma resposta bem-sucedida.](../.gitbook/assets/wh13.png)

E se formos ao nosso canal e verificarmos devemos ver o seguinte;

![Sucesso!](../.gitbook/assets/wh14.png)

## Parabéns

Você conseguiu usar o Discord para armazenar seus recortes rápidos de imagem, mas ainda não terminamos, no próximo capítulo vamos cobrir [Zapier](https://zapier.com/) e [IFTTT](https://ifttt.com/) para usos mais avançados.
