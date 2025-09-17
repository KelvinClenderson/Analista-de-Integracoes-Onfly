# Automação de Relatórios Financeiros

**Autor:** Kelvin Clenderson Viana de Araujo

## 1. Visão Geral e Lógica do Processo

A lógica central do processo é a seguinte:

1.  **Monitorar e Receber:** O fluxo é iniciado por um gatilho que monitora a chegada de novos e-mails.
2.  **Validar:** O sistema primeiro valida se o e-mail recebido contém anexos.
3.  **Organizar:** Caso existam anexos, uma pasta exclusiva para a operação é criada no Google Drive, nomeada com o assunto do e-mail e a data do processamento, garantindo a organização dos arquivos.
4.  **Processar:** O código principal extrai os anexos, prepara-os para o upload e gera uma mensagem de confirmação dinâmica, que se adapta para listar um ou múltiplos arquivos.
5.  **Armazenar:** Os arquivos extraídos são salvos na pasta criada no Google Drive.
6.  **Notificar:** Com base no resultado da validação, um e-mail de confirmação é enviado ao remetente original. Se o processo foi bem-sucedido, o e-mail contém a lista de arquivos processados, um link para a pasta no Drive e a cotação do dólar do dia. Caso contrário, um e-mail de aviso é enviado solicitando uma ação corretiva.
7.  **Monitorar Erros:** Um gatilho de erro independente monitora todo o workflow, enviando uma notificação detalhada para um administrador caso qualquer etapa falhe inesperadamente.

## 2. Fluxo Detalhado (Etapa por Etapa)

Aqui está a descrição de cada nó na ordem de execução.

### `1. Recebe o E-mail` (Nó: IMAP Trigger)

- **Propósito:** É o ponto de partida da automação. Este nó monitora constantemente a caixa de entrada configurada e inicia o fluxo de trabalho sempre que um novo e-mail chega.
- **Configuração Chave:** Utiliza uma credencial IMAP para se conectar ao servidor de e-mail. A opção "Read IMAP" foi escolhida por sua compatibilidade com diversos provedores, não se limitando ao Gmail.
- **Lógica Aplicada:** O nó já baixa todos os anexos do e-mail e os disponibiliza como dados binários para os nós seguintes.

### `2. Verifica se Há Anexo` (Nó: IF)

- **Propósito:** Roteia o fluxo com base na presença de anexos. É a primeira e mais importante validação para evitar que o fluxo prossiga desnecessariamente.
- **Configuração Chave:** A condição `exists` verifica se o objeto `binary` existe no item que saiu do gatilho.
- **Lógica Aplicada:**
  - **Saída `true`**: O e-mail contém anexos e o fluxo segue para o caminho de processamento.
  - **Saída `false`**: O e-mail não contém anexos. O fluxo é desviado para o caminho de aviso ao usuário.

### Caminho de Sucesso (Saída `true` do IF)

#### `3. Criar Pasta no Drive` (Nó: Google Drive)

- **Propósito:** Cria uma nova pasta no Google Drive para armazenar os arquivos daquela execução específica.
- **Configuração Chave:** O nome da pasta é gerado dinamicamente usando o assunto do e-mail original e a data/hora atual, garantindo que cada execução tenha um repositório único e rastreável. Ex: `Relatório Financeiro - 17/09/2025 00:44`.
- **Lógica Aplicada:** O ID desta pasta recém-criada é usado nos passos seguintes para direcionar o upload dos arquivos.

#### `4. Extrair e Preparar Anexos` (Nó: Code)

- **Propósito:** Este é o cérebro da automação. O código JavaScript executa várias tarefas críticas.
- **Lógica Aplicada:**
  1.  **Referência Direta:** O código usa `$('Recebe o E-mail').first()` para "alcançar" e pegar os dados do gatilho original. Isso garante que, mesmo após vários nós, a referência ao e-mail original nunca seja perdida.
  2.  **Extração de Nomes:** Ele itera sobre todos os anexos e cria uma lista com seus nomes (`fileNames`).
  3.  **Geração de Mensagem Dinâmica:** Com base na quantidade de nomes na lista, ele formata uma mensagem em HTML que será usada no e-mail final. A mensagem se adapta para o singular ("O arquivo X foi processado...") ou plural ("Os arquivos: - X - Y Foram processados...").
  4.  **Separação dos Itens:** O código cria um item de saída separado para cada anexo encontrado. Cada item contém o binário do arquivo e toda a informação de contexto (remetente, assunto, mensagem de confirmação, etc.), preparando-os para o upload individual.

#### `5. Salvar Arquivo no Drive` (Nó: Google Drive)

- **Propósito:** Realiza o upload de cada arquivo extraído para o Google Drive.
- **Configuração Chave:**
  - `Input Data Field`: `arquivos_extraidos` (o nome da propriedade binária definida no nó de código).
  - `File Name`: É preenchido dinamicamente com o nome original do arquivo.
  - `Folder ID`: Utiliza a expressão `{{ $('Criar Pasta no Drive').item.json.id }}` para garantir que o arquivo seja salvo na pasta que foi criada no passo 3.

#### `6. Limit1` (Nó: Limit) - Ponto de Melhoria

- **Propósito:** Este nó atualmente serve para unificar o fluxo novamente em um único item, permitindo que os nós seguintes (cotação e e-mail) executem apenas uma vez.

#### `7. Consultar Cotação do Dólar` (Nó: HTTP Request)

- **Propósito:** Faz uma chamada em tempo real para a API pública `open.er-api.com` para obter a cotação do Dólar (USD) para o Real (BRL).
- **Lógica Aplicada:** A chamada é feita no final do fluxo de sucesso para enriquecer o e-mail de confirmação com um dado relevante e atualizado.

#### `8. Enviar E-mail de Sucesso` (Nó: Send Email)

- **Propósito:** Envia o e-mail de confirmação final para o remetente original.
- **Configuração Chave:** O corpo do e-mail é construído em HTML, usando expressões para inserir dinamicamente o nome do remetente, a mensagem com a lista de arquivos, o link para a pasta no Drive e a cotação do dólar.
- **Lógica Aplicada:** Este nó representa a conclusão bem-sucedida do fluxo, fornecendo um feedback completo e útil ao usuário.

### Caminho de Aviso (Saída `false` do IF)

#### `9. Enviar E-mail de Aviso` (Nó: Send Email)

- **Propósito:** Notifica o remetente quando um e-mail é recebido sem anexos.
- **Lógica Aplicada:** O corpo do e-mail é claro e direto, informando o problema (ausência de anexo) e solicitando a ação necessária (reenviar o e-mail com o arquivo).

## 3. Tratamento de Erros

### `Report de bugs` (Nó: Error Trigger)

- **Propósito:** Este gatilho especial não faz parte do fluxo principal. Ele é acionado automaticamente **se qualquer nó do workflow falhar** durante a execução.
- **Lógica Aplicada:** Ao capturar um erro, ele passa os detalhes da falha (nome do workflow, ID da execução, nó que falhou, mensagem de erro) para o nó seguinte.

### `Envia notificação de erro` (Nó: Send Email)

- **Propósito:** Envia um alerta imediato para um administrador ou time de desenvolvimento.
- **Lógica Aplicada:** O e-mail é formatado para conter todas as informações essenciais do erro, incluindo um link direto para a execução que falhou, permitindo uma depuração rápida e eficiente. Esta é uma prática recomendada para a manutenção de automações em produção.
