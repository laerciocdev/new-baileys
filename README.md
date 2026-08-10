# new-baileys

Fork comunitário da [WhiskeySockets/Baileys](https://github.com/WhiskeySockets/Baileys), atualizado com base na versão 7.0.0-rc.9 da Baileys e com as alterações do fork `laerciocdev/new-baileys` aplicadas.

> Este pacote não é o projeto oficial da WhiskeySockets. Ele é um fork independente criado para manter compatibilidade com alguns fluxos usados em bots WhatsApp.

## Base desta versão

- Base upstream: `WhiskeySockets/Baileys`
- Versão base: `7.0.0-rc.9`
- Versão do fork: `7.0.0-rc.9-patch.1`
- Repositório do fork: `https://github.com/laerciocdev/new-baileys`

## O que esta versão traz

Esta versão mantém a base 7.0.0-rc.9 da Baileys e aplica as modificações:

- compatibilidade extra para envio de botões e mensagens interativas;
- suporte legado para campos como `buttons`, `templateButtons`, `sections`, `buttonText` e `interactiveButtons`;
- suporte a `nativeFlowMessage` para botões interativos, incluindo `quick_reply`, `cta_copy` e `cta_url`;
- helpers públicos no socket para conversão entre PN/JID e LID:
  - `sock.getLIDForPN(pn)`;
  - `sock.getLIDsForPNs(pns)`;
  - `sock.getPNForLID(lid)`;
- `onWhatsApp()` ajustado para consultar PN/JID e LID no mesmo fluxo e retornar:
  - `jid`;
  - `lid`;
  - `exists`;
- ajuste no `relayMessage` para adicionar o node `biz` necessário em mensagens interativas, botões e listas.

## Instalação pelo GitHub

```bash
npm i github:laerciocdev/new-baileys
```

Também pode usar a URL completa:

```bash
npm i https://github.com/laerciocdev/new-baileys.git
```

## Instalação pela npm

```bash
npm i @laerciocdev/new-baileys
```

## Import

Com o nome atual do pacote:

```ts
import makeWASocket from '@laerciocdev/new-baileys'
```

Ou com imports nomeados:

```ts
import makeWASocket, {
  useMultiFileAuthState,
  makeCacheableSignalKeyStore,
  DisconnectReason,
  Browsers,
  proto
} from '@laerciocdev/new-baileys'
```

## Exemplo básico

```ts
import makeWASocket, {
  useMultiFileAuthState,
  makeCacheableSignalKeyStore,
  fetchLatestBaileysVersion
} from '@laerciocdev/new-baileys'
import pino from 'pino'

const logger = pino({ level: 'silent' })
const { state, saveCreds } = await useMultiFileAuthState('./auth')
const { version } = await fetchLatestBaileysVersion()

const sock = makeWASocket({
  version,
  logger,
  auth: {
    creds: state.creds,
    keys: makeCacheableSignalKeyStore(state.keys, logger)
  }
})

sock.ev.on('creds.update', saveCreds)
```

## Exemplo de botões legados

```ts
const buttons = [
  {
    buttonId: 'id1',
    buttonText: { displayText: 'Botão 1' },
    type: 1
  },
  {
    buttonId: 'id2',
    buttonText: { displayText: 'Botão 2' },
    type: 1
  }
]

await sock.sendMessage(jid, {
  text: 'Oi, essa mensagem tem botões',
  footer: 'new-baileys',
  buttons,
  viewOnce: true
})
```

## Exemplo de botões interativos/native flow

```ts
await sock.sendMessage(jid, {
  text: 'Escolha uma opção',
  footer: 'new-baileys',
  interactiveButtons: [
    {
      name: 'quick_reply',
      buttonParamsJson: JSON.stringify({
        display_text: 'Responder',
        id: 'responder_1'
      })
    },
    {
      name: 'cta_copy',
      buttonParamsJson: JSON.stringify({
        display_text: 'Copiar código',
        copy_code: 'ABC123'
      })
    },
    {
      name: 'cta_url',
      buttonParamsJson: JSON.stringify({
        display_text: 'Abrir GitHub',
        url: 'https://github.com/laerciocdev/new-baileys'
      })
    }
  ],
  viewOnce: true
})
```

## Exemplo de botões com imagem

```ts
await sock.sendMessage(jid, {
  image: { url: './media/menu.jpg' },
  caption: 'Menu principal',
  footer: 'new-baileys',
  interactiveButtons: [
    {
      name: 'quick_reply',
      buttonParamsJson: JSON.stringify({
        display_text: 'Menu',
        id: '.menu'
      })
    },
    {
      name: 'cta_url',
      buttonParamsJson: JSON.stringify({
        display_text: 'GitHub',
        url: 'https://github.com/laerciocdev/new-baileys'
      })
    }
  ],
  viewOnce: true
})
```

## Exemplo de botões com GIF

No WhatsApp, GIF costuma funcionar melhor como vídeo `.mp4` com `gifPlayback: true`:

```ts
await sock.sendMessage(jid, {
  video: { url: './media/menu.mp4' },
  gifPlayback: true,
  caption: 'GIF com botões',
  footer: 'new-baileys',
  interactiveButtons: [
    {
      name: 'quick_reply',
      buttonParamsJson: JSON.stringify({
        display_text: 'Responder',
        id: '.responder'
      })
    }
  ],
  viewOnce: true
})
```

## Exemplo de lista legada

```ts
await sock.sendMessage(jid, {
  text: 'Veja as opções disponíveis:',
  footer: 'new-baileys',
  title: 'Menu principal',
  buttonText: 'Abrir lista',
  sections: [
    {
      title: 'Categoria 1',
      rows: [
        {
          title: 'Opção A',
          description: 'Descrição da opção A',
          rowId: 'opcao_a'
        },
        {
          title: 'Opção B',
          description: 'Descrição da opção B',
          rowId: 'opcao_b'
        }
      ]
    }
  ],
  viewOnce: true
})
```

> Observação: listas e alguns formatos `single_select` podem continuar instáveis em WhatsApp Web, iOS ou grupos. Para maior compatibilidade, prefira `quick_reply`, `cta_copy`, `cta_url` ou menus por texto.

## Exemplo de helpers LID/JID

```ts
const lid = await sock.getLIDForPN('5511999999999@s.whatsapp.net')

const pn = await sock.getPNForLID('1234567890@lid')

const mappings = await sock.getLIDsForPNs([
  '5511999999999@s.whatsapp.net',
  '5511888888888@s.whatsapp.net'
])

console.log({ lid, pn, mappings })
```

## Exemplo do onWhatsApp com LID

```ts
const results = await sock.onWhatsApp(
  '5511999999999@s.whatsapp.net',
  '1234567890@lid'
)

console.log(results)
// [
//   { jid: '5511999999999@s.whatsapp.net', lid: '...', exists: true },
//   { jid: '...', lid: '1234567890@lid', exists: true }
// ]
```

## Observações

- Este fork é comunitário e não é o pacote oficial da WhiskeySockets.
- O código-fonte vem da base do GitHub da Baileys.
- Recomenda-se travar a versão no seu projeto para evitar perda de compatibilidade em atualizações futuras.
- O uso de automação no WhatsApp deve respeitar as regras da plataforma e ser feito por responsabilidade do usuário.

## Créditos

- Base original: [WhiskeySockets/Baileys](https://github.com/WhiskeySockets/Baileys)
- Fork/modificações: [srlczinn/new-baileys](https://github.com/laerciocdev/new-baileys)

## Licença

MIT. Consulte o arquivo [LICENSE](LICENSE).
