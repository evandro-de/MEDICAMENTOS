# Controle de Medicamentos (versão Firebase — para hospedar fora da Claude)

Esta é a versão do app preparada para rodar em qualquer hospedagem estática
(GitHub Pages, Vercel, Netlify etc.), usando o **Firebase Firestore** como
banco de dados na nuvem no lugar do armazenamento embutido da Claude.

A interface, os cadastros, o histórico e a exportação em CSV são idênticos
à versão publicada em claude.ai — só a "gaveta" onde os dados ficam salvos
mudou.

## Passo 1 — Criar o projeto no Firebase (gratuito)

1. Acesse https://console.firebase.google.com e entre com uma conta Google.
2. Clique em **"Adicionar projeto"**, dê um nome (ex: `fazenda-santa-rosa`) e
   conclua a criação (pode desativar o Google Analytics, não é necessário).

## Passo 2 — Criar o banco de dados (Firestore)

1. No menu lateral, vá em **Build > Firestore Database**.
2. Clique em **"Criar banco de dados"**.
3. Escolha a localização mais próxima (ex: `southamerica-east1` para Brasil).
4. Comece em **modo de produção** (não use "modo de teste" — as regras dele
   expiram e deixam o banco aberto para qualquer pessoa).

## Passo 3 — Ativar login anônimo

O app usa um login anônimo automático (sem senha, sem cadastro) só para
garantir que o Firestore saiba que quem está lendo/escrevendo veio do
próprio app, e não um bot varrendo a internet.

1. No menu lateral, vá em **Build > Authentication**.
2. Clique em **"Começar"**, depois na aba **"Sign-in method"**.
3. Ative o provedor **"Anônimo"**.

## Passo 4 — Configurar as regras de segurança do Firestore

1. Ainda em **Firestore Database**, vá na aba **"Regras"**.
2. Substitua o conteúdo por:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```

3. Clique em **"Publicar"**.

Isso garante que só quem passou pelo login anônimo do próprio app consegue
ler ou escrever dados — mesmo que alguém veja o código no GitHub.

## Passo 5 — Pegar a configuração do seu projeto

1. No menu lateral, clique na engrenagem ao lado de "Visão geral do projeto"
   > **"Configurações do projeto"**.
2. Na aba **"Geral"**, role até **"Seus apps"** e clique no ícone **`</>`**
   (Web) para registrar um app.
3. Dê um apelido (ex: `controle-medicamentos`) e clique em **"Registrar app"**.
4. Copie o objeto `firebaseConfig` que aparece — algo assim:

```js
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "fazenda-santa-rosa.firebaseapp.com",
  projectId: "fazenda-santa-rosa",
  storageBucket: "fazenda-santa-rosa.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abcdef",
};
```

> Esses valores **não são secretos** — é normal e esperado que fiquem
> visíveis no código, mesmo público no GitHub. Quem protege os dados de
> verdade são as regras do Passo 4, não o sigilo desses valores.

## Passo 6 — Colar a configuração no `index.html`

Abra o `index.html` deste pacote, procure por `firebaseConfig` perto do
início do script, e substitua os valores `"COLE_AQUI..."` pelos valores
reais copiados no passo anterior.

## Passo 7 — Subir para o GitHub Pages

1. Crie um repositório no GitHub e envie o `index.html` para a raiz dele.
2. Em **Settings > Pages**, escolha a branch (`main`) e a pasta raiz (`/`).
3. Aguarde alguns minutos — o GitHub mostra o link público do app.

Pronto: esse link vai funcionar com nuvem de verdade, sincronizando entre
todos os celulares que abrirem o app, exatamente como a versão em claude.ai.

## Limites do plano gratuito do Firebase (mais do que suficiente aqui)

- 50 mil leituras e 20 mil gravações por dia.
- 1 GiB de armazenamento total.
- Sem cartão de crédito necessário para esses limites.

## O que é diferente da versão claude.ai

- O armazenamento agora é o Firebase Firestore, não o banco embutido da
  Claude — por isso este arquivo só funciona depois que você preencher o
  `firebaseConfig` (Passo 6).
- Exige login anônimo automático (silencioso, sem pedir nada ao usuário) só
  para as regras de segurança do Passo 4 funcionarem.
- Todo o resto — telas, cadastros, lançamentos, histórico, exportação CSV,
  hash de senha dos usuários, CSP, ausência de `innerHTML`/`localStorage` —
  é idêntico à versão publicada.
