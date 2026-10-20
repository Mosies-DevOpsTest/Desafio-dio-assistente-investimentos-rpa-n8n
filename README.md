# 🤖 Assistente de Investimentos com RPA e IA Generativa

Projeto desenvolvido para o desafio da [DIO](https://dio.me), com o
objetivo de demonstrar um fluxo de automação que coleta dados de
clientes, cruza essas informações com uma base de produtos financeiros e
utiliza IA generativa para preparar mensagens personalizadas, enviadas
por e-mail.

A solução integra **Python**, **n8n**, **JavaScript**, **Google Gemini**
e **Gmail**.

> **Aviso importante:** este projeto tem finalidade educacional e
> demonstrativa. As mensagens geradas por IA não constituem recomendação
> financeira validada. Rentabilidades, condições dos produtos e
> adequação ao perfil do investidor precisam ser verificadas em fontes
> oficiais antes de qualquer uso real. Não envie dados pessoais ou
> financeiros reais sem implementar os controles de segurança,
> privacidade e consentimento necessários.

## 📌 Índice

-   [Visão geral](#-visão-geral)
-   [Arquitetura da solução](#️-arquitetura-da-solução)
-   [Tecnologias e ferramentas](#️-tecnologias-e-ferramentas)
-   [Etapas do pipeline](#️-etapas-do-pipeline)
-   [Estrutura esperada dos dados](#-estrutura-esperada-dos-dados)
-   [Pré-requisitos](#-pré-requisitos)
-   [Como executar](#-como-executar)
-   [Exemplo ilustrativo de saída](#-exemplo-ilustrativo-de-saída)
-   [Validações e pontos de atenção](#-validações-e-pontos-de-atenção)
-   [Limitações conhecidas](#-limitações-conhecidas)

## 📌 Visão geral

O projeto simula um cenário em que dados de investidores são extraídos
de uma página web de demonstração, enviados a um webhook do n8n,
processados e relacionados a produtos financeiros de uma base CSV. Em
seguida, o Google Gemini gera um texto contextualizado e o Gmail realiza
o envio.

O fluxo foi organizado para demonstrar:

-   Extração de dados com Python e BeautifulSoup.
-   Comunicação HTTP com o webhook do n8n.
-   Leitura e normalização de dados tabulares.
-   Cruzamento entre perfil de investidor e produtos disponíveis.
-   Preparação de contexto para um modelo de linguagem (LLM).
-   Normalização dos destinatários e envio de e-mails.

## 🏗️ Arquitetura da solução

``` mermaid
flowchart LR
    A["Página web de demonstração<br/>HTML / GitHub Pages"] -->|Web scraping| B["RPA em Python<br/>BeautifulSoup e Requests"]
    B -->|POST com JSON| C["Webhook do n8n"]

    subgraph N8N["Orquestração no n8n"]
        C --> D["Leitura e normalização<br/>da base CSV"]
        D --> E["Merge e cruzamento<br/>por perfil"]
        E --> F["Preparação do prompt<br/>JavaScript"]
        F --> G["Google Gemini<br/>Geração de texto"]
        G --> H["Normalização do e-mail<br/>JavaScript"]
        H --> I["Gmail<br/>Envio da mensagem"]
    end
```

## 🛠️ Tecnologias e ferramentas

  -----------------------------------------------------------------------
  Tecnologia                          Função no projeto
  ----------------------------------- -----------------------------------
  **Python**                          Extração de dados (*web scraping*)
                                      com BeautifulSoup e envio de
                                      requisições HTTP com Requests.

  **n8n**                             Orquestração do fluxo, recebimento
                                      por webhook, transformação de dados
                                      e integração entre serviços.

  **JavaScript (ES6)**                Limpeza, transformação e
                                      normalização dos dados nos nós
                                      Code.

  **Google Gemini (LLM)**             Geração de mensagens
                                      contextualizadas a partir dos dados
                                      fornecidos ao modelo.

  **Gmail**                           Envio das mensagens por e-mail.

  **CSV**                             Base tabular de produtos
                                      financeiros usada no cruzamento.
  -----------------------------------------------------------------------

## ⚙️ Etapas do pipeline

### 1. Extração de dados com RPA (Python)

O script `extrair_clientes.py` acessa a página web de demonstração,
extrai campos como nome, e-mail, saldo e perfil e envia os dados em
formato JSON para o webhook do n8n.

**Atenção:** a estrutura HTML da página de origem precisa corresponder
aos seletores utilizados no script. Mudanças no site podem interromper a
extração.

### 2. Carga e normalização da base de investimentos

O workflow lê a base `data.csv` e normaliza os campos necessários,
especialmente os valores mínimos de aplicação. A conversão deve
considerar o formato usado no arquivo (por exemplo, separador decimal e
símbolo de moeda).

### 3. Cruzamento de perfis (Merge)

Os dados dos clientes são relacionados aos produtos associados aos
perfis previstos no projeto: **Conservador**, **Moderado** e
**Arrojado**.

O cruzamento precisa usar valores padronizados. Diferenças de grafia,
espaços ou acentuação podem impedir correspondências. Também é
necessário verificar se o saldo do cliente atende ao valor mínimo de
aplicação de cada produto.

### 4. Preparação do prompt e geração de texto (JavaScript + Gemini)

Um nó Code prepara as instruções e os dados enviados ao Google Gemini. O
modelo gera uma mensagem explicativa com base no contexto recebido.

A saída do modelo deve ser tratada como texto gerado por IA, não como
validação independente da rentabilidade, liquidez, risco ou adequação
dos produtos. Não inclua no prompt dados pessoais além dos estritamente
necessários.

### 5. Normalização e envio por Gmail

Antes do envio, o workflow pode remover espaços desnecessários e
normalizar o endereço de e-mail para minúsculas. O assunto pode ser
construído dinamicamente.

Antes de ativar o envio, teste com destinatários controlados e confirme
o tratamento de falhas, duplicidades e respostas inválidas do modelo.

## 🧾 Estrutura esperada dos dados

O documento original informa que o projeto utiliza `data.csv`, mas não
apresenta sua estrutura completa. Portanto, confirme os nomes das
colunas no arquivo real e nos nós do workflow antes de executar.

Se a base seguir o formato ilustrativo abaixo, ela poderá conter estas
colunas:

``` csv
perfil,produto,minimo,rentabilidade
Conservador,Poupança,100,6.2%
Conservador,CDB Liquidez Diária,1000,12.5%
Conservador,Tesouro Selic,500,13.0%
```

**Os valores acima são apenas exemplos ilustrativos**, não cotações
atuais nem promessa de rentabilidade. O workflow e o arquivo CSV
precisam usar os mesmos nomes de campos e formatos. Defina também o
período de referência da rentabilidade (por exemplo, ao ano) e sua
fonte; sem isso, o valor percentual pode ser interpretado
incorretamente.

## ✅ Pré-requisitos

-   Python 3.9 ou superior.
-   Instância do n8n Cloud ou instalação local (por exemplo, via Docker
    ou npm).
-   Chave de API do Google Gemini, obtida no [Google AI
    Studio](https://aistudio.google.com/).
-   Conta e credenciais do Gmail configuradas no n8n.
-   Arquivos do projeto, incluindo `extrair_clientes.py`, `data.csv` e
    `workflow.json`, caso estejam disponíveis no repositório.

## 🚀 Como executar

### 1. Clonar o repositório

Substitua `SEU-USUARIO` pelo nome correto da conta ou organização do
GitHub e confirme o endereço real do repositório.

``` bash
git clone https://github.com/SEU-USUARIO/dio-lab-assistente-investimentos-rpa-n8n.git
cd dio-lab-assistente-investimentos-rpa-n8n
```

O endereço acima contém um marcador de posição e **não deve ser
considerado um link confirmado**.

### 2. Instalar as dependências Python

No terminal, a partir da pasta do projeto:

``` bash
python -m pip install requests beautifulsoup4
```

Se o projeto fornecer um arquivo `requirements.txt`, prefira instalar as
dependências declaradas nele:

``` bash
python -m pip install -r requirements.txt
```

### 3. Conferir a estrutura dos arquivos

A documentação original indica um diretório `rpa` e um script chamado
`extrair_clientes.py`. Se o script estiver nesse diretório, acesse-o
antes de executar:

``` bash
cd rpa
python extrair_clientes.py
```

Se a estrutura real do repositório for diferente, ajuste o caminho
conforme os arquivos existentes. O documento de origem menciona execução
de notebook ou script, mas só identifica explicitamente
`extrair_clientes.py`; este README documenta a execução do script.

### 4. Importar o workflow no n8n

1.  Abra o n8n e crie ou importe o workflow.
2.  Use a opção **Import from File** e selecione `workflow.json`, se
    esse arquivo estiver disponível.
3.  Configure as credenciais exigidas pelos nós do Google Gemini e do
    Gmail.
4.  Confira a URL e o método HTTP do webhook e atualize a configuração
    do script Python para apontar para a URL correta.
5.  Verifique os caminhos dos arquivos e o mapeamento das colunas de
    `data.csv`.
6.  Execute um teste manual com dados fictícios antes de habilitar o
    fluxo para uso recorrente.

> A importação de um workflow não configura automaticamente todas as
> credenciais, URLs, permissões ou caminhos locais. Revise cada nó antes
> de executar.

## 📄 Exemplo ilustrativo de saída

A mensagem abaixo demonstra o formato possível. Nome, saldo, produtos e
percentuais são fictícios e não representam uma recomendação financeira
real.

``` text
Olá, Ana Silva!

Com base nos dados de demonstração recebidos, identificamos o perfil
Conservador e um saldo informado de R$ 12.500,00.

O fluxo encontrou as seguintes opções na base de exemplo:

- Poupança (aplicação mínima: R$ 100; rentabilidade informada: 6,2%)
- CDB Liquidez Diária (aplicação mínima: R$ 1.000; rentabilidade informada: 12,5%)
- Tesouro Selic (aplicação mínima: R$ 500; rentabilidade informada: 13,0%)

Os valores de rentabilidade precisam ser verificados quanto à fonte,
data de referência, período, impostos, taxas, liquidez e riscos.
Esta mensagem é uma demonstração automatizada, não uma recomendação
financeira validada.
```

## 🔎 Validações e pontos de atenção

-   **Dados de entrada:** validar campos obrigatórios, tipos, valores
    ausentes e formato do e-mail.
-   **Web scraping:** confirmar que a coleta respeita os termos de uso
    do site e as regras aplicáveis de privacidade.
-   **CSV:** validar codificação, delimitador, separador decimal, campos
    obrigatórios e duplicidades.
-   **Cruzamento:** padronizar os nomes dos perfis e confirmar que cada
    cliente só recebe produtos que passaram pelas regras definidas no
    workflow.
-   **Saldo e valor mínimo:** comparar valores numéricos normalizados,
    sem misturar texto e números.
-   **Rentabilidade:** registrar fonte, data e período; não apresentar
    valores ilustrativos como atuais ou garantidos.
-   **IA generativa:** validar a resposta do modelo, controlar saídas
    inesperadas e não permitir que o modelo invente dados de produtos.
-   **E-mail:** testar destinatários, assunto, duplicidades, falhas de
    entrega e tratamento de erros.
-   **Segurança:** guardar chaves e credenciais no mecanismo de
    credenciais do n8n ou em variáveis de ambiente apropriadas; não
    colocá-las no código, no CSV ou no repositório.
-   **Privacidade:** usar dados fictícios nos testes e avaliar as
    obrigações aplicáveis à proteção de dados pessoais antes de
    processar dados reais.

## ⚠️ Limitações conhecidas

-   A execução depende da estrutura da página web usada para a extração.
-   Os nomes de campos e o formato de `data.csv` precisam corresponder
    aos esperados pelo workflow.
-   O repositório e os arquivos `workflow.json`, `data.csv` e
    `extrair_clientes.py` não foram verificados de forma independente
    nesta documentação; confirme se todos estão presentes e atualizados.
-   A documentação original não especifica a fonte nem o período dos
    percentuais de rentabilidade. Esses dados devem ser definidos antes
    de qualquer uso além da demonstração.
-   O projeto demonstra integração técnica, mas não comprova, por si só,
    a adequação financeira das opções selecionadas.

------------------------------------------------------------------------

Projeto educacional para demonstrar integração entre RPA, automação de
workflows e IA generativa. Antes de usar em produção, implemente
validações, monitoramento, tratamento de erros, controles de segurança e
revisão das mensagens geradas.
