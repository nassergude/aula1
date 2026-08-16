# 5. Arquitetura Proposta

A aplicação **QuimiPort** será organizada utilizando uma arquitetura em camadas, com separação de responsabilidades entre **Apresentação, Aplicação/Serviços, Domínio, Infraestrutura, Auditoria e Permissões**.

A arquitetura foi definida considerando as regras de negócio dos domínios de **Produtos Químicos, Cargas Químicas e Movimentações**, permitindo que as regras relacionadas à operação portuária permaneçam organizadas e independentes dos detalhes de tecnologia.

O objetivo é proporcionar **baixo acoplamento, facilidade de manutenção, testabilidade e possibilidade de evolução da aplicação nas próximas fases**.

---

## 5.1 Visão geral da arquitetura

```text
┌──────────────────────────────────────────────────────┐
│                   APRESENTAÇÃO                       │
│                                                      │
│          Frontend / API / Controllers                │
└──────────────────────────┬───────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────┐
│                    APLICAÇÃO                          │
│                                                      │
│                Casos de Uso / Serviços               │
└──────────────────────────┬───────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────┐
│                     DOMÍNIO                           │
│                                                      │
│  Produtos Químicos │ Cargas Químicas │ Movimentações│
│                                                      │
│  Entidades │ Regras │ Status │ Transições │ Valores │
└──────────────────────────┬───────────────────────────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
┌──────────────────┐ ┌──────────────┐ ┌─────────────────┐
│   AUDITORIA      │ │  PERMISSÃO   │ │ INFRAESTRUTURA  │
│                  │ │              │ │                 │
│ Histórico        │ │ Perfis       │ │ Banco de dados  │
│ Usuários         │ │ Acessos      │ │ Repositórios    │
│ Data/Hora        │ │ Autorizações │ │ Integrações     │
└──────────────────┘ └──────────────┘ └─────────────────┘
```

A camada de **domínio** concentra as regras relacionadas ao comportamento das entidades. A camada de **aplicação/serviço** coordena os casos de uso e as operações do sistema. Já as camadas de **auditoria, permissão e infraestrutura** fornecem responsabilidades complementares necessárias ao funcionamento da aplicação.

---

## 5.2 Camada de Apresentação

A camada de apresentação será responsável pela comunicação entre os usuários e a aplicação.

Ela poderá ser implementada futuramente por meio de uma aplicação web e de uma API.

### Responsabilidades

- receber requisições dos usuários;
- apresentar informações das cargas;
- apresentar informações dos produtos químicos;
- permitir operações autorizadas;
- enviar dados para os casos de uso;
- apresentar mensagens de sucesso ou erro.

A camada de apresentação **não deverá conter regras de negócio**.

Por exemplo, a interface não deverá decidir se uma carga pode passar de `EM_INSPECAO` para `LIBERADA`. Essa decisão pertence às regras do domínio de Movimentações.

---

## 5.3 Camada de Aplicação / Serviços

A camada de aplicação será responsável por **orquestrar os casos de uso do QuimiPort**.

Ela receberá as solicitações da camada de apresentação e coordenará as operações necessárias no domínio.

### Exemplos de casos de uso

- Cadastrar Produto Químico;
- Editar Produto Químico;
- Excluir Produto Químico;
- Criar Carga Química;
- Alterar Carga Química;
- Alterar status da carga;
- Registrar movimentação;
- Consultar histórico de movimentações;
- Cancelar carga;
- Validar documentação;
- Liberar carga;
- Bloquear carga.

A aplicação deverá utilizar as regras definidas no domínio em vez de duplicá-las nos controllers.

---

## 5.4 Camada de Domínio

A camada de domínio representa o **núcleo do QuimiPort**.

Ela será responsável pelos conceitos e comportamentos específicos do negócio de gerenciamento de cargas químicas.

Os três principais domínios definidos pela equipe são:

- **Produtos Químicos**;
- **Cargas Químicas**;
- **Movimentações**.

### 5.4.1 Domínio de Produtos Químicos

A entidade `ProdutoQuimico` possuirá informações como:

- Nome;
- Descrição;
- Status;
- Classe de Risco;
- Data de criação;
- Data de alteração;
- Usuário de criação;
- Usuário de alteração.

### Regras principais

O domínio deverá garantir que:

- o nome do produto químico seja único;
- o produto possua uma classe de risco válida;
- produtos associados a cargas não possam ser excluídos;
- ativação ou inativação do produto não altere cargas já existentes.

### Classificação de risco

A classe de risco deverá seguir a classificação definida no projeto:

```text
1 - Explosivos
2 - Gases
3 - Líquidos inflamáveis
4 - Sólidos inflamáveis
5 - Substâncias oxidantes e peróxidos orgânicos
6 - Substâncias tóxicas e infecciosas
7 - Materiais radioativos
8 - Substâncias corrosivas
9 - Substâncias e artigos perigosos diversos
```

### 5.4.2 Domínio de Cargas Químicas

A entidade `CargaQuimica` representa uma carga que será movimentada no contexto portuário.

### Informações principais

- Produto químico;
- Quantidade;
- Unidade de medida;
- Descrição;
- Destino;
- Técnico Responsável;
- Inspetor Responsável;
- Status;
- Previsão de saída;
- Previsão de entrega;
- Data de criação;
- Data de alteração;
- Usuário de criação;
- Usuário de alteração.

### Regras principais

Na criação da carga devem ser obrigatoriamente informados:

- produto químico;
- quantidade;
- unidade de medida;
- destino.

Além disso:

- não é permitido associar uma carga a um produto químico inativo;
- se o produto for inativado depois da associação, a carga pode continuar normalmente;
- o produto associado somente poderá ser alterado enquanto a carga estiver `CADASTRADA`;
- a classificação de risco da carga será herdada do produto químico;
- a quantidade deverá ser maior que zero;
- o técnico responsável não será informado na criação ou edição da carga;
- o inspetor responsável também não será informado na criação ou edição da carga.

### 5.4.3 Domínio de Movimentações

O domínio de Movimentações será responsável pelo **ciclo de vida da carga**.

Toda alteração de status deverá gerar uma movimentação, e o histórico deverá permanecer disponível para consulta.

Cada movimentação deverá registrar:

- `IdCarga`;
- `Step`;
- Status Anterior;
- Status Atual;
- Notes;
- Data da movimentação;
- Usuário da movimentação.

### Fluxo de status

```text
CADASTRADA
     │
     ▼
EM_PREPARACAO
     │
     ▼
EM_INSPECAO
   /      \
  ▼        ▼
BLOQUEADA  LIBERADA
  │           │
  │           ▼
  └──────► EM_MOVIMENTACAO
               │
               ▼
           FINALIZADA
```

As transições válidas são:

- `CADASTRADA → EM_PREPARACAO`;
- `EM_PREPARACAO → EM_INSPECAO`;
- `EM_INSPECAO → LIBERADA`;
- `EM_INSPECAO → BLOQUEADA`;
- `LIBERADA → EM_MOVIMENTACAO`;
- `EM_MOVIMENTACAO → FINALIZADA`;
- `BLOQUEADA → EM_PREPARACAO`.

O cancelamento funciona como uma exceção ao fluxo principal: uma carga pode ser cancelada a partir de qualquer status, exceto `FINALIZADA`, e não poderá possuir novas movimentações depois de `CANCELADA`.

---

## 5.5 Regras de negócio concentradas no domínio

A arquitetura deverá impedir que as regras sejam espalhadas pela aplicação.

### Exemplo 1

**Regra:** Toda alteração de status de uma carga deve gerar uma movimentação.

**Responsabilidade:** domínio de **Movimentações**.

### Exemplo 2

**Regra:** A quantidade da carga deve ser maior que zero.

**Responsabilidade:** domínio de **Cargas Químicas**.

### Exemplo 3

**Regra:** O nome do produto químico deve ser único.

**Responsabilidade:** domínio de **Produtos Químicos**.

Dessa forma, cada regra fica próxima da entidade ou serviço responsável pelo comportamento correspondente.

---

## 5.6 Camada de Auditoria

A auditoria será responsável por manter a **rastreabilidade das operações realizadas no sistema**.

### Produtos Químicos

Deverão ser registrados:

- data/hora de criação;
- data/hora de alteração;
- usuário responsável pela criação;
- usuário responsável pela alteração.

### Cargas Químicas

Deverão ser registrados:

- data/hora de criação;
- data/hora de alteração;
- usuário responsável pela criação;
- usuário responsável pela alteração.

### Movimentações

Deverão ser registrados:

- data e hora;
- usuário responsável;
- status anterior;
- status atual;
- step da movimentação.

A auditoria será independente das regras principais do domínio, mas será acionada pelas operações realizadas pela aplicação.

---

## 5.7 Camada de Permissões

A camada de permissão será responsável por verificar **quais operações cada perfil pode realizar**.

### Administrador do sistema

Possui acesso a todas as etapas da carga e pode cancelar uma carga.

### Gestor operacional

Possui acesso a todas as etapas da carga e pode cancelar uma carga.

### Operador portuário

Possui acesso durante:

- `LIBERADA`;
- `EM_MOVIMENTACAO`.

### Responsável técnico

Possui acesso durante:

- `EM_PREPARACAO`;
- `BLOQUEADA`.

### Analista de documentação

Possui acesso durante:

- `EM_PREPARACAO`.

### Analista de qualidade

Possui acesso durante:

- `EM_INSPECAO`.

---

## 5.8 Camada de Infraestrutura

A infraestrutura será responsável pelos recursos tecnológicos necessários para executar a aplicação.

### Responsabilidades

- acesso ao banco de dados;
- implementação dos repositórios;
- persistência de Produtos Químicos;
- persistência de Cargas Químicas;
- persistência de Movimentações;
- armazenamento do histórico;
- integrações externas;
- serviços de autenticação;
- logs técnicos.

A camada de infraestrutura não deverá conter as regras de negócio principais.

Por exemplo, o banco poderá armazenar uma carga, mas a decisão sobre **se uma carga pode mudar de `EM_INSPECAO` para `LIBERADA`** deverá ser determinada pelas regras do domínio.

---

## 5.9 Fluxo de uma operação

Como exemplo, considere a alteração do status de uma carga.

```text
Usuário
   │
   ▼
Apresentação
   │
   ▼
Caso de Uso
"Alterar Status da Carga"
   │
   ▼
Verificação de Permissão
   │
   ▼
Domínio da Carga/Movimentação
   │
   ├── Verifica status atual
   ├── Verifica transição permitida
   ├── Executa alteração
   └── Gera movimentação
   │
   ▼
Auditoria
   │
   ▼
Infraestrutura
   │
   ▼
Banco de Dados
```

Nesse fluxo, a interface não executa diretamente a alteração. Ela solicita a operação, enquanto a aplicação coordena o processo e o domínio garante que as regras sejam respeitadas.

---

## 5.10 Separação de responsabilidades

| Camada | Responsabilidade |
|---|---|
| **Apresentação** | Interface e comunicação com o usuário |
| **Aplicação** | Orquestração dos casos de uso |
| **Domínio** | Entidades e regras de negócio |
| **Auditoria** | Registro e rastreabilidade das operações |
| **Permissão** | Controle de acesso dos usuários |
| **Infraestrutura** | Banco de dados, repositórios e integrações |

Essa divisão está alinhada às categorias utilizadas nas regras de negócio: **Entidade, Serviço, Auditoria e Permissão**.

---

## 5.11 Justificativa da arquitetura

A arquitetura proposta foi escolhida para manter as regras de negócio do QuimiPort organizadas de acordo com seus respectivos domínios.

A separação entre **Produtos Químicos, Cargas Químicas e Movimentações** permite que cada conjunto de regras seja mantido de forma coesa.

Além disso, a separação entre domínio, aplicação e infraestrutura evita que regras como:

- produto químico deve possuir classe de risco válida;
- quantidade da carga deve ser maior que zero;
- produto inativo não pode ser utilizado em novas cargas;
- alterações de status devem gerar movimentações;
- transições de status devem respeitar o fluxo definido;
- cargas finalizadas ou canceladas não podem possuir novas movimentações;

fiquem dependentes de uma interface, banco de dados ou framework específico.

Dessa forma, a arquitetura fornece uma base para que o QuimiPort possa evoluir nas próximas fases sem precisar alterar o núcleo das regras de negócio.

---

## 5.12 Conclusão

A arquitetura proposta para o **QuimiPort** utiliza uma organização em camadas, separando **Apresentação, Aplicação, Domínio, Auditoria, Permissões e Infraestrutura**.

O **Domínio** concentra as regras específicas de Produtos Químicos, Cargas Químicas e Movimentações. A **Aplicação** coordena os casos de uso, enquanto a **Auditoria** garante rastreabilidade, a **Permissão** controla os acessos e a **Infraestrutura** fornece os recursos tecnológicos necessários.

Essa organização permite que as regras já definidas pela equipe sejam mantidas de forma centralizada e consistente, proporcionando uma arquitetura preparada para implementação, testes e evolução do QuimiPort nas próximas fases do projeto.
