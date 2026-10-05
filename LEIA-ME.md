# Receitas — pôr no GitHub e gerar o APK

Esta pasta já tem tudo. Não precisas de instalar nada no computador: é o GitHub
que compila o APK, de graça.

Depois de montado, cada versão nova é **substituir um ficheiro e carregar em
guardar**. O APK aparece sozinho uns minutos depois, e a app avisa-te dentro do
telemóvel em *Definições → Verificar atualizações*.

---

## O que está aqui dentro

```
www/index.html              a app
package.json                as dependências que o GitHub instala
capacitor.config.json       nome da app e identificador
resources/icon.png          o ícone
.github/workflows/apk.yml   as instruções de compilação
.gitignore                  ficheiros que não vão para o GitHub
```

---

# Parte 1 — montar (uma vez só)

## 1. Criar o repositório

1. Vai a **https://github.com/new**
2. **Repository name:** `receitas`
3. Deixa **Public**. *(Privado também funciona, mas aí o link do APK pede sessão
   iniciada sempre que o quiseres descarregar no telemóvel.)*
4. **Não** marques nada em "Initialize this repository with" — nem README, nem
   .gitignore, nem licença. A pasta já traz tudo.
5. **Create repository**

## 2. Enviar os ficheiros

Na página que aparece a seguir, clica em **uploading an existing file**.

Depois, no explorador do Windows:

1. Abre esta pasta
2. Seleciona tudo com **Ctrl+A**
3. Arrasta para a página do GitHub
4. Em baixo, escreve `primeira versão` e carrega em **Commit changes**

> **Atenção à pasta `.github`.** O Windows esconde pastas que começam por ponto.
> Se depois do upload não vires a pasta `.github` na lista de ficheiros do
> repositório, nada vai ser compilado. Para a enviar: no GitHub carrega em
> **Add file → Create new file**, escreve no nome
> `.github/workflows/apk.yml` (as barras criam as pastas sozinhas), e cola lá
> dentro o conteúdo do ficheiro que está nesta pasta.

## 3. Ligar as Actions

1. No repositório, separador **Actions**
2. Se aparecer um aviso verde, carrega em **I understand my workflows, go ahead
   and enable them**

A primeira compilação arranca sozinha. Demora **5 a 10 minutos** na primeira vez
(o GitHub tem de descarregar o Gradle e as ferramentas do Android); nas
seguintes são 3 a 4.

## 4. Fixar a chave de assinatura — **não saltes este passo**

Um APK é assinado com uma chave. Se a chave mudar entre versões, o Android
recusa-se a instalar a atualização por cima e dá um *"Aplicação não instalada"*
sem explicar porquê. Por omissão, cada compilação no GitHub cria uma chave nova.

Para fixar a chave, vamos usar a que o Android Studio já criou no teu
computador.

**a)** Abre o **PowerShell** (tecla Windows, escreve `powershell`) e cola:

```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("$env:USERPROFILE\.android\debug.keystore")) | Set-Clipboard
```

Isto copia a chave para a área de transferência. Não mostra nada — é normal.

> **Se der erro a dizer que não encontra o ficheiro**, é porque o Android Studio
> ainda não criou a chave. Abre o Android Studio, deixa-o terminar o arranque, e
> tenta outra vez. Em último caso, salta este passo: o APK sai na mesma, mas
> para atualizares tens de desinstalar a app antes de instalar a nova.

**b)** No repositório do GitHub:

1. **Settings** (no topo do repositório, não o das tuas definições de conta)
2. Menu da esquerda: **Secrets and variables → Actions**
3. Botão **New repository secret**
4. **Name:** `DEBUG_KEYSTORE_B64`
5. **Secret:** cola com **Ctrl+V**
6. **Add secret**

**c)** Volta a **Actions → Criar APK → Run workflow → Run workflow** para
compilar outra vez, agora com a chave fixa.

> Esta chave serve só para assinar a tua app pessoal. Não dá acesso à tua conta
> nem a mais nada. Se um dia quiseres publicar na Play Store, aí sim é preciso
> uma chave de distribuição a sério — diz-me quando chegar a altura.

## 5. Instalar no telemóvel

1. No telemóvel, abre o repositório e vai a **Releases** (à direita, ou em
   `github.com/OTEUNOME/receitas/releases`)
2. Descarrega o ficheiro `receitas-0.4.apk`
3. Abre-o. O Android vai pedir autorização para **instalar aplicações
   desconhecidas** — é normal, porque o APK não vem da Play Store. Aceita.

> **O identificador da app mudou** de `com.bbtecnodev.padaria` para
> `com.bbtecnodev.receitas`, porque a app deixou de ser só da padaria. Se já
> tinhas instalado uma versão com o nome antigo, o Android trata esta como
> outra app e ficas com as duas — desinstala a antiga. Como ainda não chegaste
> a publicar nenhuma, esta é a altura certa para mudar.

## 6. Apontar a app ao repositório

Dentro da app: **Definições → Atualizações → Repositório no GitHub**

Escreve `Bernas717/receitas` (ou o nome que deste ao repositório) e
sai do campo. A partir daqui, o botão **Verificar** diz-te quando há versão
nova.

---

# Parte 2 — lançar uma versão nova

É isto, de cada vez:

1. Abre `www/index.html` no GitHub e carrega no **lápis** (ou **Add file →
   Upload files** e arrasta o ficheiro novo por cima)
2. **Commit changes**
3. Espera 3 a 4 minutos
4. Em **Releases** aparece o APK novo

No telemóvel: **Definições → Verificar** → descarregar → instalar por cima.

## A versão tem de subir

O número da versão sai do próprio ficheiro da app, da primeira entrada do
registo de alterações:

```js
const CHANGELOG = [
 { v:'0.4', data:'05-10-2026', ...
```

Se enviares um ficheiro com a mesma versão, a release é substituída e a app não
dá por nada — continua a achar que já está atualizada. **Sobe sempre o número**
quando mandas uma versão nova.

A regra que combinámos: **uma versão por dia de trabalho**. Alterações no mesmo
dia entram na mesma versão; no dia seguinte sobe para a próxima (0.4 → 0.5).

---

# Quando corre mal

**Em Actions aparece uma cruz vermelha**
Clica na linha que falhou, depois no passo com a cruz, e lê a última linha a
vermelho. Manda-me essa mensagem.

**Não aparece nada em Actions**
A pasta `.github` não foi enviada. Vê o aviso do passo 2.

**O APK não instala: "Aplicação não instalada"**
É a chave. Faz o passo 4, ou desinstala a app antiga e instala de novo.

**A app diz "Não encontrei esse repositório"**
O nome está errado em *Definições → Repositório*, ou ainda não há nenhuma
release publicada. Confirma em `github.com/OTEUNOME/receitas/releases`.

**A app diz que já está atualizada mas eu enviei uma versão nova**
Não subiste o número da versão no `CHANGELOG`.

**A compilação demora imenso**
A primeira vez demora. Se passar de 15 minutos, alguma coisa está presa — manda
o registo.

---

# O que fica a custar

Nada. Repositórios públicos têm Actions sem limite; privados têm 2000 minutos
por mês, e cada compilação destas gasta 3 a 5.
