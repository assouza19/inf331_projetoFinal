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
    > * Assina no barramento mensagens de tópico "`/cliente/{id}`" e "`/fornecedor/{id}`" através da interface `Usuario`, pois ambos os usuários (`cliente` e `Fornecedor` são interfaces que extendem de `Usuário`). Depois de receber a mensagem, o componente `Autenticação` inicializa o processo de autenticação do usuário.
  - Publica:
    > * Responsável por dizer, através da interface recebida `Autenticação`, se ele está autorizado ou não a realizar ações no sistema (como finalizar uma compra, cadastrar um produto, visualizar seus pedidos, etc). Essa interface `Usuário` é extendida por duas outras interfaces denominadas `Fornecedor` e `Cliente`. O componente de `Autenticação`, assim que verifica as validações necessárias, publica no barramento a mensagem de tópico `/autenticacao/cliente/{id}` ou `/autenticacao/fornecedor/{id}`, a depender do tipo de usuário enviado;
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
    > * Publica no barramento a mensagem de tópico "`/pagamento/pedido/{id}`" através da interface `Pagamento` para notificar o componente `Entrega` do novo pedido e também ao componente `Financeiro` para emitir nota fiscal entre outros, caso tenha sido bem sucedido.
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
    > * Publica no barramento a mensagem de tópico "`/leilao/{id}/recomendados`" através da interface `Recomendados` o Rankeamento que será disponibilizado na camada de apresentação para o cliente solicitante.
    > * Publica no barramento a mensagem de tópico "`/auditoria/recomendacao/{id}/{acao}`" para que o componente de `Auditoria` seja notificado sempre que ocorrer uma ação.

* Componente `Financeiro`:
  - Assina:
    > * Assina no barramento mensagens de tópico "`/pagamento/pedido/{id}`" através da interface `Pagamento`. Quando recebe a mensagem, o componente `Financeiro` armazena as informações do pedido efetuado, emite nota fiscal, atualiza dashboards e relatórios gerenciais.
  - Publica: 
    > * Publica no barramento a mensagem de tópico "`/financeiro/transacoes`" através da interface `Financeiro` para notificar ao fornecedor que uma nova venda foi concretizada.
    > * Publica no barramento a mensagem de tópico "`/auditoria/financeiro/{id}/{acao}`" para que o componente de `Auditoria` seja notificado sempre que ocorrer uma ação.

* Componente `Auditoria`:
  - Assina:
    > * Responsável por assinar o barramento de mensagem de todas as ações que os componentes enviam ("`/auditoria/cliente/{id}/{acao}`", "`/auditoria/fornecedor/{id}/{acao}`", "`/auditoria/produto/{id}/{acao}`", "`/auditoria/pedido/{id}/{acao}`", "`/auditoria/pagamento/{id}/{acao}`", "`/auditoria/entrega/{id}/{acao}`", "`/auditoria/recomendacao/{id}/{acao}`", "`/auditoria/autenticacao/{id}/{acao}`" e "`/auditoria/financeiro/{id}/{acao}`") através da `Interface Registro`. Quando recebe uma mensage, o componente realiza a integração com banco de dados para armazenar o log da ação (dados como: quem fez ação, o que fez, em que data fez) para uma possível auditoria do Sistema;
  - Publica: 
    > * N/A

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

**Tópico**:  
Publica: 
* `/autenticacao/cliente/{id}` 
* `/autenticacao/fornecedor/{id}`
> A publicação do tópico correto dependerá do tipo de `Usuário` que está tentando se autenticar.

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

> Esta é uma interface abstraída, pois a autenticação poderá ser feita tanto por fornecedor quanto por cliente.
> Portanto, criamos essa interface "pai" que será extendida pelas interfaces `Cliente` e `Fornecedor`

**Tópico**: 
Assina:
* `/cliente/{id}`
* `/fornecedor/{id}`
> A assinatura do tópico correto dependerá do tipo de `Usuário` que está tentando se autenticar.

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

> Este componente é responsável por todo o gerenciamento do Cliente. Serviços como: Manter o cadastro do cliente, visualizar seus pedidos, cadastrar endereços de entrega, etc.

![ClienteComponent](images/Cliente.png)

**Interfaces**
> * Interface Autenticação
> * Interface Cliente

As interfaces listadas são detalhadas a seguir:

## Detalhamento das Interfaces

### Interface `Autenticação`

> Essa interface é responsável por receber o status da autenticação que foi realizada no componente de `Autenticação`;

**Tópico**: 
Assina:
* `/autenticacao/cliente/{id}`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
{
	"token": "MKT43242DSAD",
	"idCliente": 543,
	"nome": "Carlos",
	"autenticado": true
}
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`token` | `Um hash gerado que expira quando o usuário deixa de interagir com o MarketPlace por muito tempo, forçando-o a logar novamente`
`idCliente` | `identificador único do cliente que foi autenticado (utilizado para logs, por exemplo)`
`nome` | `Primeiro nome do cliente, para ser utilizado no front-end`
`autenticado` | `Status da autenticação. Caso bem sucedida, retornará true. Caso não, retornará false (senha incorreta, e-mail inválido, cliente inexistente, etc)`

## Interface `Cliente`

> Esta interface é responsável por fornecer uma instância da interface `Cliente` com os dados do mesmo, que já foi autenticado, para uso posterior (por exemplo, efetivar uma compra).

**Tópico**: 
Publica:
* `/cliente/{id}`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest-cliente.png)

~~~json
{
  "id": 1290,
  "nome": "João Carlos Pereira",
  "documento": "20038924860",
  "tipoDePessoa": "PF",
  "endereco": {
    "logradouro": "Rua 13 de maio, 10",
    "bairro": "Centro",
    "cidade": "Itapira",
    "uf": "SP",
    "cep": "13970970",
  },
}
~~~

Detalhamento da mensagem JSON:

**Cliente**
Atributo | Descrição
-------| --------
`id` | `Identificador do cliente`
`nome` | `Nome do cliente`
`documento` | `Documento do cliente, podendo ser CPF ou CNPJ`
`tipoDePessoa` | `Tipo de pessoa, podendo ser PF ou PJ`
`endereco` | `Informações de endereço do cliente`

**Endereço**
Atributo | Descrição
-------| --------
`logradouro` | `Logradouro do cliente`
`bairro` | `Bairro do cliente`
`cidade` | `Cidade do cliente`
`uf` | `Estado do cliente`
`cep` | `CEP do cliente`

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

> Esta interface é responsável por fornecer uma instância da interface `Fornecedor` com os dados do mesmo, que já foi autenticado, para uso posterior (por exemplo, realizar o cadastro de um novo produto).

**Tópico**: 
Publica:
* `/fornecedor/{id}`

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

> Interface que receberá as informações do `Financeiro`

**Tópico**:  
Assina: 
* `/financeiro/transacoes` 

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest-relatorio-financeiro.png)

~~~json
{
	"idRelatorio": 23421,
	"cnpj": 23242121233,
	"periodo": {
		"inicio": "2009-10-04",
		"fim": "2009-11-04"
	},
	"totalVendas": 14200.00,
	"quantidadeProdutos": 17,
	"vendas": [
    {
		"id": 1,
    "pagamento": "dinheiro",
		"produtos": [
      {
				"idProduto": "1245",
				"valor": 2000.00,
				"quantitidade": 4
			},
			{
				"idProduto": "3323",
				"valor": 100.00,
				"quantitidade": 2
			},
			{
				"idProduto": "5555",
				"valor": 500.00,
				"quantitidade": 4
			},
			{
				"idProduto": "9931",
				"valor": 200.00,
				"quantitidade": 5
			},
			{
				"idProduto": "6633",
				"valor": 1500.00,
				"quantitidade": 2
			}
		]
	}
]
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
vendas | lista de pedidos, contendo os produtos e forma de pagamento realizadas dentro do período 

**Produto**
Atributo | Descrição
-------| --------
idProduto | identificador do produto
valor | valor do produto
quantidade | quantidade de determinado produto vendida no período

### Interface `Autenticação`

> Essa interface é responsável por receber o status da autenticação que foi realizada no componente de `Autenticação`;

**Tópico**: 
Assina:
* `/autenticacao/fornecedor/{id}`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
{
	"token": "MKT43254787DSAD",
	"idFornecedor": 231,
	"razaoSocial": "Magazine Luiza SA",
	"autenticado": true
}
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`token` | `Um hash gerado que expira quando o usuário deixa de interagir com o MarketPlace por muito tempo, forçando-o a logar novamente`
`idFornecedor` | `identificador único do fornecedor que foi autenticado (utilizado para logs, por exemplo)`
`razaoSocial` | `Razão social do fornecedor logado para ser utilizado no front-end`
`autenticado` | `Status da autenticação. Caso bem sucedida, retornará true. Caso não, retornará false (senha incorreta, e-mail inválido, fornecedor inexistente, etc)`

### Interface `Leilão`

> Essa interface é responsável por avisar ao componente `Recomendação` se o fornecedor deseja participar do leilão, juntamente com o valor oferecido;

**Tópico**:
Publica:
* `/leilao/{id}/{idFornecedor}/{oferta}`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
{
	"idLeilao": 1,
	"idFornecedor": 342,
	"aceitaParticipar": true,
	"lance": 200.0
}
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`idLeilao` | `Fornecer ao componente Recomendação qual a referência do leilão`
`idFornecedor` | `Fornecer ao componente Recomendação qual fornecedor respondeu sua mensagem`
`aceitaParticipar` | `Informar ao componente Recomendação se deseja ou não participar do leilão`
`lance` | `Caso aceite participar, deverá informar o lance oferecido para o produto em questão`

### Interface `ParticipaLeilão`

> Esta interface é responsável por receber um novo pedido de leilão. Nela constam informações do produto desejado.

**Tópico**: 
Assina:
* `/leilao/{id}/{idProduto}`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
{
	"idLeilao": 432,
	"idProduto": "A9842",
	"descricaoProduto": "Televisão LG 50 polegadas"
}
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`idLeilao` | `Fornecer ao componente Fornecedor qual a referência do leilão`
`idProduto` | `Fornecer ao componente Fornecedor qual o identificador único do produto`
`descricaoProduto` | `Descreve o produto desejado pelo cliente`

## Componente `Produto`

> Este componente é responsável por manter os dados do produto, desde o cadastro do mesmo, categorização, precificação, buscar o produto referido no leilão, etc.

![ProdutoComponent](images/Produto.png)

**Interfaces**
> * Interface Produto
> * Interface Fornecedor

A interface listada será detalhada a seguir:

## Detalhamento da Interface

### Interface `Produto`

> A interface produto é provida pelo componente com todos os detalhes do produto, tais como, categoria, fornecedor, precificação e também busca o produto referido no leilão de acordo com o fornecedor.

**Tópico**: 
Publica:
* `/produto/{id}`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
{
	"id": 42,
	"idCategoria": 231,
	"idFornecedor": 123,
	"idFabricante": 786,
  "titulo": "Televisao LG",
	"descricaoProduto": "Televisão Smart TV LG 50 polegadas com Wifi ",
	"precoUnitario": 2500.00,
	"imagens": [{
			"url": "imagetelevisaoFrontal.png",
			"contentDescription": "Foto da televisão de frente"
		},
		{
			"url": "imagetelevisaoLateral.png",
			"contentDescription": "Foto da televisão de lado"
		}
	]
}
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`id` | `Identificador único do produto`
`idCategoria` | `Identificador único da categoria do produto`
`idFornecedor` | `Identificador único do fornecedor, para saber quem está fornecendo`
`idFabricante` | `Identificador único do fabricante, para saber quem produziu o produto`
`titulo` | `Titulo que aparecerá para o cliente antes que ele abra os detalhes`
`descricaoProduto` | `Descreve o produto detalhadamente`
`precoUnitario` | `Valor unitário`
`imagens` | `Listagem das imagens que o produto possui`

### Interface `Fornecedor`

> Esta interface é responsável por fornecer uma instância da interface `Fornecedor` com os dados do mesmo, que já foi autenticado, para uso posterior (por exemplo, realizar o cadastro de um novo produto, ou para obter detalhes do fornecedor na tela de detalhes do produto).

**Tópico**: 
Assina:
* `/fornecedor/{id}`

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

> Interface responsável pelo envio dos dados do pedido (incluindo descrição dos produtos escolhidos, cliente que realizou a compra e dados do fornecedor).

**Tópico**:
Publica: 
* `pedido/{id}`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest-pedido.png)

```json
{
  "idPedido": 1500,
  "data": 2020-08-09,
  "cnpjFornecedor": 23242121233,
  "idCliente": 3200,
  "valorTotal": 200.00,
  "formaDePagamento": "Cartão de Crédito",
  "produtos": [
    {
     "idProduto": "1245",
	   "valor": 50.00,
     "quantitidade": 1
    },
    {
     "idProduto": "3323",
	   "valor": 150.00,
     "quantitidade": 1
    },
  ],
}
```

Detalhamento da mensagem JSON:

**Pedido**

| Atributo             | Descrição                |
| -------------------- | ------------------------ |
| `idPedido` | `Identificador do pedido` |
| `data` | `Data da realização do pedido` |
| `cnpjFornecedor` | `CNPJ do fornecedor que está atendendo o pedido` |
| `idCliente` | `Identificador do cliente que realizou o pedido` |
| `valorTotal` | `Valor total do pedido` |
| `formaDePagamento` | `Forma de pagamento utilizada`
| `produtos` | `Relação de produtos do pedido` |

**Produto**

Atributo | Descrição
-------| --------
`idProduto` | `Identificador do produto`
`valor` | `Valor do produto`
`quantidade` | `Quantidade de determinado produto no pedido do cliente`

### Interface `ProdutosEscolhidos`

> Interface responsável por receber os produtos escolhidos pelo cliente, com base nas recomendações do leilão 

**Tópico**: 
Assina: 
* `/produto/{id}`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest-recomendacao.png)

~~~json
{
  "idPedido": 1340,
  "produtos": [
    {
     "idProduto": "1245",
	   "valor": 50.00,
     "quantitidade": 1,
     "idFornecedor": 302
    },
    {
     "idProduto": "3323",
	   "valor": 150.00,
     "quantitidade": 1,
     "idFornecedor": 502
    },
  ],
}
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`idPedido` | `Identificador do pedido de venda`
`produtos` | `Relação de produtos escolhidos`


### Interface `Cliente`

> Interface responsável por receber os dados do cliente.

**Tópico**: 
Assina: 
* `/cliente/{id}`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest-cliente.png)

~~~json
{
  "id": 1290,
  "nome": "João Carlos Pereira",
  "documento": "20038924860",
  "tipoDePessoa": "PF",
  "endereco": {
    "logradouro": "Rua 13 de maio, 10",
    "bairro": "Centro",
    "cidade": "Itapira",
    "uf": "SP",
    "cep": "13970970",
  },
}
~~~

Detalhamento da mensagem JSON:

**Cliente**
Atributo | Descrição
-------| --------
`id` | `Identificador do cliente`
`nome` | `Nome do cliente`
`documento` | `Documento do cliente, podendo ser CPF ou CNPJ`
`tipoDePessoa` | `Tipo de pessoa, podendo ser PF ou PJ`
`endereco` | `Informações de endereço do cliente`

**Endereço**
Atributo | Descrição
-------| --------
`logradouro` | `Logradouro do cliente`
`bairro` | `Bairro do cliente`
`cidade` | `Cidade do cliente`
`uf` | `Estado do cliente`
`cep` | `CEP do cliente`


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
Publica: 
`pagamento/pedido/{id}`

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

> Interface responsável pelo recebimento do pedido, com todos os produtos selecionados pelo cliente.

**Tópico**:
Assina: `/pedido/{id}`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest-pedido.png)

```json
{
  "idPedido": 1500,
  "data": 2020-08-09,
  "cnpjFornecedor": 23242121233,
  "idCliente": 3200,
  "valorTotal": 200.00,
  "formaDePagamento": "Cartão de Crédito",
  "produtos": [
    {
     "idProduto": "1245",
	   "valor": 50.00,
     "quantitidade": 1
    },
    {
     "idProduto": "3323",
	   "valor": 150.00,
     "quantitidade": 1
    },
  ],
}
```

Detalhamento da mensagem JSON:

**Pedido**

| Atributo             | Descrição                |
| -------------------- | ------------------------ |
| `idPedido` | `Identificador do pedido` |
| `data` | `Data da realização do pedido` |
| `cnpjFornecedor` | `CNPJ do fornecedor que está atendendo o pedido` |
| `idCliente` | `Identificador do cliente que realizou o pedido` |
| `valorTotal` | `Valor total do pedido` |
| `formaDePagamento` | `Forma de pagamento utilizada`
| `produtos` | `Relação de produtos do pedido` |

**Produto**

Atributo | Descrição
-------| --------
`idProduto` | `Identificador do produto`
`valor` | `Valor do produto`
`quantidade` | `Quantidade de determinado produto no pedido do cliente`



### Interface `Cliente`

> Interface responsável por receber os dados do cliente .

**Tópico**: 
Assina: 
* `cliente/#`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest-cliente.png)

~~~json
{
  "id": 1290,
  "nome": "João Carlos Pereira",
  "documento": "20038924860",
  "tipoDePessoa": "PF",
  "endereco": {
    "logradouro": "Rua 13 de maio, 10",
    "bairro": "Centro",
    "cidade": "Itapira",
    "uf": "SP",
    "cep": "13970970",
  },
}
~~~

Detalhamento da mensagem JSON:

**Cliente**
Atributo | Descrição
-------| --------
`id` | `Identificador do cliente`
`nome` | `Nome do cliente`
`documento` | `Documento do cliente, podendo ser CPF ou CNPJ`
`tipoDePessoa` | `Tipo de pessoa, podendo ser PF ou PJ`
`endereco` | `Informações de endereço do cliente`

**Endereço**
Atributo | Descrição
-------| --------
`logradouro` | `Logradouro do cliente`
`bairro` | `Bairro do cliente`
`cidade` | `Cidade do cliente`
`uf` | `Estado do cliente`
`cep` | `CEP do cliente`


## Componente `Recomendação`

> Este componente possui a principal função de orquestrar o processo de leilão, que envolve: Receber o produto desejado pelo cliente, notificar fornecedores de um novo leilão, buscar histórico dos mesmos, realizar o rankeamento e retornar para a camada de view as melhores opções de compra para o produto desejado.

![RecomendacaoComponent](images/Recomendacao.png)

**Interfaces**
> * Interface Produto
> * Interface Recomendados
> * Interface Leilão
> * Interface ParticipaLeilão

As interfaces listadas são detalhadas a seguir:

## Detalhamento das Interfaces

### Interface `Produto`

> A interface produto é recebida com todos os detalhes do produto, onde o componente `Recomendação` pegará somente o `titulo` do produto desejado para iniciar o processo de leilão.

**Tópico**: 
Assina:
* `/produto/{id}`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
{
	"id": 42,
	"idCategoria": 231,
	"idFornecedor": 123,
	"idFabricante": 786,
  "titulo": "Televisao LG",
	"descricaoProduto": "Televisão Smart TV LG 50 polegadas com Wifi ",
	"precoUnitario": 2500.00,
	"imagens": [{
			"url": "imagetelevisaoFrontal.png",
			"contentDescription": "Foto da televisão de frente"
		},
		{
			"url": "imagetelevisaoLateral.png",
			"contentDescription": "Foto da televisão de lado"
		}
	]
}
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`id` | `Identificador único do produto`
`idCategoria` | `Identificador único da categoria do produto`
`idFornecedor` | `Identificador único do fornecedor, para saber quem está fornecendo`
`idFabricante` | `Identificador único do fabricante, para saber quem produziu o produto`
`titulo` | `Titulo que aparecerá para o cliente antes que ele abra os detalhes`
`descricaoProduto` | `Descreve o produto detalhadamente`
`precoUnitario` | `Valor unitário`
`imagens` | `Listagem das imagens que o produto possui`

### Interface `Recomendados`

> Interface para envio do Rankeamento  dos fornecedores com base no processamento do leilão, de acordo com o produto escolhido. Essa interface será utilizada na view para que o cliente seja capaz de escolher dentre os recomendados.

**Tópico**:  
Publica: 
* `/leilao/{id}/recomendados`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest-relatorio-recomendacao.png)

~~~json
{
  "idRecomendacao": 3234,
  "acuracia": 0.8,
  "quantidadeOfertas": 5,
  "ofertas": {
    "oferta": {
       "idProduto": 1245,
	   "idFornecedor": 3342,
	   "rankeamento": 1,
	   "valor": 33.11
    },
    "oferta": {
       "idProduto": 2332,
	   "idFornecedor": 42432,
	   "rankeamento": 2,
	   "valor": 44.11,
    },
	"oferta": {
       "idProduto": 1123,
	   "idFornecedor": 223344,
	   "rankeamento": 3,
	   "valor": 52.11,
    },
	"oferta": {
       "idProduto": 3324,
	   "idFornecedor": 122123,
	   "rankeamento": 4,
	   "valor": 55.11,
    },
	"oferta": {
       "idProduto": 1245,
	   "idFornecedor": 3342,
	   "rankeamento": 5,
	   "valor": 63.11,
    },
  }  
}
~~~

Detalhamento da mensagem JSON:

**Relatório Financeiro**
Atributo | Descrição
-------| --------
idRecomendacao | identificador da recomendacao
acuracia | acuracia da recomendação 
quantidadeOfertas | quantidade de ofertas a serem publicadas para o cliente

**Oferta**
Atributo | Descrição
-------| --------
idProduto | identificador do produto
idFornecedor | identificador do fornecedor
rankeamento | rankeamento do produto pelo algoritmo de recomendação
valor | valor da oferta

### Interface `Leilão`

> Esta interface tem como objetivo receber a resposta que vier do Fornecedor, onde ele informará se aceita ou não o leilão, e caso sim, qual a sua oferta.

**Tópico**:
Assina:
* `/leilao/{id}/{idFornecedor}/{oferta}`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
{
	"idLeilao": 1,
	"idFornecedor": 342,
	"aceitaParticipar": true,
	"lance": 200.0
}
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`idLeilao` | `Fornecer ao componente Recomendação qual a referência do leilão`
`idFornecedor` | `Fornecer ao componente Recomendação qual fornecedor respondeu sua mensagem`
`aceitaParticipar` | `Campo em que o fornecedor escolhe ou não participar do leilão`
`lance` | `Caso aceite participar, o lance oferecido para o produto em questão estará nesse campo`

### Interface `ParticipaLeilão`

> Esta interface é responsável por enviar a mensagem aos fornecedores qualificados que um novo leilão iniciou.

**Tópico**: 
Publica:
* `/leilao/{id}/{idProduto}`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
{
	"idLeilao": 432,
	"idProduto": "A9842",
	"descricaoProduto": "Televisão LG 50 polegadas"
}
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`idLeilao` | `Fornecer ao componente Fornecedor qual a referência do leilão`
`idProduto` | `Fornecer ao componente Fornecedor qual o identificador único do produto`
`descricaoProduto` | `Descreve o produto desejado pelo cliente`

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
	"operação": "Compra",
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
timestamp | timestamp com a hora exata da operação
infos |  detalhes da operação sendo auditada
md5 |  hash do arquivo de log da operação


### Interface `Auditoria`

> Interface para publicação do resultado da auditoria da operação

**Tópico**:  
Publica: 
`auditoria/{id}/dados` 

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest-auditoria.png)

~~~json
{
  "idAuditoria": 23421,
  "infos": {
	"operação": "Compra",
	"quandidadeProdutos" : 2,
	"idUsuario": 122333,
	"valorTotal": 2233.23,
  },
  "md5": "92e9f1ae8ef91056bf6ba49cd79f0869",
  "isValidado": "TRUE",
}
~~~

Detalhamento da mensagem JSON:

**Registro**
Atributo | Descrição
-------| --------
idAuditoria | identificador do arquivo de auditoria
infos |  detalhes da operação sendo auditadq
md5 |  hash do arquivo de log da operação
isValidado | resultado da auditoria (true se sucesso, false caso contrarío)

# Nível 2
## Diagrama do Nível 2

![Diagrama Nível 2](images/Diagrama_nivel_2.png)

### Detalhamento da interação de componentes

O detalhamento deve seguir um formato de acordo com o exemplo a seguir:

* O componente `Entrega Pedido Compra` assina no barramento mensagens de tópico "`pedido/+/entrega`" através da interface `Solicita Entrega`.
  * Ao receber uma mensagem de tópico "`pedido/+/entrega`", dispara o início da entrega de um conjunto de produtos.
* Os componentes `Solicita Estoque` e `Solicita Compra` se comunicam com componentes externos pelo barramento:
  * Para consultar o estoque, o componente `Solicita Estoque` publica no barramento uma mensagem de tópico "`produto/<id>/estoque/consulta`" através da interface `Consulta Estoque` e assina mensagens de tópico "`produto/<id>/estoque/status`" através da interface `Posição Estoque` que retorna a disponibilidade do produto.

Para cada componente será apresentado um documento conforme o modelo a seguir:

## Componente `<Recomendação>`

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
