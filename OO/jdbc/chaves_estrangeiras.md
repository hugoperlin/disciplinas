# Chaves Estrangeiras

Quando desenvolvemos sistemas orientados a objetos geralmente existem relacionamentos entre as classes. Estas relações são representadas através da definição de atributos de outras classes dentro de um determinado objeto.

## Inserindo dados em Relação 1:N

Observe o código a seguir, que mostra a relação **1:N** entre duas classes. Note que o **N** da relação é representado por um vetor.

```java

public class Setor{
    private int id;
    private String nome;
    private String descricao;
    private Funcionario[] funcionarios;
    //...
}

public class Funcionario{
    private int id;
    private String nome;
    private String cpf;
    private Setor setor;
    //...
}
```

No exemplo acima, existe uma relação de **1:N** entre a classe ``Setor`` e a classe ``Funcionario``, ou seja, um setor possui um conjunto de funcionários, e todo funcionário pertece a um setor.

Ao representar essa relação de objetos em um banco de dados relacional, utilizamos o conceito de chave estrangeira. Assim, existem duas tabelas no banco, uma para armazenar os setores e outra para os funcionários. Na tabela de funcionários, deve existir uma coluna que representa a chave estrangeira para referenciar  uma linha na tabela de setores.

Assim, ao inserir um novo objeto da classe funcionário, deve-se informar o id do setor que aquele funcionário possui.

```java

//classe JDBCFuncionarioDAO

    public Resultado<Funcionario> salvar(Funcionario funcionario){

        String sql = "INSERT INTO funcionarios (nome, cpf, id_setor) VALUES (?, ?, ?)";
        
        try(Connection connection=fabrica.getConnection()){
            PreparedStatement stmt = connection.prepareStatement(sql,Statement.RETURN_GENERATED_KEYS);
            stmt.setString(1, funcionario.getNome());
            stmt.setString(2, funcionario.getCpf());
            stmt.setString(3, funcionario.getSetor().getId()); //aqui acessamos o id do setor
            stmt.executeUpdate();

            int id = DBUtils.getLastId(stmt);
            funcionario.setId(id);

            return Resultado.sucesso("Funcionario cadastrado",funcionario);

        }catch(SQLException e){
            return Resultado.erro(e.getMessage());
        }
    }
//...
```

## Inserindo dados em uma Relação N:M

Quando trabalhamos com banco de dados relacionais, às vezes precisamos representar situações em que uma entidade se relaciona com outra de forma múltipla. Por exemplo:

- Uma **venda** pode ter vários **produtos**
- Um **produto** pode aparecer em várias **vendas**

No banco de dados, isso é uma **relação N:M**.  
E relações N:M não podem ser representadas diretamente em tabelas.  
Por isso, precisamos criar uma tabela intermediária, como:

```
vendas_produtos
--------------------
id (PK)
id_venda (FK)
id_produto (FK)
quantidade
valor_unitario
```

Essa tabela é responsável por ligar vendas e produtos.



### Representação na Orientação a Objetos

Na programação orientada a objetos, a relação N:M **continua existindo**, mas é representada de maneiras diferentes dependendo do modelo.

**Opção 1 — Representar N:M diretamente**

```java
class Venda {
    private List<Produto> produtos;
}

class Produto {
    private List<Venda> vendas;
}
```

Essa é uma representação N:M direta: um objeto conhece vários do outro e vice-versa.

---

**Opção 2 - Quando a relação parece virar 1:N?**

Isso acontece quando criamos um **objeto intermediário** para representar o relacionamento — equivalente à tabela auxiliar do banco.

Exemplo:

```java
public class ItemVenda {
    private Produto produto;
    private int quantidade;
    private double preco;
    //...
}

public class Venda{
    private int id;
    private Cliente cliente;
    private List<ItemVenda> itens;
    //...
}
```

Agora o modelo fica assim:

- Uma `Venda` tem vários **ItemVenda** → relação **1:N**
- Um `Produto` aparece em vários **ItemVenda** → relação **1:N**

Representação conceitual:

```
Venda 1 --- N ItemVenda N --- 1 Produto
```

A relação N:M ainda existe conceitualmente, mas agora ela está “quebrada” em duas relações 1:N.

Essa abordagem é muito útil porque permite armazenar informações adicionais:

- quantidade do produto  
- preço no momento da venda  
- descontos  


O código para inserir uma nova venda utilizando seria o seguinte:

```java
//Classe JDBCVendaDAO

    public Resultado<Venda> salvar(Venda venda){

        String sql = "INSERT INTO funcionarios (id_cliente) VALUES (?)";
        try(Connection connection=fabrica.getConnection()){
            PreparedStatement stmt = connection.prepareStatement(sql,Statement.RETURN_GENERATED_KEYS);
            stmt.setString(1, venda.getCliente().getId());
            stmt.executeUpdate();

            int id = DBUtils.getLastId(stmt);
            venda.setId(id);

            return Resultado.sucesso("Venda cadastrada",venda);

        }catch(SQLException e){
            return Resultado.erro(e.getMessage());
        }
    }
```

Note que primeiro é inserida uma nova venda na tabela vendas. 

```java
//JDBCItemVendaDAO
    public Resultado<ItemVenda> salvar(ItemVenda item, int idVenda){

        String sql = "INSERT INTO vendas_produtos(id_venda,id_produto,quantidade,valor) VALUES (?,?,?,?)";
        try(Connection connection=fabrica.getConnection()){
            PreparedStatement stmt = connection.prepareStatement(sql,Statement.RETURN_GENERATED_KEYS);
            stmt.setInt(1, idVenda);
            stmt.setInt(2, item.getProduto().getId());
            stmt.setInt(3, item.getQuantidade());
            stmt.setDouble(4,item.getValor());
            stmt.executeUpdate();

            return Resultado.sucesso("Item cadastrado",item);

        }catch(SQLException e){
            return Resultado.erro(e.getMessage());
        }
    }
```


## Recuperando Dados

Quando são empregados os padrões de projeto DAO e Repository na arquitetura de um sistema orientado a objetos, deve-se tomar um certo cuidado quando se trabalha com a recuperação de dados quando existem relação entre objetos.

Um **DAO (Data Access Object)** é responsável exclusivamente por:

- Executar SQL  
- Abrir e fechar conexões  
- Mapear `ResultSet` para objetos  
- Fazer operações CRUD de **uma única entidade**

O DAO conversa com a *tabela*, não com o sistema.

Portanto:

- `ProdutoDAO` trabalha apenas com a tabela `produtos`
- `VendaDAO` trabalha apenas com a tabela `vendas`

DAO **não deve** depender de outro DAO. 

### Quem monta objetos complexos? O Repository.

O **Repository** é uma camada de mais alto nível, responsável por representar **agregados** e combinar dados vindos de vários DAOs.

Ele deve:

- Coordenar múltiplos DAOs  
- Carregar objetos completos  
- Lidar com regras de composição entre entidades  

### Estrutura recomendada

```
VendaRepository
 ├── VendaDAO (dados básicos da venda)
 └── ProdutoDAO (dados dos produtos)
```

---

### Exemplo de implementação

```java
public class VendaRepository {

    private  VendaDAO vendaDAO;
    private  ProdutoDAO produtoDAO;
    private  ItemVendaDAO itemVendaDAO;

    public Resultado<Venda> buscarVenda(int id) {
        Resultado<Venda> resultado = vendaDAO.buscar(id);

        if(resultado.foiSucesso()){
            Venda venda = resultado.comoSucesso().getObj();
            List<ItemVenda> itens = itemVendaDAO.buscarItensVenda(id);
    
            for (ItemVenda item : itens) {
                Resultado<Produto> resultadoProduto = produtoDAO.buscarProdutoItem(item.getId());
                if(resultadoProduto.foiSucesso()){

                    item.setProduto(resultadoProduto.comoSucesso().getObj());
                }else{
                    return Resultado.erro(resultaoProduto.getMessage());
                }
            }
            venda.setItens(itens);

            return resultado;
        }else{
            return Resultado.erro(resultado.getMessage());
        }

    }
}
```

Benefícios:

- Cada DAO faz apenas sua parte  
- Repository coordena várias fontes de dados  
- Arquitetura clara, modular e de fácil manutenção  
