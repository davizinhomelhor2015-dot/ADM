# DP Lynwood — Painel do Comando (site privado)

Repositório separado, pronto pra publicar com **GitHub Pages**. Este é o painel **restrito** onde o Comando vê todos os registros enviados, sinaliza linguagem imprópria, bane IP/aparelho e troca a própria senha de acesso.

O site público onde qualquer pessoa envia sugestão/reclamação (`ouvidoria.html`) fica num **repositório à parte** — veja a seção 5.

| Arquivo | O que é |
|---|---|
| `index.html` | O Painel do Comando em si (nomeado `index.html` pra virar a página raiz no GitHub Pages). |
| `database.rules.json` | Trecho de regras do Realtime Database — igual ao do outro repositório, só precisa publicar uma vez (ver seção 2). |

Usa o mesmo projeto Firebase que você já tinha (`comando-geral-98c87`).

---

## 1. Subir para o GitHub e ativar o Pages

⚠️ **Este repositório dá acesso ao painel de moderação — considere deixá-lo PRIVADO no GitHub** (Settings → General → Danger Zone → Change visibility), já que a URL de login ficará pública se o repo for público. Repositórios privados também funcionam com GitHub Pages (em contas Pro/Team) — se sua conta não tiver isso, o repositório pode ficar público mesmo assim: o login continua protegido pelo Firebase Authentication, só a URL fica descoberta mais facilmente.

1. Crie um repositório novo no GitHub, por exemplo `comando-lynwood`.
2. Suba `index.html` e `database.rules.json` para a raiz do repositório:
   ```
   git init
   git add .
   git commit -m "Painel do Comando DP Lynwood"
   git branch -M main
   git remote add origin https://github.com/SEU-USUARIO/comando-lynwood.git
   git push -u origin main
   ```
3. No GitHub, vá em **Settings → Pages**.
4. Em **Source**, selecione **Deploy from a branch** → branch **main** → pasta **/ (root)** → **Save**.
5. Espere 1–2 minutos. A URL final fica algo como:
   `https://SEU-USUARIO.github.io/comando-lynwood/`
6. Abra `index.html` neste repositório e troque o link **📢 Ouvidoria** (perto do topo do arquivo, dentro do `<header class="topbar">`) pela URL real do repositório da Ouvidoria, depois de publicá-lo — hoje ele aponta para um endereço de exemplo (`SEU-USUARIO.github.io/ouvidoria-lynwood`).

## 2. Publicar as regras do Realtime Database

Se você já publicou as regras ao configurar o repositório da Ouvidoria, **pode pular esta seção** — é a mesma regra pros dois sites, não precisa fazer de novo.

Caso ainda não tenha feito:

1. Acesse [console.firebase.google.com](https://console.firebase.google.com) → projeto `comando-geral-98c87` → **Compilação → Realtime Database → Regras**.
2. Copie o conteúdo de `database.rules.json` e **cole dentro** do seu JSON de regras atual, dentro de `"rules": { ... }`, ao lado do que já existe — não apague nada que já estava lá. Fica assim, por exemplo:

```json
{
  "rules": {

    /* ↓↓↓ o que você já tinha continua aqui, sem mudar ↓↓↓ */

    /* ↑↑↑ até aqui ↑↑↑ */

    "suggestions": {
      ".read": "auth != null",
      "$id": {
        ".write": "auth != null || !data.exists()",
        ".validate": "newData.hasChildren(['name','message']) && newData.child('name').isString() && newData.child('name').val().length > 0 && newData.child('name').val().length <= 60 && newData.child('message').isString() && newData.child('message').val().length > 0 && newData.child('message').val().length <= 1000"
      }
    },

    "banned": {
      ".read": true,
      ".write": "auth != null",
      ".indexOn": ["value"]
    }
  }
}
```

> (Remova os comentários `/* ... */` — JSON de verdade não aceita comentários.)

3. Clique em **Publicar**.

## 3. Ativar o login do Comando

A tela de login pede só **Usuário** e **Senha** — não precisa digitar e-mail. Por baixo dos panos, o Firebase Authentication continua exigindo um identificador em formato de e-mail (é assim que o serviço funciona), então o painel converte automaticamente, por exemplo `adm` → `adm@dplynwood.local`, sem quem faz login precisar saber disso. Se digitar algo com "@", o painel usa exatamente o que foi digitado — então um e-mail de verdade também continua funcionando, se preferir.

1. **Compilação → Authentication → Sign-in method** → ative **E-mail/senha**.
2. Aba **Users** → **Add user** → cadastre o usuário já convertido, por exemplo `adm@dplynwood.local`, com a senha desejada (mín. 6 caracteres). Pode cadastrar mais de uma pessoa (ex.: `comando@dplynwood.local`, `patrulha1@dplynwood.local`...).
3. Na tela de login do painel, a pessoa digita só `adm` (sem o `@dplynwood.local`) e a senha.
4. Depois de logado, cada pessoa pode trocar a própria senha a qualquer momento clicando em **⚙ Senha**, dentro do próprio painel — não precisa voltar ao Firebase Console.

Alternativa via linha de comando (sem abrir o Console), usando a `apiKey` pública do projeto — troque `adm` pelo usuário e `admin1234` pela senha que quiser:

```bash
curl -s "https://identitytoolkit.googleapis.com/v1/accounts:signUp?key=AIzaSyDVIKbfNVw4_iQGdiLrJhRfcPSVNjeZDLU" \
  -H "Content-Type: application/json" \
  -d '{"email":"adm@dplynwood.local","password":"admin1234","returnSecureToken":true}'
```
Rode isso no terminal do seu computador. Se `EMAIL_EXISTS` aparecer, esse usuário já foi criado antes (pode logar direto). O domínio `@dplynwood.local` é só um identificador interno — não precisa existir de verdade, o Firebase não envia e-mail nenhum para lá.

> **Login de exemplo já pronto pra testar:** usuário `adm`, senha `admin1234` — é só rodar o comando acima uma vez pra criar essa conta no seu projeto. Depois, troque para uma senha mais forte pelo próprio painel (**⚙ Senha**).

## 4. Testar

1. Abra a URL do GitHub Pages, faça login com o usuário e senha cadastrados.
2. Os registros enviados pela Ouvidoria devem aparecer, com o selo **⚠ Linguagem imprópria** se aplicável.
3. Clique em **Banir IP** ou **Banir aparelho** num registro de teste, depois tente enviar outro pelo mesmo navegador na Ouvidoria — deve aparecer a mensagem de bloqueio lá.
4. Clique em **⚙ Senha**, informe a senha atual e uma nova, confirme — deslogue e entre de novo com a senha nova para conferir.

## 5. Sobre o repositório da Ouvidoria

Este repositório só tem o Painel do Comando. O site público onde qualquer pessoa envia sugestão/reclamação (`ouvidoria.html`) fica em um **segundo repositório separado**, com seu próprio README explicando como publicar e configurar as regras. Depois de publicar os dois, edite o link **📢 Ouvidoria** no topo deste painel (passo 6 da seção 1) pra apontar pra URL real dele.

---

## O que foi corrigido nesta versão

- **Falso positivo no filtro de palavrões** (na Ouvidoria): a palavra "cu" estava sendo detectada dentro de palavras comuns como "escuro". Corrigido com detecção por limite de palavra.
- **Brecha de segurança nas regras do banco:** a regra pública de escrita permitia sobrescrever/apagar sugestões de outras pessoas. Agora é "somente criação" pública — editar ou apagar exige login do Comando.
- **Login por usuário**, sem precisar digitar e-mail.
- **Troca de senha pelo próprio painel**, sem depender do Firebase Console.

## Sobre o banimento de IP/aparelho (leitura importante)

- **IP**: obtido via serviço público (ipify) no navegador de quem envia. Bloqueia bem a mesma rede/pessoa insistindo, mas troca de rede/VPN contorna.
- **Aparelho**: identificador salvo no navegador (`localStorage`). Some se a pessoa limpar os dados do navegador ou usar outro navegador/aba anônima.
- Por isso o painel guarda também o **nome digitado** — cruzando nome + IP + aparelho dá pra identificar reincidentes na prática.

## Estrutura dos dados (Realtime Database)

```
/suggestions/{id}: name, message, hasProfanity, profanityWords, ip, deviceId, fingerprint, userAgent, status, createdAt
/banned/{id}: type ("ip" | "device"), value, reason, bannedAt
```

Qualquer ajuste, é só chamar.
