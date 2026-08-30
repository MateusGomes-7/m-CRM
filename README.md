# m-CRM — Central Inteligente de Cadastro e Enriquecimento de Clientes

## 📌 Descrição da Solução

O **m-CRM** é uma aplicação web que resolve um problema comum em ambientes corporativos: o cadastro manual e demorado de dados de clientes empresariais. Em vez de o usuário precisar preencher razão social, situação cadastral, atividade econômica e endereço completo à mão, o sistema faz isso automaticamente.

O usuário informa apenas **CNPJ** e **CEP** de um cliente novo. A partir desses dois dados, uma automação consulta APIs públicas oficiais, busca as informações da empresa e do endereço, trata e organiza esses dados, e os grava de volta no cadastro — sem qualquer intervenção manual adicional.

O resultado é uma central de cadastro rápida, confiável (dados vindos de fontes oficiais) e organizada, reduzindo erros de digitação e tempo operacional do time comercial/administrativo.

### Fluxo resumido para o usuário
1. Usuário acessa o m-CRM e faz login.
2. Na tela "Novo Cliente", informa CNPJ e CEP.
3. O sistema salva o registro inicial.
4. Em poucos minutos, o cadastro é enriquecido automaticamente com Razão Social, Situação Cadastral, Atividade Principal e Endereço completo.
5. O usuário consulta, busca e gerencia todos os clientes na tela "Clientes".

---

## 🔌 APIs Utilizadas

### 1. BrasilAPI — Dados de CNPJ
- **Endpoint:** `https://brasilapi.com.br/api/cnpj/v1/{cnpj}`
- **Autenticação:** nenhuma (API pública e gratuita)
- **Dados obtidos:** razão social, situação cadastral, atividade econômica principal (CNAE), entre outros.
- **Motivo da escolha:** API brasileira mantida pela comunidade, com dados oficiais da Receita Federal, gratuita, sem necessidade de chave/token, o que simplifica a integração sem abrir mão de confiabilidade.

### 2. ViaCEP — Dados de Endereço
- **Endpoint:** `https://viacep.com.br/ws/{cep}/json/`
- **Autenticação:** nenhuma (API pública e gratuita)
- **Dados obtidos:** logradouro, bairro, localidade (cidade), UF.
- **Motivo da escolha:** referência de mercado para consulta de CEP no Brasil, estável, gratuita e sem burocracia de cadastro/autenticação.

---

## 🔄 Fluxo de Integração entre os Sistemas

```
┌──────────────┐       ┌─────────────────┐       ┌──────────────────┐
│   Lovable    │──────▶│    Airtable      │◀─────▶│       Make        │
│ (Interface)  │ grava │  (Banco no-code) │ lê/    │  (Automação/ETL)  │
│              │  CNPJ │                  │escreve │                    │
│              │  e CEP│                  │        │                    │
└──────────────┘       └─────────────────┘       └─────────┬─────────┘
                                                              │
                                              ┌───────────────┼───────────────┐
                                              ▼                                ▼
                                      ┌───────────────┐               ┌───────────────┐
                                      │   BrasilAPI    │               │    ViaCEP      │
                                      │ (dados CNPJ)   │               │ (dados CEP)    │
                                      └───────────────┘               └───────────────┘
```

**Passo a passo técnico:**
1. O usuário cadastra CNPJ e CEP pela interface (Lovable), que grava um novo registro na tabela `Dados` do Airtable.
2. O Make monitora a tabela periodicamente (gatilho **Watch Records**, a cada 15 minutos) em busca de registros novos.
3. Para cada registro novo, o Make faz uma chamada HTTP GET à **BrasilAPI**, usando o CNPJ do registro.
4. Em paralelo, faz uma chamada HTTP GET ao **ViaCEP**, usando o CEP do registro.
5. Os dados retornados (JSON) são tratados e mapeados para os campos correspondentes.
6. O Make atualiza o mesmo registro no Airtable (**Update a Record**), preenchendo Razão Social, Situação Cadastral, Atividade Principal e Endereço completo.
7. A interface (Lovable) consulta o Airtable e exibe a lista de clientes já atualizada.

---

## 🔐 Autenticação e Segurança

- **Acesso à aplicação:** protegido por tela de login (e-mail/senha), com autenticação gerenciada pelo backend do Lovable.
- **Acesso ao Airtable:** feito via **Personal Access Token**, com escopo restrito apenas à base do projeto (leitura e escrita de registros), evitando exposição de outras bases ou dados do workspace.
- **Chamadas às APIs externas (BrasilAPI e ViaCEP):** não exigem chave, mas o tráfego ocorre via HTTPS, garantindo criptografia em trânsito.
- **Boas práticas aplicadas:** nenhuma credencial (token, senha) é exposta na interface do usuário ou em código-fonte público; a comunicação entre os sistemas é feita apenas pelos serviços de automação (Make), fora da camada visível ao usuário final.

---

## 🗄️ Armazenamento e Manipulação de Dados

- **Banco de dados no-code:** Airtable, tabela `Dados`, com os campos: CNPJ, CEP, Razão Social, Situação Cadastral, Atividade Principal, Endereço, Data de Criação (campo automático do tipo *Created time*, usado como gatilho da automação).
- **Tratamento de dados:** os dados brutos retornados pelas APIs (JSON) são convertidos e concatenados (no caso do endereço, unindo logradouro, bairro, cidade e UF em um único campo de texto legível) antes de serem persistidos.
- **Consistência:** o CNPJ é normalizado (remoção de pontuação) antes de ser usado nas chamadas às APIs, evitando falhas por formatação divergente entre o que o usuário digita e o que as APIs externas esperam.

---

## 🖥️ Interface

Desenvolvida no **Lovable**, com identidade visual minimalista (paleta verde-menta), contendo:
- Tela de login.
- Tela "Novo Cliente" (cadastro simplificado via CNPJ/CEP).
- Tela "Clientes" (listagem com busca, badges de situação cadastral e visualização dos dados enriquecidos).

---

## 📸 Prints da Aplicação

Os prints das telas (Login, Novo Cliente, Clientes), do cenário de automação no Make e da tabela enriquecida no Airtable estão disponíveis na pasta do Google Drive abaixo:

🔗 [Prints da aplicação — Google Drive](https://drive.google.com/drive/folders/1AytZ6GitsGm63x9j4litLdRT1w8Tawl2?usp=sharing)

---

## ▶️ Como Executar / Acessar o Projeto

Este projeto é composto por três serviços integrados, sem necessidade de instalação local:

1. **Interface (Lovable):** acesse o link publicado da aplicação: https://projetomcrm.lovable.app
2. **Banco de dados (Airtable):** os dados podem ser consultados diretamente na base `Mini CRM` (acesso restrito, mediante convite): https://airtable.com/invite/l?inviteId=invAttFLnI0MOFie9&inviteToken=fb7b00a4978ca147fa16b1052df26d79df5cf3f28e0462acf9985621292866b1&utm_medium=email&utm_source=product_team&utm_content=transactional-alerts
3. **Automação (Make):** o cenário de integração roda automaticamente a cada 15 minutos, sem necessidade de ação manual, desde que esteja ativado no painel do Make.

Para reproduzir o projeto do zero:
1. Criar uma base no Airtable com a tabela `Dados` (campos listados na seção de Armazenamento).
2. Gerar um Personal Access Token no Airtable (Builder Hub → Personal Access Tokens) com escopo `data.records:read`, `data.records:write` e `schema.bases:read`, restrito à base criada.
3. Criar um cenário no Make com os módulos: Airtable Watch Records → HTTP (BrasilAPI) → HTTP (ViaCEP) → Airtable Update a Record.
4. Publicar a interface no Lovable, conectando-a à mesma base do Airtable via API REST, usando o Base ID e o Personal Access Token gerados.

---

## 🧭 Tecnologias Utilizadas

| Camada | Ferramenta |
|---|---|
| Interface | Lovable |
| Banco de dados no-code | Airtable |
| Automação / Integração | Make |
| APIs externas | BrasilAPI, ViaCEP |

---

## 👤 Autor

**Mateus Gomes**
Projeto desenvolvido como trabalho acadêmico da disciplina de Integração de APIs — UniFECAF.
