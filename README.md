# PetiscaPeixe

Sistema de operacao para restaurante com:

- painel administrativo
- atendimento por garcom via PIN
- telas de cozinha, bar e caixa
- cardapio publico por QR Code
- impressao local por agente Windows

## Stack

- React 19
- Vite
- TypeScript
- Firebase Authentication
- Cloud Firestore

## Fluxos principais

- `admin`, `manager`, `cashier`, `kitchen` e `bar` entram com Google
- `waiter` entra com PIN e recebe uma sessao autenticada anonima no Firebase, restrita ao papel de garcom
- pedidos geram `orders`, `orderItems`, movimentos de estoque e `printJobs`
- o dispositivo do caixa pode consumir `printJobs` e enviar para o agente local em `http://localhost:17321`

## Executar localmente

1. Instale as dependencias:
   `npm install`
2. Configure o Firebase do projeto.
   O app usa [`firebase-applet-config.json`](firebase-applet-config.json) na raiz do repositório como configuração do cliente.
3. Ative no Firebase Authentication:
   `Google`
   `Anonymous`
4. Rode o ambiente:
   `npm run dev`

## Build

- `npm run build`
- `npm run preview`

O build empacota tambem o agente local de impressao em `public/PrintAgent.exe`.

## Imagens dos produtos (Firebase Storage)

O Firestore **não armazena arquivos binários**; ele guarda apenas o campo `imageUrl`. As fotos vão para o **Firebase Storage** (bucket em `firebase-applet-config.json`).

1. No [Console Firebase](https://console.firebase.google.com) → **Build → Storage** → **Começar** (plano Blaze / créditos cobrem o uso típico de cardápio).
2. Publique as regras: `npm run deploy:storage` (ou `npm run deploy:firebase` com as regras do Firestore).
3. Em **Authentication → Settings → Domínios autorizados**, inclua o domínio onde o app roda (ex.: produção e `localhost`).

## Publicar regras do Firestore (Firebase CLI)

O repositório inclui `firebase.json` e `.firebaserc` alinhados ao projeto e ao **banco nomeado** em `firebase-applet-config.json` (`firestoreDatabaseId`). Editar só `firestore.rules` no Git **não** atualiza a nuvem até você fazer deploy.

**Uma vez na máquina (login interativo):**

1. `npm install`
2. `npx firebase login` — abre o navegador e associa sua conta Google ao CLI.

**Sempre que quiser enviar regras:**

- Firestore: `npm run deploy:rules`
- Storage: `npm run deploy:storage`
- Ambos: `npm run deploy:firebase`

**Automação (CI ou ambiente sem browser):** na máquina onde já fez login, rode `npx firebase login:ci`, guarde o token com segredo (ex.: variável `FIREBASE_TOKEN` no GitHub) e use `firebase deploy --only firestore:rules` com esse token na pipeline. Não commite o token.

## Observacoes operacionais

- as regras do Firestore dependem das roles em `users` e `waiterSessions`
- o cadastro de garcons continua sendo feito na tela administrativa `Garcons`
- a impressao automatica depende de configuracao em `Configuracoes` e do agente local ativo
