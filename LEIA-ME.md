# Controle de Fretes — como colocar no ar

App de uso pessoal (só você acessa), com login, dados salvos na nuvem (Firestore)
e sincronização automática entre celular e computador. Segue o mesmo padrão dos
seus outros apps: `index.html` único + Firebase + Vercel, sem build, sem ambiente local.

## 1. Criar (ou reaproveitar) um projeto Firebase

Pode usar um projeto novo só para este app, ou um dos que você já tem.

1. Acesse https://console.firebase.google.com e crie um projeto (ou abra um existente).
2. No menu lateral, vá em **Build > Authentication > Sign-in method** e ative o
   provedor **E-mail/senha**.
3. Vá em **Build > Firestore Database** e clique em **Criar banco de dados**
   (pode escolher a região `southamerica-east1` — mais perto do Brasil).
4. Em **Configurações do projeto (ícone de engrenagem) > Geral**, role até
   "Seus aplicativos" e clique no ícone `</>` para registrar um app da Web.
   O Firebase vai te dar um objeto `firebaseConfig` — copie ele.

## 2. Colar a configuração no `index.html`

Abra o `index.html` e substitua este trecho perto do topo do `<script>`:

```js
const firebaseConfig = {
  apiKey: "COLE_AQUI",
  authDomain: "COLE_AQUI.firebaseapp.com",
  projectId: "COLE_AQUI",
  storageBucket: "COLE_AQUI.appspot.com",
  messagingSenderId: "COLE_AQUI",
  appId: "COLE_AQUI"
};
```

pelos valores reais que o Firebase te deu.

## 3. Regras de segurança do Firestore

Isso é o que garante que só você (autenticado) vê os seus dados. Em
**Firestore Database > Regras**, cole:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/{document=**} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

Clique em **Publicar**.

## 4. Criar seu acesso

Na primeira vez que abrir o app (local ou já publicado), clique em
"Ainda não tenho conta — criar acesso", digite seu e-mail e uma senha, e pronto:
sua conta fica ligada a esse projeto Firebase. Depois disso é só entrar com
e-mail e senha em qualquer aparelho.

> Dica: se quiser impedir que qualquer pessoa crie uma conta nova pelo app
> (caso o link vaze), depois de criar a sua conta você pode ir em
> **Authentication > Sign-in method > E-mail/senha** e, futuramente, trocar
> a tela de cadastro por um convite manual seu direto no console. Para uso
> pessoal do dia a dia isso não é obrigatório.

## 5. Subir pro GitHub e conectar no Vercel

Igual você já faz nos outros projetos:

1. Crie um repositório novo no GitHub (ex: `controle-fretes`).
2. Suba os arquivos: `index.html`, `manifest.json`, `sw.js` e a pasta `icons/`
   (com `icon-192.png` e `icon-512.png`) — pelo editor web do GitHub mesmo.
3. No Vercel, importe esse repositório como novo projeto. Como não tem build
   (é só HTML/CSS/JS puro), pode deixar o "Framework Preset" como **Other** —
   não precisa de comando de build nem pasta de output especial.
4. Após o deploy, abra a URL, crie seu acesso (passo 4) e comece a cadastrar
   os fretes.

## 6. Instalar como app no celular

Com o site já publicado, abra a URL no Chrome do celular e use
"Adicionar à tela inicial" — vira um app com ícone próprio, como os outros
PWAs que você já usa.

---

**Estrutura de dados no Firestore** (caso queira consultar/exportar depois):

```
users/{seu-uid}/fretes/{id}       → origem, destino, dataInicio, dataFim, status, faturamento, observacoes
users/{seu-uid}/despesas/{id}     → categoria, valor, data, freteId (ou null), descricao
users/{seu-uid}/meta/config       → reinvestimento, emergencia, desgaste, viagem (percentuais)
```
