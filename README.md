### Financial API & Authentication Microservice 

## Resumo do Desafio

Este projeto foi desenvolvido como parte de um desafio de criação de uma API RESTful para gestão de finanças utilizando **NestJS** e **PostgreSQL**. A aplicação gerencia contas financeiras, transações, e relatórios de forma segura, escalável e testável.

O sistema foi dividido em dois microserviços principais:
1. **auth-service**: Responsável pela autenticação e criação de usuários.
2. **finance-api**: Responsável pelas operações financeiras como criação de contas, transações (depósitos, retiradas e transferências), e relatórios financeiros.

### Funcionalidades Implementadas:

- **Gestão de Contas Financeiras**: CRUD de contas (nome, número, saldo, tipo).
- **Gestão de Transações**: Depósitos, retiradas e transferências, com regras de negócio para evitar saldos negativos.
- **Suporte a Múltiplas Moedas e Taxas de Câmbio**: Utilização de uma API externa para buscar taxas de câmbio em tempo real e realizar conversões.
- **Operações Agendadas**: Transações programadas e recorrentes utilizando a ferramenta Bull com Redis para processamento em segundo plano.
- **Segurança**: Autenticação via JWT, diferenciação de papéis de usuário (admin vs. usuário comum) e validação de dados.
- **Arquitetura Orientada a Eventos**: Publicação e consumo de eventos para processos como transferências e atualizações de contas.
- **Relatórios Financeiros**: Extratos de contas e relatórios de movimentação financeira.

## Arquitetura do Sistema

O sistema é baseado em uma **arquitetura de microserviços**, composta por dois serviços principais:

1. **Auth Service**:
   - Responsável pela criação de usuários, autenticação e emissão de tokens JWT.
   - Interface com a API de finanças para permitir o acesso seguro às operações financeiras.

2. **Finance API**:
   - Gerencia operações financeiras como contas bancárias, transações e relatórios.
   - Processa as transações de depósitos, retiradas e transferências, além de oferecer funcionalidades como suporte a múltiplas moedas e agendamento de transações.

### Fluxo de Comunicação entre Microserviços

A comunicação entre os dois microserviços é feita através do token JWT gerado pelo **Auth Service**. O token é enviado nas requisições à **Finance API**, garantindo que apenas usuários autenticados possam realizar operações financeiras.

### Tecnologias Utilizadas

- **NestJS**: Framework utilizado para criar ambos os microserviços.
- **PostgreSQL**: Banco de dados relacional utilizado para armazenar informações de contas e transações.
- **TypeORM**: ORM para integração com o banco de dados.
- **JWT**: Autenticação baseada em token para garantir segurança.
- **Bull + Redis**: Processamento de filas para transações agendadas e taxa de câmbio em tempo real.
- **API externa de Taxas de Câmbio**: Para conversão de moedas.
- **Docker**: Para containerização do ambiente.
- **Jest**: Para testes unitários.

## Como Rodar o Projeto

### 1. Clone os Repositórios

Clone ambos os repositórios (Auth Service e Finance API):

```bash
git clone https://github.com/Lucaswillians/auth-service.git
git clone https://github.com/Lucaswillians/finance-api.git
```

### Testando os Endpoints com o Insomnia
Para facilitar a interação com a API, um arquivo de configuração do Insomnia contendo todos os endpoints está disponível no repositório. Você pode importá-lo para o Insomnia e testar as funcionalidades da API de forma simples.

Instruções para usar o arquivo do Insomnia:
- Faça o download do arquivo financial-api-insomnia.json disponível na raiz do repositório.
- Abra o Insomnia e clique em Importar.
- Selecione o arquivo financial-api-insomnia.json.
- Agora, você pode testar todos os endpoints da API diretamente no Insomnia, incluindo autenticação e operações financeiras.

# Bull - Processamento de Filas

## O que é o Bull?

O **Bull** é uma biblioteca de enfileiramento de tarefas em Node.js, que utiliza o **Redis** para armazenar e processar filas de maneira eficiente e confiável. No contexto do nosso sistema, o Bull é utilizado para o processamento de transações agendadas e também para a obtenção e aplicação de taxas de câmbio.

### Como o Bull Funciona no Projeto?

1. **Fila de Taxas de Câmbio**:
   O Bull é utilizado para agendar e processar tarefas que envolvem a obtenção de taxas de câmbio em tempo real. Isso é importante porque, ao realizar uma transação de conversão de moedas, o sistema precisa buscar a taxa de câmbio atual e aplicar essa conversão na transação.
   
2. **Fila de Transações Agendadas**:
   Além das taxas de câmbio, o Bull é também utilizado para processar transações agendadas. Por exemplo, um usuário pode agendar um pagamento ou transferência para uma data futura. O Bull garante que essas tarefas sejam executadas automaticamente no momento correto, sem a necessidade de intervenção manual.

### Vantagens de Usar o Bull

- **Escalabilidade**: O Bull suporta múltiplos processadores de fila, permitindo que o sistema escale conforme o volume de tarefas aumenta.
- **Confiabilidade**: Com o Redis como backend, o Bull oferece alta confiabilidade para armazenar e processar filas.
- **Tarefas em Segundo Plano**: Ele permite o processamento de tarefas em segundo plano sem bloquear o fluxo principal da aplicação.

---

# Taxa de Câmbio - Como Funciona?

## O que é a Taxa de Câmbio?

A **taxa de câmbio** é a relação entre o valor de duas moedas diferentes, indicando quanto de uma moeda é necessário para adquirir uma unidade de outra. Por exemplo, se a taxa de câmbio entre o **dólar** e o **real** for 5.00, significa que 1 dólar americano é equivalente a 5 reais.

No nosso sistema, a taxa de câmbio é utilizada para converter valores de uma moeda para outra, por exemplo, ao realizar transações internacionais ou depósitos em moedas diferentes.

### Como o Sistema Obtém as Taxas de Câmbio?

O sistema utiliza uma **API externa** para buscar as taxas de câmbio em tempo real. Quando uma transação envolve conversão de moedas, o Bull é responsável por garantir que a taxa de câmbio mais atualizada seja utilizada.

### Processo de Conversão de Moeda:

1. **Agendamento de Taxa de Câmbio**:
   Ao criar ou modificar uma transação que envolva uma conversão de moeda, o Bull é usado para enfileirar uma tarefa que buscará a taxa de câmbio atual na API externa.
   
2. **Aplicação da Taxa de Câmbio**:
   Uma vez que a taxa de câmbio é obtida, o sistema converte o valor da transação para a moeda destino, utilizando a taxa de câmbio mais recente.
   
3. **Transação Processada**:
   Após a conversão, a transação é processada e registrada na base de dados. Se a transação envolver várias moedas, o sistema garantirá que todas as conversões necessárias sejam feitas de forma precisa e eficiente.

### Exemplo de Como Funciona uma Conversão:

- O usuário realiza uma transferência de **100 USD** para uma conta em **BRL (real)**.
- O Bull busca a taxa de câmbio mais recente para USD/BRL (por exemplo, 1 USD = 5.00 BRL).
- A conversão é feita: `100 USD * 5.00 = 500 BRL`.
- A transferência de **500 BRL** é registrada e processada, e o valor convertido é creditado na conta de destino.

---

# Extrato de Conta

## O que é um Extrato de Conta?

Um **extrato de conta** é um relatório detalhado que mostra todas as transações realizadas em uma conta em um determinado período. Ele inclui depósitos, retiradas, transferências e o saldo atual da conta.

### Como Funciona o Extrato de Conta?

No sistema, o extrato pode ser acessado por meio de um endpoint da API, permitindo que os usuários obtenham um relatório de todas as suas transações.

- O extrato pode ser filtrado por data, tipo de transação (depósito, saque, transferência), e outros parâmetros relevantes.
- O sistema garante que todos os detalhes da transação sejam registrados corretamente, incluindo os valores, datas, contas envolvidas e as taxas de câmbio aplicadas (quando houver).

### Exemplo de Extrato:

Suponha que um usuário tenha realizado as seguintes transações em sua conta de **USD**:

1. **Depósito**: 200 USD (data: 01/03/2025)
2. **Saque**: 50 USD (data: 02/03/2025)
3. **Transferência**: 100 USD para a conta em BRL (data: 03/03/2025), com taxa de câmbio 1 USD = 5.00 BRL

O extrato do usuário para o período de 01/03/2025 a 03/03/2025 poderia ser:

| Data       | Tipo          | Valor (USD) | Valor (BRL) | Descrição               |
|------------|---------------|-------------|-------------|-------------------------|
| 01/03/2025 | Depósito      | 200.00      | -           | Depósito em USD         |
| 02/03/2025 | Saque         | -50.00      | -           | Saque em USD            |
| 03/03/2025 | Transferência | -100.00     | 500.00      | Transferência para BRL  |

O **valor em BRL** da transferência foi convertido automaticamente pelo sistema utilizando a taxa de câmbio mais recente, garantindo que o usuário receba o valor correto.

---

### Conclusão

- **Bull** é uma ferramenta poderosa para processamento assíncrono de tarefas em segundo plano, garantindo que transações e processos como a obtenção de taxas de câmbio sejam feitos de maneira eficiente e sem impactar a performance da API principal.
- O **processo de conversão de taxas de câmbio** envolve a busca em tempo real de taxas externas e a conversão de valores para as moedas desejadas.
- O **extrato de conta** fornece uma visão clara e detalhada das transações realizadas, ajudando o usuário a acompanhar suas finanças de forma precisa.

