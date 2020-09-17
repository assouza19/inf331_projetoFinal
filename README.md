# Estrutura de Arquivos e Pastas

A seguir é apresentada a estrutura de pastas do projeto:

~~~
├── README.md 
│
├── images
│
└── resources
~~~

# Projeto `MarketPlace`

# Equipe 08
* `Aline Souza Silva RG: 36.783.233-1
* `Erik Borges RG: 15.434.065
* `Lucas Trinquinato RG: 50.260.503-0
* `Paulo Sérgio Marchioreto RG: 27.303.073-5
* `Renato César Alves de Oliveira RG: 36.536.287-6

# Nível 1

Neste primeiro nível, primeiramente listamos todas as funcionalidades que contemplam o Gerenciamento de Fornecedores em MarketPlace.
Após esse primeiro levantamento, componentizamos essas funcionalidades de modo que representem módulos do sistema, conforme pode ser visto abaixo:

## Diagrama Geral do Nível 1

> ![Modelo de diagrama no nível 1](images/modelo_nivel_1.png)

### Detalhamento da interação de componentes

Para melhor compreensão das responsabilidades de cada um dos componentes observados acima, realizamos o detalhamento de suas devidas interações, seguindo o fluxo de acontecimento:

* Componente `Autenticação`:
  - Assina:
    > N/A
  - Publica:
    > * Responsável por dizer, através da interface recebida `Usuário`, se ele está autorizado ou não a realizar ações no sistema (como finalizar uma compra, cadastrar um produto, visualizar seus pedidos, etc). Essa interface `Usuário` é extendida por duas outras interfaces denominadas `Fornecedor` e `Cliente`. O componente de `Autenticação`, assim que verifica as validações necessárias, publica no barramento a mensagem de tópico `/autenticacao/cliente/{id}` ou `/autenticacao/fornecedor/{id}`, a depender do tipo de usuário enviado;
    > * Publica no barramento a mensagem de tópico `/auditoria/autenticacao/{id}/{acao}` para que o componente de `Auditoria` seja notificado sempre que ocorrer uma ação.


* Componente `Cliente`:
  - Assina:
  > * Assina no barramento mensagens de tópico "`/autenticacao/cliente/{id}`" através da interface `Autenticação`. Quando recebe a mensagem, se a autenticação tiver sido bem sucedida, o cliente estará logado e poderá prosseguir autenticado. Caso a autenticação tenha falhado, o usuário será redirecionado para uma nova tentativa;
  - Publica: 
  > * Publica no barramento a mensagem de tópico "`/cliente/{id}`" ao logar no MarketPlace através da interface `Cliente`, permitindo a persistência dos dados;
  > * Publica no barramento a mensagem de tópico "`/auditoria/cliente/{id}/{acao}`" para que o componente de `Auditoria` seja notificado sempre que ocorrer uma ação do usuário, como por exemplo, alteração dos dados cadastrais, alteração de senha, etc.

* Componente `Fornecedor`: 
  - Assina:
  > * Assina no barramento mensagens de tópico "`/autenticacao/fornecedor/{id}`" através da interface `Autenticação`. Quando recebe a mensagem, se a autenticação tiver sido bem sucedida, o fornecedor estará logado e poderá prosseguir autenticado. Caso a autenticação tenha falhado, o fornecedor será redirecionado para uma nova tentativa. 
  > * Assina no barramento mensagens de tópico "`/financeiro/transacoes`" através da interface `Financeiro`. Quando recebe a mensagem, o fornecedor terá acesso aos detalhes financeiros de suas transações dos pedidos como: acompanhamento das vendas, acompanhamento das entregas já efetuadas, notas fiscais emitidas, valores a serem repassados, entre outros.
  > * Assina no barramento mensagens de tópico "`/leilao/{id}/{idProduto}`" através da interface `Participa Leilão`. Quando recebe a mensagem, o fornecedor verificará os dados do produto requerido, sua disponibilidade em seu estoque e, caso haja interesse, fornecer uma oferta.
  - Publica: 
  > * Publica no barramento a mensagem de tópico "`/fornecedor/{id}`" através da interface `Fornecedor` onde, quem assinar esse tópico, poderá ter acesso as informações do fornecedor.
  > * Publica no barramento a mensagem de tópico "`/leilao/{id}/{idFornecedor}/{oferta}`" através da interface `Leilão` onde o componente `Recomendação` poderá ter acesso a sua oferta ou recusa do leilão.
  > * Publica no barramento a mensagem de tópico "`/auditoria/fornecedor/{id}/{acao}`" para que o componente de `Auditoria` seja notificado sempre que ocorrer uma ação do fornecedor, como por exemplo, cadastrar uma nova filial, mudar a razão social, etc.

* Componente `Produto`:
  - Assina:
    > * Assina no barramento mensagens de tópico "`/fornecedor/{id}`" através da interface `Fornecedor`. Quando ele recebe a mensagem, as informações do fornecedor são armazenadas e utilizadas em operações dos subcomponentes do componente `Produto`, como por exemplo, cadastrar um novo produto.
  - Publica: 
    > * Publica no barramento mensagens de tópico "`/produto/{id}`" através da interface `Produto` sempre que um novo produto é adicionado ao carrinho.
    > * Publica no barramento mensagens de tópico "`/auditoria/produto/{id}`" para que o componente de `Auditoria` seja notificado sempre que ocorrer uma ação com este produto como mudança de preço, mudança nos detalhes do produto ou quaisquer outras alterações pertinentes ao produto.

* Componente `Pedido`:
  - Assina:
    > * Assina no barramento mensagens de tópico "`/produto/{id}`" através da interface `Produtos Escolhidos`. Toda vez que ele recebe essa mensagem, ele armazena o produto selecionado.
    > * Assina no barramento mensagens de tópico "`/cliente/{id}`" através da interface `Cliente`, que é onde os dados estão persistidos. Quando ele recebe a mensagem, ele relaciona o `{id}` do Cliente com o `{id}` dos fornecedores responsáveis pelos produtos selecionados que constam na interface `Produtos Escolhidos`.
  - Publica: 
    > * Publica no barramento a mensagem de tópico "`/pedido/{id}`" através da interface `Pedido` sempre que um novo pedido é efetuado.
    > * Publica no barramento a mensagem de tópico "`/auditoria/pedido/{id}`" para que o componente de `Auditoria` seja notificado sempre que ocorrer uma ação com o pedido em questão (mudança de status).

* Componente `Pagamento`:
  - Assina:
    > * Assina no barramento mensagens de tópico "`/pedido/{id}`" através da interface `Pedido`. Quando recebe a mensagem, inicializa o processo de realizar o pagamento do pedido em questão.
    > * Assina no barramento mensagens de tópico "`/cliente/{id}`" através da interface `Cliente`, que é onde os dados estão persistidos. Quando ele recebe a mensagem, ele relaciona os dados cadastrais do cliente no pedido, porém, dando a possibilidade do cliente mudá-lo caso necessário, por exemplo, quando o pagamento será realizado por outra pessoa que não o próprio cliente.
  - Publica: 
    > * Publica no barramento a mensagem de tópico "`/pagamento/pedido/{id}`" através da interface `Pagamento` para notificar o componente `Entrega` do novo pedido, caso tenha sido bem sucedido.
    > * Publica no barramento a mensagem de tópico "`/auditoria/pagamento/{id}/{acao}`" para que o componente de `Auditoria` seja notificado sempre que ocorrer uma ação neste componente, como por exemplo, pagamento recusado, pagamento em análise, pagamento estornado, etc.

* Componente `Entrega`:
  - Assina:
    > * Assina no barramento mensagens de tópico "`/pagamento/pedido/{id}`" através da interface `Pagamento`. Quando recebe a mensagem, inicializa o processo de `Entrega` dos produtos selecionados.
  - Publica: 
    > * Publica no barramento as mensagens de tópico "`/entrega/{id}`" através da interface `Entrega` sempre que houver uma mudança no status do pedido.
    > * Publica no barramento a mensagem de tópico "`/auditoria/entrega/{id}/{acao}`" para que o componente de `Auditoria` seja notificado sempre que ocorrer uma ação no componente de `Entrega`.

* Componente `Recomendação`:
  - Assina:
    > * Assina no barramento mensagens de tópico "`/produto/{id}`" através da interface `Produto`, que é disparado no momento em que o usuário define qual produto deseja adquirir, iniciando assim, o processo de leilão. Quando recebe a mensagem, o componente `Recomendação` informa todos os potenciais fornecedores do produto o interesse do cliente.
    > * Assina no barramento mensagens de tópico "`/leilao/{id}/{idFornecedor}/{oferta}`" através da interface `Participa Leilão` . Quando recebe uma mensagem, o componente `Recomendação` realiza o rankeamento dos fornecedores com base na oferta e no histórico dos mesmos.
  - Publica: 
    > * Publica no barramento a mensagem de tópico "`/leilao/{id}/{idProduto}`" através da interface `Participa Leilão` para que o componente `Fornecedor` que tiver interesse seja notificado do leilão.
    > * Publica no barramento a mensagem de tópico "`/leilao/{id}`" através da interface `Recomendados` o Rankeamento que será disponibilizado na camada de apresentação para o cliente solicitante.
    > * Publica no barramento a mensagem de tópico "`/auditoria/recomendacao/{id}/{acao}`" para que o componente de `Auditoria` seja notificado sempre que ocorrer uma ação.

* Componente `Financeiro`:
  - Assina:
    > N/A
  - Publica: 
    > Publica no barramento a mensagem de tópico "`/auditoria/financeiro/{id}/{acao}`" para que o componente de `Auditoria` seja notificado sempre que ocorrer uma ação.

* Componente `Auditoria`:
  - Assina:
    > Responsável por assinar o barramento de mensagem de todas as ações que os componentes enviam ("`/auditoria/cliente`", "`/auditoria/fornecedor`", "`/auditoria/produto`", "`/auditoria/pedido`", "`/auditoria/pagamento`", "`/auditoria/entrega`", "`/auditoria/recomendacao`", "`/auditoria/autenticacao`" e "`/auditoria/financeiro`") através da `Interface Registro`. Quando recebe uma mensage, o componente realiza a integração com banco de dados para armazenar o log da ação (dados como: quem fez ação, o que fez, em que data fez) para uma possível auditoria do Sistema;
  - Publica: 
    > N/A

Descrição dos componentes:

## Componente `Autenticação`

> Componente responsável pelo processo de autenticação e persistência do Login no MarketPlace. Disponibiliza serviços como: Login, Logout, Recuperar a senha.

![AutenticacaoComponent](images/Autenticacao.png)

**Interfaces**
> * Interface Autenticação
> * Interface Usuário

As interfaces listadas são detalhadas a seguir:

## Detalhamento das Interfaces

### Interface `Autenticação`

> Essa interface é responsável por enviar o status da autenticação;

**Tópico**: `<tópico que a respectiva interface assina ou publica>`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
<Formato da mensagem JSON associada ao objeto enviado/recebido por essa interface.>
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`<nome do atributo>` | `<objetivo do atributo>`

## Interface `Usuário`

> Resumo do papel da interface.

**Tópico**: `<tópico que a respectiva interface assina ou publica>`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
<Formato da mensagem JSON associada ao objeto enviado/recebido por essa interface.>
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`<nome do atributo>` | `<objetivo do atributo>`


## Componente `Cliente`

> Este componente é responsável por todo o gerenciamento do Cliente. Serviços como: Manter o cadastro do cliente........

![ClienteComponent](images/Cliente.png)

**Interfaces**
> * Interface Autenticação
> * Interface Cliente

As interfaces listadas são detalhadas a seguir:

## Detalhamento das Interfaces

### Interface `Autenticação`

> Resumo do papel da interface.

**Tópico**: `<tópico que a respectiva interface assina ou publica>`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
<Formato da mensagem JSON associada ao objeto enviado/recebido por essa interface.>
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`<nome do atributo>` | `<objetivo do atributo>`

## Interface `Cliente`

> Resumo do papel da interface.

**Tópico**: `<tópico que a respectiva interface assina ou publica>`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
<Formato da mensagem JSON associada ao objeto enviado/recebido por essa interface.>
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`<nome do atributo>` | `<objetivo do atributo>`


## Componente `Fornecedor`

> Este componente é responsável por gerenciar o fornecedor. Desde o cadastro, até manter seus dados atualizados.

![FornecedorComponent](images/Fornecedor.png)

**Interfaces**
> * Interface Fornecedor
> * Interface Financeiro
> * Interface Autenticação
> * Interface Leilão
> * Interface ParticipaLeilão

As interfaces listadas são detalhadas a seguir:

## Detalhamento das Interfaces

### Interface `Fornecedor`

> Resumo do papel da interface.

**Tópico**: `<tópico que a respectiva interface assina ou publica>`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
<Formato da mensagem JSON associada ao objeto enviado/recebido por essa interface.>
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`<nome do atributo>` | `<objetivo do atributo>`

### Interface `Financeiro`

> Interface para envio de relatório financeiro ao fornecedor.

**Tópico**:  
Assina: 
`pedido/{id}/dados` 
Publica: 
`financeiro/relatorios/{id}`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest-relatorio-financeiro.png)

~~~json
{
  "idRelatorio": 23421,
  "cnpj": 23242121233,
  "periodo": {
    "inicio": "2009-10-04",
    "fim": "2009-11-04",
  },
  "totalVendas": 14200.00,
  "quantidadeProdutos": 17,
  "produtos": {
    "produto": {
       "idProduto": "1245",
	   "valor": 2000.00,
       "quantitidade": 4
    },
    "produto": {
       "idProduto": "3323",
	   "valor": 100.00,
       "quantitidade": 2
    },
	"produto": {
       "idProduto": "5555",
	   "valor": 500.00,
       "quantitidade": 4
    },
	"produto": {
       "idProduto": "9931",
	   "valor": 200.00,
       "quantitidade": 5
    },
	"produto": {
       "idProduto": "6633",
	   "valor": 1500.00,
       "quantitidade": 2
    },
  }  
}
~~~

Detalhamento da mensagem JSON:

**Relatório Financeiro**
Atributo | Descrição
-------| --------
idRelatorio | identificador do relatório
cnpj | cnpj do fornecedor que solicitou o relatorio
periodo | período a ser considerado para o a construção do relatório
totalVendas | valor total do montante de vendas no período
quantidadeProdutos | quantidade produtos vendidos no período

**Produto**
Atributo | Descrição
-------| --------
idProduto | identificador do produto
valor | valor do produto
quantidade | quantidade de determinado produto vendida no período

### Interface `Autenticação`

> Resumo do papel da interface.

**Tópico**: `<tópico que a respectiva interface assina ou publica>`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
<Formato da mensagem JSON associada ao objeto enviado/recebido por essa interface.>
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`<nome do atributo>` | `<objetivo do atributo>`

### Interface `Leilão`

> Resumo do papel da interface.

**Tópico**: `<tópico que a respectiva interface assina ou publica>`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
<Formato da mensagem JSON associada ao objeto enviado/recebido por essa interface.>
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`<nome do atributo>` | `<objetivo do atributo>`

### Interface `ParticipaLeilão`

> Resumo do papel da interface.

**Tópico**: `<tópico que a respectiva interface assina ou publica>`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
<Formato da mensagem JSON associada ao objeto enviado/recebido por essa interface.>
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`<nome do atributo>` | `<objetivo do atributo>`

## Componente `Produto`

> Este componente é responsável por manter os dados do produto, desde o cadastro do mesmo, como categorização......

![ProdutoComponent](images/Produto.png)

**Interfaces**
> * Interface Produto
> * Interface Fornecedor

A interface listada será detalhada a seguir:

## Detalhamento da Interface

### Interface `Produto`

> Resumo do papel da interface.

**Tópico**: `<tópico que a respectiva interface assina ou publica>`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
<Formato da mensagem JSON associada ao objeto enviado/recebido por essa interface.>
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`<nome do atributo>` | `<objetivo do atributo>`

### Interface `Fornecedor`

> Resumo do papel da interface.

**Tópico**: `<tópico que a respectiva interface assina ou publica>`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
<Formato da mensagem JSON associada ao objeto enviado/recebido por essa interface.>
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`<nome do atributo>` | `<objetivo do atributo>`

## Componente `Pedido`

> Este componente é responsável por todo o fluxo do pedido. Desde quando o cliente encontra o produto desejado, visualiza sua descrição e o coloca no carrinho.

![PedidoComponent](images/Pedido.png)

**Interfaces**
> * Interface Pedido
> * Interface ProdutosEscolhidos
> * Interface Cliente

As interfaces listadas são detalhadas a seguir:

## Detalhamento das Interfaces

### Interface `Pedido`

> Resumo do papel da interface.

**Tópico**: `<tópico que a respectiva interface assina ou publica>`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
<Formato da mensagem JSON associada ao objeto enviado/recebido por essa interface.>
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`<nome do atributo>` | `<objetivo do atributo>`

### Interface `ProdutosEscolhidos`

> Resumo do papel da interface.

**Tópico**: `<tópico que a respectiva interface assina ou publica>`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
<Formato da mensagem JSON associada ao objeto enviado/recebido por essa interface.>
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`<nome do atributo>` | `<objetivo do atributo>`

### Interface `Cliente`

> Resumo do papel da interface.

**Tópico**: `<tópico que a respectiva interface assina ou publica>`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
<Formato da mensagem JSON associada ao objeto enviado/recebido por essa interface.>
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`<nome do atributo>` | `<objetivo do atributo>`


## Componente `Pagamento`

> Este componente é responsável pelo pagamento efetivo do pedido realizado. Dentre os serviços estão: Cálculo de frete, consulta do cadastro do usuário, consulta da instituição financeira responsável pelo pagamento e confirmação do pedido para o componente  de entrega.

![PagamentoComponent](images/Pagamento.png)

**Interfaces**
> * Interface Pagamento
> * Interface Pedido
> * Interface Cliente

As interfaces listadas são detalhadas a seguir:

## Detalhamento das Interfaces

### Interface `Pagamento`

> Interface para envio dos detalhes do pafamento.

**Tópico**:  
Assina: 
`produto/{id}/dados`
`cliente/{id}/dados` 
Publica: 
`pagamento/{id}/dados`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest-pagamento.png)

~~~json
{
  "idPagamento": 44323,
  "cpf": 33242121233,
  "dataPagamento": "2009-10-04",
  "valor": 87.66,
  "formaDePagamento": "Cartao de Credito",
  "numeroParcelas": 9,
  "statusPagamento": "Pendente",
  "produtos": {
    "produto": {
       "idProduto": "1245",
	   "valor": 55.33,
       "quantitidade": 1
    },
    "produto": {
       "idProduto": "3323",
	   "valor": 32.33,
       "quantitidade": 1
    },
  }  
}
~~~

Detalhamento da mensagem JSON:

**Pagamento**
Atributo | Descrição
-------| --------
idPagamento | identificador do pagamento
cpf | cpf do cliente
dataPagamento | data da efetivação do pagamento
valor | valor total do pedido
formaDePagamento | forma de pagamento (cartao crédito, débito, boleto, etc)
numeroParcelas | quantidade de parcelas, caso parcelado, valor default 1
statusPagamento | status do pagamento (pendente, recusado, finalizado)

**Produto**
Atributo | Descrição
-------| --------
idProduto | identificador do produto
valor | valor do produto
quantidade | quantidade de determinado produto no pedido do cliente


### Interface `Pedido`

> Resumo do papel da interface.

**Tópico**: `<tópico que a respectiva interface assina ou publica>`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
<Formato da mensagem JSON associada ao objeto enviado/recebido por essa interface.>
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`<nome do atributo>` | `<objetivo do atributo>`

### Interface `Cliente`

> Resumo do papel da interface.

**Tópico**: `<tópico que a respectiva interface assina ou publica>`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
<Formato da mensagem JSON associada ao objeto enviado/recebido por essa interface.>
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`<nome do atributo>` | `<objetivo do atributo>`

## Componente `Recomendação`

> <Resumo do papel do componente e serviços que ele oferece.>

![RecomendacaoComponent](images/Recomendacao.png)

**Interfaces**
> * Interface Produto
> * Interface Recomendados
> * Interface Leilão
> * Interface ParticipaLeilão

As interfaces listadas são detalhadas a seguir:

## Detalhamento das Interfaces

### Interface `Produto`

> Resumo do papel da interface.

**Tópico**: `<tópico que a respectiva interface assina ou publica>`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
<Formato da mensagem JSON associada ao objeto enviado/recebido por essa interface.>
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`<nome do atributo>` | `<objetivo do atributo>`

### Interface `Recomendados`

> Resumo do papel da interface.

**Tópico**: `<tópico que a respectiva interface assina ou publica>`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
<Formato da mensagem JSON associada ao objeto enviado/recebido por essa interface.>
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`<nome do atributo>` | `<objetivo do atributo>`

### Interface `Leilão`

> Resumo do papel da interface.

**Tópico**: `<tópico que a respectiva interface assina ou publica>`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
<Formato da mensagem JSON associada ao objeto enviado/recebido por essa interface.>
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`<nome do atributo>` | `<objetivo do atributo>`

### Interface `ParticipaLeilão`

> Resumo do papel da interface.

**Tópico**: `<tópico que a respectiva interface assina ou publica>`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
<Formato da mensagem JSON associada ao objeto enviado/recebido por essa interface.>
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`<nome do atributo>` | `<objetivo do atributo>`

## Componente `Entrega`

> <Resumo do papel do componente e serviços que ele oferece.>

![EntregaComponent](images/Entrega.png)

**Interfaces**
> * Interface Pagamento
> * Interface Entrega

As interfaces listadas são detalhadas a seguir:

## Detalhamento das Interfaces

### Interface `Entrega`

> Interface para envio dos produtos efetivamente comprados.

**Tópico**:  
Assina: 
`pedido/{id}/dados`
Publica: 
`etrega/{id}/dados`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest-entrega.png)

~~~json
{
  "codRastreio": 455231,
  "cpf": 33242121233,
  "cep": 12332020,
  "numero": 233,
  "valor": 12.22,
  "estimativaEntrega": "2009-10-04",
  "produtos": {
    "produto": {
       "idProduto": "1245",
       "quantitidade": 1,
	   "dimensoes": {
			"largura": 22,
			"comprimento": 33,
			"altura": 45,
	   },
	   "peso": 9.22,
    },
    "produto": {
       "idProduto": "3323",
       "quantitidade": 3,
	   "dimensoes": {
			"largura": 11,
			"comprimento": 44,
			"altura": 55,
	   },
	   "peso": 8.12,
    },
  }  
}
~~~

Detalhamento da mensagem JSON:

**Pagamento**
Atributo | Descrição
-------| --------
codRastreio | identificador da entrega e codigo de rastreio da entrega para o cliente.
cpf | cpf do cliente (destinatario)
cep | CEP do destino da entrega
numero | número do destino da entrega
valor | valor cobrado pela transportadora pelo frete
estimativaEntrega | data estimada para a entrega

**Produto**
Atributo | Descrição
-------| --------
idProduto | identificador do produto
valor | valor do produto
quantidade | quantidade de determinado produto no pedido de entrega
dimensões | dimensões do produto em cm
peso | peso do produto em Kg

### Interface `Pagamento`

> Resumo do papel da interface.

**Tópico**: `<tópico que a respectiva interface assina ou publica>`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
<Formato da mensagem JSON associada ao objeto enviado/recebido por essa interface.>
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`<nome do atributo>` | `<objetivo do atributo>`

## Componente `Financeiro`

> <Resumo do papel do componente e serviços que ele oferece.>

![FinanceiroComponent](images/Financeiro.png)


**Interfaces**
> * Interface Pagamento
> * Interface Financeiro

As interfaces listadas são detalhadas a seguir:

## Detalhamento das Interfaces

### Interface `Financeiro`

> Interface para envio de relatório financeiro ao fornecedor.

**Tópico**:  
Assina: 
`pedido/{id}/dados` 
Publica: 
`financeiro/relatorios/{id}`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest-relatorio-financeiro.png)

~~~json
{
  "idRelatorio": 23421,
  "cnpj": 23242121233,
  "periodo": {
    "inicio": "2009-10-04",
    "fim": "2009-11-04",
  },
  "totalVendas": 14200.00,
  "quantidadeProdutos": 17,
  "produtos": {
    "produto": {
       "idProduto": "1245",
	   "valor": 2000.00,
       "quantitidade": 4
    },
    "produto": {
       "idProduto": "3323",
	   "valor": 100.00,
       "quantitidade": 2
    },
	"produto": {
       "idProduto": "5555",
	   "valor": 500.00,
       "quantitidade": 4
    },
	"produto": {
       "idProduto": "9931",
	   "valor": 200.00,
       "quantitidade": 5
    },
	"produto": {
       "idProduto": "6633",
	   "valor": 1500.00,
       "quantitidade": 2
    },
  }  
}
~~~

Detalhamento da mensagem JSON:

**Relatório Financeiro**
Atributo | Descrição
-------| --------
idRelatorio | identificador do relatório
cnpj | cnpj do fornecedor que solicitou o relatorio
periodo | período a ser considerado para o a construção do relatório
totalVendas | valor total do montante de vendas no período
quantidadeProdutos | quantidade produtos vendidos no período

**Produto**
Atributo | Descrição
-------| --------
idProduto | identificador do produto
valor | valor do produto
quantidade | quantidade de determinado produto vendida no período

### Interface `Pagamento`

> Resumo do papel da interface.

**Tópico**: `<tópico que a respectiva interface assina ou publica>`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
<Formato da mensagem JSON associada ao objeto enviado/recebido por essa interface.>
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`<nome do atributo>` | `<objetivo do atributo>`

## Componente `Auditoria`

> <Resumo do papel do componente e serviços que ele oferece.>

![AuditoriaComponent](images/Auditoria.png)

**Interfaces**
> * Interface Registro
> * Interface Auditoria

As interfaces listadas são detalhadas a seguir:

## Detalhamento das Interfaces

### Interface `Registro`

> Interface para aquisição de logs para auditoria.

**Tópico**:  
Assina: 
`pedido/{id}/dados` 

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest-registro.png)

~~~json
{
  "idRegistro": 23421,
  "timestamp": "2012-01-19 03:14:07",
  "infos": {
	"quandidadeProdutos" : 2,
	"idUsuario": 122333,
	"valorTotal": 2233.23,
  },
  "md5": "92e9f1ae8ef91056bf6ba49cd79f0869"
}
~~~

Detalhamento da mensagem JSON:

**Registro**
Atributo | Descrição
-------| --------
idRegistro | identificador do registro
timestamp | timestamp com a hora exata do pedido
infos |  detalhes do pedido sendo auditado
md5 |  hash do arquivo de log da operação

### Interface `Auditoria`

> Resumo do papel da interface.

**Tópico**: `<tópico que a respectiva interface assina ou publica>`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
<Formato da mensagem JSON associada ao objeto enviado/recebido por essa interface.>
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`<nome do atributo>` | `<objetivo do atributo>`

# Nível 2

Apresente aqui o detalhamento do Nível 2 conforme detalhado na especificação com, no mínimo, as seguintes subseções:

## Diagrama do Nível 2

Apresente um diagrama conforme o modelo a seguir:

> ![Modelo de diagrama no nível 2](images/diagrama-subcomponentes.png)

### Detalhamento da interação de componentes

O detalhamento deve seguir um formato de acordo com o exemplo a seguir:

* O componente `Entrega Pedido Compra` assina no barramento mensagens de tópico "`pedido/+/entrega`" através da interface `Solicita Entrega`.
  * Ao receber uma mensagem de tópico "`pedido/+/entrega`", dispara o início da entrega de um conjunto de produtos.
* Os componentes `Solicita Estoque` e `Solicita Compra` se comunicam com componentes externos pelo barramento:
  * Para consultar o estoque, o componente `Solicita Estoque` publica no barramento uma mensagem de tópico "`produto/<id>/estoque/consulta`" através da interface `Consulta Estoque` e assina mensagens de tópico "`produto/<id>/estoque/status`" através da interface `Posição Estoque` que retorna a disponibilidade do produto.

Para cada componente será apresentado um documento conforme o modelo a seguir:

## Componente `<Nome do Componente>`

> <Resumo do papel do componente e serviços que ele oferece.>

![Componente](images/diagrama-componente.png)

**Interfaces**
> * Listagem das interfaces do componente.

As interfaces listadas são detalhadas a seguir:

## Detalhamento das Interfaces

### Interface `<nome da interface>`

> ![Diagrama da Interface](images/diagrama-interface-itableproducer.png)

> <Resumo do papel da interface.>

Método | Objetivo
-------| --------
`<id do método>` | `<objetivo do método e descrição dos parâmetros>`

## Exemplos:

### Interface `ITableProducer`

![Diagrama da Interface](images/diagrama-interface-itableproducer.png)

Interface provida por qualquer fonte de dados que os forneça na forma de uma tabela.

Método | Objetivo
-------| --------
`requestAttributes` | Retorna um vetor com o nome de todos os atributos (colunas) da tabela.
`requestInstances` | Retorna uma matriz em que cada linha representa uma instância e cada coluna o valor do respectivo atributo (a ordem dos atributos é a mesma daquela fornecida por `requestAttributes`.

### Interface `IDataSetProperties`

![Diagrama da Interface](images/diagrama-interface-idatasetproperties.png)

Define o recurso (usualmente o caminho para um arquivo em disco) que é a fonte de dados.

Método | Objetivo
-------| --------
`getDataSource` | Retorna o caminho da fonte de dados.
`setDataSource` | Define o caminho da fonte de dados, informado através do parâmetro `dataSource`.

# Multiplas Interfaces

> Escreva um texto detalhando como seus componentes  podem ser preparados para que seja possível trocar de interface apenas trocando o componente View e mantendo o Model e Controller.
>
> É recomendado a inserção de, pelo menos, um diagrama que deve ser descrito no texto. O formato do diagrama é livre e deve ilustrar a arquitetura proposta.
