# Estrutura de Arquivos e Pastas

A seguir é apresentada a estrutura de pastas do projeto:

~~~
├── README.md
│
├── images
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

### Detalhamento da interação no processo de leilão

> Componente `Recomendação`
  > * Assina no barramento mensagens de tópico "`/produto/{id}`" através da interface `Produto`, que é disparado no momento em que o usuário define qual produto deseja adquirir, iniciando assim, o processo de leilão. Quando recebe a mensagem, o componente `Recomendação` informa todos os potenciais fornecedores do produto o interesse do cliente.

  > * Publica no barramento a mensagem de tópico "`/leilao/{id}/{idProduto}`" através da interface `Participa Leilão` para que o componente `Fornecedor` que tiver interesse seja notificado do leilão.

  Após isso, a interação é por parte do fornecedor:

> Componente `Fornecedor`
  > * Assina no barramento mensagens de tópico "`/leilao/{id}/{idProduto}`" através da interface `Participa Leilão`. Quando recebe a mensagem, o fornecedor verificará os dados do produto requerido, sua disponibilidade em seu estoque e, caso haja interesse, fornecer uma oferta.

  Após o fornecedor decidir que vai participar, ocorre a seguinte interação:

  > * Publica no barramento a mensagem de tópico "`/leilao/{id}/{idFornecedor}/{oferta}`" através da interface `Leilão` onde o componente `Recomendação` poderá ter acesso a sua oferta ou recusa do leilão.

  Após a publicação da mensagem informando que o fornecedor deseja participar do leilão, ocorre a seguinte interação:

> Componente `Recomendação`
  > * Assina no barramento mensagens de tópico "`/leilao/{id}/{idFornecedor}/{oferta}`" através da interface `Participa Leilão` . Quando recebe uma mensagem, o componente `Recomendação` realiza o rankeamento dos fornecedores com base na oferta e no histórico dos mesmos.

# Descrição dos componentes:

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

![Diagrama Classes REST](images/diagrama-classes-rest-autenticacao.png)

~~~json
{
	"token": "MKT43242DSAD",
  "idUsuario": 405,
	"nome": "Carlos",
	"autenticado": true
}
~~~

Detalhamento da mensagem JSON:


Atributo | Descrição
-------| --------
`token` | `Um hash gerado que expira quando o usuário deixa de interagir com o MarketPlace por muito tempo, forçando-o a logar novamente`
`idUsuario` | `identificador único do usuário que foi autenticado (utilizado para logs, por exemplo)`
`nome` | `Primeiro nome do usuário, para ser utilizado no front-end`
`autenticado` | `Status da autenticação. Caso bem sucedida, retornará true. Caso não, retornará false (senha incorreta, e-mail inválido, usuário inexistente, etc)`


## Interface `Usuário`

> Esta é uma interface abstraída, pois a autenticação poderá ser feita tanto por fornecedor quanto por cliente.
> Portanto, criamos essa interface "pai" que será extendida pelas interfaces `Cliente` e `Fornecedor`

**Tópico**:
Assina:
* `/cliente/{id}`
* `/fornecedor/{id}`
> A assinatura do tópico correto dependerá do tipo de `Usuário` que está tentando se autenticar.

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest-usuario.png)

~~~json
{
  "id": 230,
  "email": "joao@empresa.com"
}
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`id` | `Identificador do usuário`
`email` | `Endereço de e-mail`


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

![Diagrama Classes REST](images/diagrama-classes-rest-autenticacao-cliente.png)

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
`idUsuario` | `identificador único do cliente que foi autenticado (utilizado para logs, por exemplo)`
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

![Diagrama Classes REST](images/diagrama-classes-rest-fornecedor.png)

~~~json
{
  "id": 562,
  "razaoSocial": "Diversão Brinquedos LTDA",
  "cpnj": "01009389000120",
  "inscricaoEstadual": "139007230654",
  "telefone": "1932459030",
  "site": "www.diversaobrinquedos.com",
  "endereco": {
    "logradouro": "Rua XV de Novembro, 10",
    "bairro": "Centro",
    "cidade": "Campinas",
    "uf": "SP",
    "cep": "11810970",
  },
}
~~~

Detalhamento da mensagem JSON:

**Fornecedor**
Atributo | Descrição
-------| --------
`id` | `Identificador do fornecedor`
`razaoSocial` | `Razão social do fornecedor`
`cnpj` | `Número do CNPJ do fornecedor`
`inscricaoEstadual` | `Número da inscrição estadual do fornecedor`
`telefone` | `Telefone do fornecedor`
`site` | `Endereço do site do fornecedor`
`endereco` | `Informações do endereço do fornecedor`

**Endereço**
Atributo | Descrição
-------| --------
`logradouro` | `Logradouro do fornecedor`
`bairro` | `Bairro do fornecedor`
`cidade` | `Cidade do fornecedor`
`uf` | `Estado do fornecedor`
`cep` | `CEP do fornecedor`

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

![Diagrama Classes REST](images/diagrama-classes-rest-autenticacao-fornecedor.png)

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

![Diagrama Classes REST](images/diagrama-classes-rest-leilao.png)

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

![Diagrama Classes REST](images/diagrama-classes-rest-participaleilao.png)

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

![Diagrama Classes REST](images/diagrama-classes-rest-produto.png)

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

![Diagrama Classes REST](images/diagrama-classes-rest-fornecedor.png)

~~~json
{
  "id": 562,
  "razaoSocial": "Diversão Brinquedos LTDA",
  "cpnj": "01009389000120",
  "inscricaoEstadual": "139007230654",
  "telefone": "1932459030",
  "site": "www.diversaobrinquedos.com",
  "endereco": {
    "logradouro": "Rua XV de Novembro, 10",
    "bairro": "Centro",
    "cidade": "Campinas",
    "uf": "SP",
    "cep": "11810970",
  },
}
~~~

Detalhamento da mensagem JSON:

**Fornecedor**
Atributo | Descrição
-------| --------
`id` | `Identificador do fornecedor`
`razaoSocial` | `Razão social do fornecedor`
`cnpj` | `Número do CNPJ do fornecedor`
`inscricaoEstadual` | `Número da inscrição estadual do fornecedor`
`telefone` | `Telefone do fornecedor`
`site` | `Endereço do site do fornecedor`
`endereco` | `Informações do endereço do fornecedor`

**Endereço**
Atributo | Descrição
-------| --------
`logradouro` | `Logradouro do fornecedor`
`bairro` | `Bairro do fornecedor`
`cidade` | `Cidade do fornecedor`
`uf` | `Estado do fornecedor`
`cep` | `CEP do fornecedor`

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

![Diagrama Classes REST](images/diagrama-classes-rest-produtos-escolhidos.png)

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

> Interface para envio dos detalhes do pagamento.

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

![Diagrama Classes REST](images/diagrama-classes-rest-produto.png)

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

![Diagrama Classes REST](images/diagrama-classes-rest-leilao.png)

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

![Diagrama Classes REST](images/diagrama-classes-rest-participaleilao.png)

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

> Interface para recebimento dos detalhes do pagamento.

**Tópico**:  
Assina:
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
amento | status do pagamento (pendente, recusado, finalizado)


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

> Interface para recebimento dos detalhes do pagamento.

**Tópico**:  
Assina:
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
"`/auditoria/cliente/{id}/{acao}`", "`/auditoria/fornecedor/{id}/{acao}`", "`/auditoria/produto/{id}/{acao}`", "`/auditoria/pedido/{id}/{acao}`", "`/auditoria/pagamento/{id}/{acao}`", "`/auditoria/entrega/{id}/{acao}`", "`/auditoria/recomendacao/{id}/{acao}`", "`/auditoria/autenticacao/{id}/{acao}`" e "`/auditoria/financeiro/{id}/{acao}`

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
`auditoria/{id}/resultado`

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
  "isValidado": "true",
}
~~~

Detalhamento da mensagem JSON:

**Registro**
Atributo | Descrição
-------| --------
idAuditoria | identificador do arquivo de auditoria
infos |  detalhes da operação sendo auditadq
md5 |  hash do arquivo de log da operação
isValidado | resultado da validação da auditoria (true se sucesso, false caso contrarío)

# Nível 2
## Diagrama do Nível 2

![Diagrama Nível 2](images/Diagrama_nivel_2_v1.png)

### Detalhamento da interação de componentes

* Assina no barramento mensagens de tópico "`/produto/{id}`" através da interface "`Produto`", que é disparado no momento em que o usuário define qual produto deseja adquirir, iniciando assim, o processo de leilão:
  * Ao receber uma mensagem de tópico "`/produto/{id}`", dispara o início do leilão entre os fornecedores.
* O componente "`Rankeamento Fornecedores`" recebe os dados do produto a ser adquirido e aciona o componente "`Comunica Fornecedores`";
* O componente "`Comunica Fornecedores`" recebe os dados do produto a ser adquirido e utiliza as interfaces externas para encontrar fornecedores interessados:
  * Através da interface externa "`Comunicação`" este componente publica no barramento a mensagem de tópico "`/leilao/{id}/{idProduto}`" para que o componente "`Fornecedor`" que tiver interesse seja notificado do leilão;
  * Como o componente externo assinou o barramento mensagens de tópico "`/leilao/{id}/{idFornecedor}/{oferta}`" através da interface `Participa Leilão`, ele recebe as ofertas dos fornecedores interessados através da interface requerida "`Fornecedores Interessados`";
  * Quando  recebe uma mensagem através da interface `Participa Leilão`, esse componente envia as informações da oferta para o componente "`Rankeamento Fornecedores`".
* Quando o componente "`Rankeamento Fornecedores`" recebe os dados de uma oferta vinda do componente "`Comunica Fornecedores`", ele realiza o rankeamento dos fornecedores com base na oferta e no histórico dos mesmos:
  * Através da interface requerida "`Histórico Fornecedores`" este componente solicita o histórico do fornecedor que enviou a proposta.
* O componente "`Model Recomendação`" recebe a solicitação de histórico do fornecedor através da interface provida "`Histórico Fornecedores`":
  * Este componente encaminha a solicitação recebida para o sub-componente "`Monta Histórico`";
* O componente "`Monta Histórico`" monta o histórico do fornecedor:
  * Este componente envia uma solicitação para o componente "`DataSetComponent`" através da interface "`ITableProducer`";
  * O componente "`ITableProducer`" acessa o banco de dados através dos dados da propriedade "`dataSource`" e busca os dados históricos do fornecedor;
  * O componente "`ITableProducer`" retorna os dados para o componente "`Monta Histórico`" utilizando a interface "`ITableProducer`";
  * O componente "`Monta Histórico`" normaliza os dados vindos do banco e monta o histórico do fornecedor;
  * O componente "`Monta Histórico`" retorna o histórico do fornecedor.
* O "`Model Recomendação`" retorna o histórico do usuário para o componente "`Controller Recomendação`" utilizando a interface "`Histórico Fornecedores`";
* O componente "`Controller Recomendação`" recebe o histórico do fornecedor através da interface "`Histórico Fornecedores`":
  * O componente "`Controller Recomendação`" encaminha o histórico do fornecedor para o sub-componente "`Rankeamento Fornecedores`".
* O componente "`Rankeamento Fornecedores`" recebe o histórico do fornecedor e realiza o rankeamento dos fornecedores e, consequentemente, dos produtos recomendados:
  * Esse componente utiliza os dados da oferta enviada pelo fornecedor em conjunto com o histórico para realizar o rankeamento;
  * Esse componente precisa aguardar a chegada de um número mínimo de oferta, com um limite de tempo determinado, para realizar o rankeamento.
* O componente "`Rankeamento Fornecedores`" retorna os produtos recomendados rankeados através da interface "`Produtos Recomendados`":
  * O componente "`Controller Recomendação`" encaminha o retorno deste componente para o componente "`View Recomendação`".
* O componente "`View Recomendação`" exibe os produtos recomendados de forma rankeada para o usuário:
  * O componente "`View Recomendação`" encaminha a lista de produtos recomendados para o sub-componente "`Gerenciar Recomendados`";
  * O componente "`Gerenciar Recomendados`" decodifica o retorno e repassa o resultado aos componentes "`Filtrar Recomendados`" e/ou "`Comparar Recomendados`";
  * Os componentes "`Filtrar Recomendados`" e/ou "`Comparar Recomendados`" exibem os produtos recomendados rankeados para o cliente.

## Componente `Rankeamento Fornecedores`

> <Resumo do papel do componente e serviços que ele oferece.>

![Rankeamento](https://github.com/assouza19/inf331_projetoFinal/blob/master/images/Reankeamento%20-%20Nivel2.png)

**Interfaces**
> * Ordena Fornecedores

As interfaces listadas são detalhadas a seguir:

## Detalhamento das Interfaces

### Interface `OrdenaFornecedores`

> ![Diagrama da Interface](images/diagrama-interface-ordena-fornecedores.png)

> Interface oferecida pelo componente Rankeamento Fornecedores para ordenar uma lista de fornecedores interessados em vender um determinado produto no leilão. 

Método | Objetivo
-------| --------
`ordenaFornecedores` | Interface provida que recebe um HashMap <key, HashMap <key, value>> que mapeia ID de fornecedores a uma outra estrutrura de mapeamento com as informações relevantes do fornecedor para o leilão: valor da oferta, quantidade de produtos disponiveis, localização do fornecedor. A mesma estrutura é retornada, porém com a propriedade adicional que "Ranking" que atribui a cada Fornedor uma posição com base na lógica interna do componente e no histórico desse fornecedor. 

## Componente `Comunica Fornecedores`

> <Resumo do papel do componente e serviços que ele oferece.>

![Rankeamento](https://github.com/assouza19/inf331_projetoFinal/blob/master/images/Comunica%20-%20Nivel2.png)

**Interfaces**
> * Interface Produto Selecionados
> * Interface Fornecedores Participantes
> * Interface Comunica Leilão
> * Interface Fornecedores Recomendados

As interfaces listadas são detalhadas a seguir:

## Detalhamento das Interfaces

### Interface `ProdutoSelecionados`

> ![Diagrama da Interface](images/diagrama-interface-produtos-selecionados.png)

>Interface oferecida pelo componente Comunica Fornecedores para receber os Produtos que o cliente tem interesse.

Método | Objetivo
-------| --------
`setProdutosEscolhidos` | Interface provida que recebe um Map que mapeia ID de produtos escolhidos pelo cliente a suas informações, ou seja, HashMap<IdProduto, values>. Esse Map é então processado internamente para que o componente comunique no barramento os produtos escolhidos para prosseguir o leilão (ComunicaLeilão).

### Interface `ComunicaLeilao`

> ![Diagrama da Interface](images/diagrama-interface-comunica-leilao.png)

> Interface responável por comunicar no barramento os  barramento os produto(s) escolhidos para leilão.

Método | Objetivo
-------| --------
`comunicaLeilao` | Interface provida que retorna para o barramento um Map que mapeia o ID do produto particpante do leilão e informações relevantes para os fornecedores para participaram do leilão, como: "valor máximo", "localização", etc. Retorna: HashMap<IdProduto, values>

### Interface `FornecedoresParticipantes`

> ![Diagrama da Interface](images/diagrama-interface-fornecedores-participantes.png)

> Interface provida responsável por receber as ofertas do forncedores interessados no leilão.

Método | Objetivo
-------| --------
`setOfertasFornecedores` | Interface provida que recebe um Map que mapeia ID de forncedores participantes às suas ofertas para o leião, ou seja, recebe um HashMap <IdFornecedor, HashMap <IdOferta, value>> . Esse Map é então armazendado internamente para que o componente processe ofertas e repeasse ao componente de Rankeamento os fornecedores e as ofertas dos mesmos para ordenação.

### Interface `FornecedoresRecomendados`

> ![Diagrama da Interface](images/diagrama-interface-fornecedores-recomendados.png)

> Interface provida responsável por comunicar os fornecedores recomendados baseado no rankeamento feito.

Método | Objetivo
-------| --------
`getFornecedoresRecomendados` | Interface provida que retorna para o barramento um Map com as ofertas recomendadas dos fornecedores após o rankeamento feito pelo componente de Rankeamento Fornecedores. Ou seja,retorna o mapemento do ID do fornecedores ordenados pelo ranking com suas respectivas ofertas: HashMap <IdFornecedor, HashMap <IdOferta, value>>

# Multiplas Interfaces

> Para que nossos componentes suportem quantas interfaces distintas forem necessárias, optamos por utilizar a técnica de Modularização. Essa técnica é comumente utilizada em grandes projetos por justamente abstrairem blocos do seu código em módulos separados, de forma que possam ser reutilizadas em outros projetos ou módulos.
> Por exemplo, como os componentes de `Model` e `Controller` são estruturas compartilhadas dentre as infinitas possibilidades de Views que possam surgir, o ideal é que fiquem em um módulo separado e assim, possam ser reutilizados. Isolamos o componente de banco de dados e de views em outros módulos individuais também. Para exemplificar nossa arquitetura, utilizamos como base o cenário de que nosso MarketPlace suportará 3 interfaces: Aplicações Web, Android e iOS. Veja abaixo o detalhamento da arquitetura:

## Detalhamento da arquitetura

### Módulo de Model e Controller
> Nesse módulo ocorrerá o tratamento dos dados e conterá as regras de negócio (Controller) e também toda a parte da comunicação com o `Database` (Model).
  > * Para isso, utilizaremos a tecnologia `Sprint Framework`;
> É nesse módulo também que ficará a camada de transferência de dados entre view/controller.
  > * Para a transferência de dados utilizaremos a abordagem de `RESTful Service`;

### Módulo de data base
> Nesse módulo consideramos bastante a volatilidade dos nossos dados. Por exemplo: Uma determinada categoria de produto pode conter campos descritivos  que para outras categorias não faria sentido. Como por exemplo: Na categoria `livraria`, não faria sentido se tivessem atributos como `Voltagem` ou `Instruções de uso`.
Da mesma forma, que mesmo vários produtos pertencendo a mesma categoria, pode haver necessidade de campos específicos em cada um deles. Levamos em consideração esta facilidade de adaptação, e também na performance no tráfego das nossas informações.
  > * Por esse motivo, optamos por utilizar o `MongoDB com Sharding` que, com toda certeza, nos trará a versatilidade que buscamos, a performance que prezamos e ainda por cima é baseado em `JSON`, um formato totalmente leve e de fácil compreensão, que tem se tornado cada vez mais utilizado.

### Views
> A grande questão aqui foi realmente isolar toda a parte de view em módulos separados, para que cada tecnologia pudesse implementar sua interface levando em consideração a melhor abordagem para sua tecnologia, e que pudesse se comunicar com o módulo de `Model e Controller`, ocorrendo assim, a reutilização dos mesmos. Ou seja, para cada nova tecnologia que fosse necessária implementar, seria apenas criar um novo módulo de view e injetar o módulo de `Model e Controller` para sua utilização.
Para mantermos um certo nível de organização, também foi definido que, independente da interface, todas devem enviar `Request HTTP` e a resposta será por `HTML/JSON`. Dessa forma, não teremos surpresa como, por exemplo, retornar um tipo descohecido que um dos módulos pode não saber como tratar.

  > * `Módulo WebApp`: Módulo criado para implementar as interfaces utilizando `Html`, `CSS` e `JavaScript`.
  > * `Módulo Android`: Módulo criado para implementar as interfaces utilizando: `Kotlin`;
  > * `Módulo iOS`: Módulo criado para implementar as interfaces utilizando: `swift`;

Com essa abordagem de modularização, além de melhor distribuir as responsabilidades, também deixa nossos componentes mais estruturados para receber novas interfaces que surgirem. Da mesma forma se, caso algum dia seja necessário trocar o `MongoDB`, por estar em um módulo separado, o impacto nos demais módulos seria mínimo.
>
> ![Diagrama da Interface](images/diagrama-solucao-mvc.png)

