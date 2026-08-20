# DP Lynwood — Painel do Comando (site privado)

Repositório separado, pronto pra publicar com **GitHub Pages**. Este é o painel **restrito** onde o Comando vê todos os registros enviados, sinaliza linguagem imprópria, bane IP/aparelho e troca a própria senha de acesso.

O site público onde qualquer pessoa envia sugestão/reclamação (`ouvidoria.html`) fica num **repositório à parte** — veja a seção 4.

| Arquivo | O que é |
|---|---|
| `index.html` | O Painel do Comando em si (nomeado `index.html` pra virar a página raiz no GitHub Pages). |

Usa o mesmo projeto Firebase que você já tinha (`comando-geral-98c87`) — **sem precisar abrir o Firebase Console em nenhum momento.**

---

## 1. Subir para o GitHub e ativar o Pages

⚠️ **Este repositório dá acesso ao painel de moderação — considere deixá-lo PRIVADO no GitHub** (Settings → General → Danger Zone → Change visibility), já que a URL de login ficará pública se o repo for público.

1. Crie um repositório novo no GitHub, por exemplo `comando-lynwood`.
2. Suba `index.html` para a raiz do repositório:
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
6. Abra `index.html` neste repositório e troque o link **📢 Ouvidoria** (perto do topo do arquivo, dentro do `<header class="topbar">`) pela URL real do repositório da Ouvidoria, depois de publicá-lo.

## 2. Login — pronto para usar, sem tocar no Firebase Console

Usuário: **`adm`** · Senha: **`admin1234`**

Isso já funciona assim que você publica o site — não precisa criar nada no Firebase, ativar Authentication, nem colar regra nenhuma. Veja como:

- O painel **não usa** o serviço de Authentication do Firebase (que exigiria você entrar com sua conta Google no Console para ativar). Em vez disso, o usuário e a senha ficam guardados dentro do próprio banco de dados (Realtime Database), como um hash — nunca como texto puro.
- Na **primeira vez** que alguém abre o painel publicado, ele mesmo cria esse login padrão automaticamente.
- Depois de entrar, é possível trocar a senha a qualquer momento pelo botão **⚙ Senha**, dentro do próprio painel.

**Leitura importante sobre segurança:** como não existe Firebase Authentication aqui, esta tela de login é uma **trava de acesso ao painel**, não uma trava criptográfica do banco de dados em si. Ela funciona porque, para o site inteiro funcionar sem você tocar no Firebase Console, o projeto precisa já estar com as regras do Realtime Database relativamente abertas (o padrão de projetos novos em "modo de teste", que é bem comum). Nesse cenário, alguém que soubesse a URL direta do seu banco de dados poderia, em teoria, ler ou escrever nos dados sem passar pela tela de login — a tela protege o *painel*, não substitui uma trava no *banco* em si (isso só é possível com Firebase Authentication de verdade, que exige aqueles cliques no Console que você pediu para evitar). Se algum dia quiser esse nível extra de segurança, é só pedir — mostro exatamente os 2 cliques que fariam isso.

## 3. Testar

1. Abra a URL do GitHub Pages, faça login com `adm` / `admin1234`.
2. Os registros enviados pela Ouvidoria devem aparecer, com o selo **⚠ Linguagem imprópria** se aplicável.
3. Clique em **Banir IP** ou **Banir aparelho** num registro de teste, depois tente enviar outro pelo mesmo navegador na Ouvidoria — deve aparecer a mensagem de bloqueio lá.
4. Clique em **⚙ Senha**, informe a senha atual e uma nova, confirme — deslogue e entre de novo com a senha nova para conferir.

Se aparecer erro de conexão ao tentar logar, o mais provável é que as regras do seu Realtime Database estejam bloqueando escrita/leitura pública — nesse caso específico, ajustar isso exigiria sim abrir o Firebase Console (não há outro jeito, é uma configuração do seu projeto que só você pode mudar).

## 4. Sobre o repositório da Ouvidoria

Este repositório só tem o Painel do Comando. O site público onde qualquer pessoa envia sugestão/reclamação (`ouvidoria.html`) fica em um **segundo repositório separado**, com seu próprio README. Depois de publicar os dois, edite o link **📢 Ouvidoria** no topo deste painel (passo 6 da seção 1) pra apontar pra URL real dele.

---

## O que foi corrigido/mudado nesta versão

- **Falso positivo no filtro de palavrões** (na Ouvidoria): a palavra "cu" estava sendo detectada dentro de palavras comuns como "escuro". Corrigido com detecção por limite de palavra.
- **Login local, sem Firebase Console:** trocado o login por e-mail/senha do Firebase Authentication por um login guardado no próprio banco de dados — nenhuma etapa manual no Firebase é mais necessária para o painel funcionar.
- **Troca de senha pelo próprio painel**, sem depender de nenhum serviço externo.

## Sobre o banimento de IP/aparelho (leitura importante)

- **IP**: obtido via serviço público (ipify) no navegador de quem envia. Bloqueia bem a mesma rede/pessoa insistindo, mas troca de rede/VPN contorna.
- **Aparelho**: identificador salvo no navegador (`localStorage`). Some se a pessoa limpar os dados do navegador ou usar outro navegador/aba anônima.
- Por isso o painel guarda também o **nome digitado** — cruzando nome + IP + aparelho dá pra identificar reincidentes na prática.

## Estrutura dos dados (Realtime Database)

```
/suggestions/{id}: name, message, hasProfanity, profanityWords, ip, deviceId, fingerprint, userAgent, status, createdAt
/banned/{id}: type ("ip" | "device"), value, reason, bannedAt
/panelConfig/auth: user, salt, hash  (login do painel — senha nunca fica em texto puro)
```

Qualquer ajuste, é só chamar.
