# Guia de Configuração de Credenciais - Workflow Onfly

## Introdução

Este documento detalha o processo de configuração das credenciais necessárias para o funcionamento do workflow de automação de relatórios financeiros. O fluxo de trabalho se conecta a três serviços externos distintos e, portanto, requer três tipos de credenciais:

1.  **IMAP:** Para ler e receber os e-mails da caixa de entrada.
2.  **Google Drive API:** Para criar pastas e salvar os arquivos de anexo.
3.  **SMTP:** Para enviar os e-mails de confirmação e de erro.

Siga os passos abaixo para configurar cada uma delas.

---

## 1. Credencial de Leitura de E-mail (IMAP) [Link da documentação](https://docs.n8n.io/integrations/builtin/credentials/imap/)

Esta credencial permite que o n8n acesse a caixa de entrada para ler os e-mails que chegam.

- **Nó que utiliza esta credencial:** `1. Receber E-mail (Gatilho)`

### Passo a Passo para Configuração

1.  No workflow do n8n, vá até o nó `1. Receber E-mail (Gatilho)`.
2.  Clique no campo **Credentials**.
3.  Selecione **"Create new credential"**.
4.  Preencha os seguintes campos:
    - **Name:** Dê um nome de fácil identificação (ex: `Gmail - financeiro@onfly.com`).
    - **Host:** O endereço do servidor IMAP do seu provedor (ex: `imap.gmail.com`).
    - **Port:** A porta do servidor IMAP (ex: `993`).
    - **User:** O seu endereço de e-mail completo.
    - **Password:** A sua senha. **(Veja a dica importante abaixo para usuários do Gmail!)**
    - **SSL/TLS:** Mantenha ativado para uma conexão segura.
    - **Allow Self-Signed Certificates** Mantenha desativado
5.  Clique em **"Save"**.

> ### ⚠️ DICA IMPORTANTE (Para usuários do Gmail)
>
> Se você usa uma conta do Gmail, **não é possível usar sua senha de login normal**. Devido às políticas de segurança do Google, você precisa gerar uma **"Senha de App"** (App Password).
>
> **Como gerar uma Senha de App:**
>
> 1.  Acesse a página de segurança da sua Conta do Google: [myaccount.google.com/security](https://myaccount.google.com/security)
> 2.  Verifique se a **"Verificação em duas etapas"** está ativada. Ela é obrigatória para gerar senhas de app.
> 3.  Na mesma página, procure pela seção **"Senhas de app"**.
> 4.  Clique em "Selecionar app" e escolha "Outro (_nome personalizado_)". Dê o nome "n8n" e clique em "Gerar".
> 5.  O Google exibirá uma senha de 16 caracteres. **Copie esta senha e cole-a no campo "Password" da sua credencial IMAP no n8n.**

---

## 2. Credencial do Google Drive (OAuth2)

Esta credencial autoriza o n8n a gerenciar arquivos e pastas no seu Google Drive em seu nome, usando o protocolo seguro OAuth2.

- **Nós que utilizam esta credencial:** `3. Criar Pasta no Drive` e `6a. Salvar Arquivo no Drive`

---

### Cenário 1: Conexão Rápida (n8n.cloud ou pré-configurado)

Se você está usando o n8n.cloud ou uma instância já configurada pelo administrador, o processo é simples:

1.  Vá até um dos nós do Google Drive (ex: `3. Criar Pasta no Drive`).
2.  No campo **Credentials**, selecione **"Create New"**.
3.  Dê um nome para a credencial (ex: `Google Drive - Onfly`).
4.  Clique no botão **"Sign in with Google"**.
5.  Uma nova janela do navegador será aberta. Faça login com a conta Google desejada.
6.  Conceda as permissões solicitadas pelo n8n para gerenciar arquivos.
7.  Após a autorização, a janela será fechada, e a credencial estará conectada. Clique em **"Save"**.

---

### Cenário 2: Configuração Completa (Primeira vez ou Self-Hosted)

Se o botão "Sign in with Google" não funcionar ou se você estiver configurando uma instância n8n própria (self-hosted), você precisará criar suas próprias credenciais no Google Cloud. Siga os passos abaixo.

#### **Etapa A: Preparação no Google Cloud**

1.  **Acesse o Google Cloud Console:** Faça login em [console.cloud.google.com](https://console.cloud.google.com/).
2.  **Crie um Novo Projeto:** No menu superior, selecione o dropdown de projetos e clique em **"Novo projeto"**. Dê um nome a ele (ex: "Automacao-n8n") e clique em **"Criar"**.
3.  **Ative a API do Google Drive:**
    - No menu de busca, procure por **"Google Drive API"**.
    - Selecione-a e clique em **"Ativar"**.

#### **Etapa B: Configurar a Tela de Consentimento**

Esta é a tela que os usuários verão ao autorizar a conexão.

1.  No menu lateral, vá para **"APIs e serviços" > "Tela de consentimento OAuth"**.
2.  Selecione o tipo de usuário **"Externo"** e clique em **"Criar"**.
3.  Preencha os campos obrigatórios:
    - **Nome do app:** `Automação n8n Onfly`
    - **E-mail de suporte do usuário:** Seu e-mail.
    - **Informações de contato do desenvolvedor:** Seu e-mail.
4.  Clique em **"Salvar e continuar"** nas próximas seções até chegar ao final. Não é preciso preencher os outros campos para este caso de uso.

#### **Etapa C: Criar as Credenciais (Client ID e Secret)**

Aqui é onde você obterá as chaves para colocar no n8n.

1.  No menu lateral, vá para **"APIs e serviços" > "Credenciais"**.
2.  Clique em **"+ CRIAR CREDENCIAIS"** e selecione **"ID do cliente OAuth"**.
3.  Configure os seguintes campos:
    - **Tipo de aplicativo:** `Aplicativo da Web`.
    - **Nome:** Dê um nome (ex: `Credencial n8n Web`).
    - **Em "URIs de redirecionamento autorizados",** clique em **"+ ADICIONAR URI"**.
      - **Agora, volte para o n8n:** Na tela de criação de credencial do Google Drive, você verá um campo chamado **"OAuth Redirect URL"**. Copie essa URL.
      - **Cole a URL** que você copiou do n8n no campo do Google Cloud.
4.  Clique em **"Criar"**. Uma janela aparecerá com seu **"ID do cliente"** e sua **"Chave secreta do cliente"**. Mantenha esta janela aberta.

#### **Etapa D: Finalizar a Configuração no n8n**

1.  Volte para a tela de criação de credenciais no n8n.
2.  **Copie o "ID do cliente"** do Google Cloud e cole no campo **"Client ID"** do n8n.
3.  **Copie a "Chave secreta do cliente"** do Google Cloud e cole no campo **"Client Secret"** do n8n.
4.  Agora sim, clique em **"Sign in with Google"** e finalize a autorização.
5.  Clique em **"Save"**.

---

## 3. Credencial de Envio de E-mail (SMTP)

Esta credencial permite que o n8n envie e-mails através do seu provedor de e-mail.

- **Nós que utilizam esta credencial:** `8a. Enviar E-mail de Sucesso`, `6b. Enviar E-mail de Aviso` e `Envia notificação de erro`.

### Passo a Passo para Configuração

1.  Vá até um dos nós de envio de e-mail (ex: `8a. Enviar E-mail de Sucesso`).
2.  Clique no campo **Credentials**.
3.  Selecione **"Create New"**.
4.  No campo **"Node Type"**, procure e selecione **"SMTP"**.
5.  Preencha os seguintes campos:
    - **Name:** Dê um nome de fácil identificação (ex: `SMTP - financeiro@onfly.com`).
    - **Host:** O endereço do servidor SMTP do seu provedor (ex: `smtp.gmail.com`).
    - **Port:** A porta do servidor SMTP (ex: `465` ou `587`).
    - **User:** O seu endereço de e-mail completo.
    - **Password:** A sua senha ou Senha de App. **(A mesma regra do IMAP se aplica ao Gmail!)**
    - **SSL/TLS:** Ative se usar a porta `465`. Desative se usar a porta `587` (que geralmente usa STARTTLS).
    - **Sender Name/Email:** Você pode definir um nome e e-mail padrão aqui, que serão usados caso não sejam especificados no nó.
6.  Clique em **"Save"**.

> ### ⚠️ DICA IMPORTANTE (Para usuários do Gmail)
>
> Assim como no IMAP, para usar o SMTP do Gmail você **precisa de uma Senha de App**. O processo para gerá-la é exatamente o mesmo descrito na Seção 1. Você pode usar a mesma senha de app para ambas as credenciais ou gerar uma nova para o SMTP.

Com estas três credenciais configuradas, seu workflow estará pronto para ser ativado e executado corretamente.
