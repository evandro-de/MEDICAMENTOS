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

## Banco de dados pelos arquivos do GitHub (sem nuvem)

O banco de animais e o histórico de sanidade **não vão mais para a nuvem**: vêm
de dois arquivos que ficam no GitHub, na mesma pasta do `index.html`, e o Render
publica junto com o app.

| Arquivo                                  | O que é                                    |
|------------------------------------------|--------------------------------------------|
| `ANIMAIS_APK.zip` (ou `.csv`)            | animais (SISBOV) e o local atual de cada um |
| `sanidade.zip` (ou `.csv`)               | histórico de medicações já aplicadas        |
| `manifest.json` (opcional)               | data de atualização de cada arquivo         |

**Use `.zip`**: o `sanidade.csv` tem ~25 MB (o GitHub trava em 25 MB e demora
para subir); compactado vira ~1 MB. Os dois `.zip` que vêm neste pacote já estão
prontos — **suba-os como estão, sem descompactar**.

### Como atualizar o banco (passo a passo)

1. No computador: botão direito no arquivo novo (ex.: `sanidade.csv`) →
   **Enviar para → Pasta compactada**. Fica `sanidade.zip` (e `ANIMAIS_APK.zip`).
2. No GitHub, abra o repositório → **Add file → Upload files** → envie o `.zip`
   (mesmo nome: ele substitui o antigo) → **Commit changes**.
3. Espere o Render publicar: no Render, aba **Events** → "Deploy live".
4. No app: **Cadastros → Recarregar dados agora** (ou só abra o app de novo).

O app aceita `.zip` ou `.csv`, na raiz ou numa pasta chamada `dados/`, e também
`animais_apk`/`animais` em minúsculas. Não importa o nome do arquivo *dentro* do zip.

### Se algo não funcionar

Abra **Cadastros → Banco de dados**: os **Detalhes da busca** (abrem sozinhos
quando há erro) mostram o que o app encontrou em cada nome de arquivo e o motivo
técnico, e logo abaixo há o endereço do app, o estado do `manifest.json` e se a
internet do aparelho está ligada.

| O que aparece                                  | O que fazer                                                    |
|------------------------------------------------|----------------------------------------------------------------|
| "não existe (404)" em todos                    | o arquivo não subiu, ou está em outra pasta: confira no GitHub |
| "o site devolveu uma página HTML"              | no Render → **Redirects/Rewrites**, apague a regra que manda `/*` para `/index.html` |
| "formato inválido"                             | arquivo errado (ou planilha sem as colunas esperadas)          |
| "não consegui abrir o zip"                     | refaça o zip (botão direito → Enviar para → Pasta compactada)  |
| "Sem conexão com o site"                       | a internet do aparelho está fora/fraca: tente de novo com sinal, ou use o **Plano B** |
| "embora o site esteja respondendo"             | algo no aparelho/rede bloqueia o download: use o `.zip` (bem menor) ou o **Plano B**; teste sem bloqueador de anúncios |
| "o app foi aberto direto do arquivo (file://)" | abra pelo endereço do Render (https://...onrender.com), não o arquivo do computador |
| dados antigos depois de subir                  | espere "Deploy live" no Render e toque em **Recarregar dados agora** |

**Plano B (sem GitHub):** em **Cadastros → Plano B: carregar arquivo deste
aparelho**, escolha o `.csv` ou `.zip` direto do celular/computador (pode
escolher os dois de uma vez; o app reconhece qual é qual). Vale só para aquele
aparelho e não vai para o GitHub nem para a nuvem. O arquivo mais novo (do
aparelho ou do site) é o que vale; "Voltar a usar só os arquivos do site"
desfaz.

### Como o app carrega

Cada aparelho baixa e processa os arquivos uma vez e guarda o resultado; nas
próximas aberturas só confere se mudou (menos de 1 segundo). Se o celular ficar
sem internet, usa o que já tem guardado.

A data de "atualizado em" é a mais recente entre a do `manifest.json` e a do
arquivo no servidor.

### Atenção

- **Privacidade:** repositório público e site do Render são públicos — qualquer
  pessoa com o endereço consegue baixar os arquivos (animais, locais e histórico
  de medicações). Se isso for um problema, deixe o repositório **privado** (o
  Render publica de repositório privado) e não divulgue o endereço do site. O PIN
  do app protege a tela, não os arquivos.
- O que já estava gravado no Firebase pela importação antiga (`animais/*`,
  `config/sanidade_*`, `config/animais_index`) não é mais usado e pode ser
  apagado no console do Firestore. Lançamentos, cadastros e usuários continuam
  no Firebase normalmente.

## Relatório de medicação (aba Histórico)

O relatório tem uma linha por medicamento aplicado, com as colunas:
`Identificacao;Farmaco;Dose;Motivo;Data;Lote;Modalidade;Aplicador;Protocolo;Temperatura;CurralDestino`
(o **curral de destino** vem logo depois da temperatura).

- **Extrair Excel (.xlsx):** o SISBOV sai como **número inteiro** (formato `0`,
  sem notação científica); dose, lote e temperatura numéricos também viram número.
- **Extrair CSV:** o SISBOV vai só com dígitos, sem aspas, como número. Um CSV
  não guarda o tipo da célula, por isso o Excel costuma mostrá-lo como `1,05E+14`
  — para ver inteiro, use o botão Excel.

### Não repetir o que já foi lançado no sistema

Cada lançamento guarda uma marca de **exportado** (`exportadoEm`). Na aba
Histórico → "Extrair relatório de medicação":

- O relatório leva **só os lançamentos novos** (ainda não exportados) do período.
  Depois de baixar com sucesso, eles ficam marcados como "já lançados no sistema",
  e uma próxima exportação não os traz de novo.
- Se o envio for cancelado (ou o arquivo não baixar), **nada é marcado**.
- **Marcar como "já lançados" ao exportar:** desmarque se for só para conferir.
- **Incluir também os já exportados:** exporta tudo de novo, sem mudar a data da
  exportação original.
- **Desfazer:** volta a última exportação para "novo". **Só marcar … sem exportar:**
  serve para o que já foi lançado no sistema antes desta função existir.
- Na lista, cada cartão mostra "novo" ou "✓ exportado" e dá para filtrar
  (Todos / Novos / Já exportados) e marcar ou desmarcar um lançamento sozinho.
- Se corrigir um lançamento que já foi exportado, use "Marcar como não exportado"
  nele para ele sair de novo na próxima exportação.

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
