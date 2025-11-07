# Padrões DAO e Repository em Java com JDBC

------------------------------------------------------------------------

## Por que esses padrões existem?

Quando uma aplicação começa a crescer, é comum que o código SQL apareça
misturado com a lógica da aplicação. Isso gera vários problemas:

-   Código repetido para abrir conexão, executar SQL e tratar erros
-   Dificuldade de manutenção
-   Lógica de negócio misturada com detalhes técnicos
-   Trocar de banco de dados se torna quase impossível

Para resolver isso, surgem dois padrões muito usados: **DAO** e
**Repository**.

------------------------------------------------------------------------

## O que é DAO (Data Access Object)?

O DAO é um objeto especializado em acessar dados.\
Toda a responsabilidade de conversar com o sistema de **persistência de dados** escolhido para aquele projeto (arquivo texto, banco de dados relacional, banco de dados não-relacional, etc).

### O DAO faz:

-   Abrir e fechar conexões com o sistema de persistência
-   Executar comandos específicos (SQL, instruções de escrita e leitura, etc)
-   Montar objetos Java a partir da persistência.
-   Evitar repetição de código

O DAO é como um "tradutor": transforma objetos Java em conteúdo a ser armanzenado de maneira persistente. Esse conteúdo pode estar no formato de tabela de banco de dados, arquivo texto, arquivo Json, arquivo XML, etc.

Uma maneira de deixar o código-fonte flexível é utilizar o conceito de [interface](../interfaces-java.md) e definir um conjunto básico de operações que um DAO precisa executar, que geralmente é um CRUD (**C**reate, **R**ead, **U**pdate, **D**elete);


``` java
public interface ContatoDAO {
    public Resultado<Contato> salvar(Contato contato);
    public Resultado<Contato> buscar(int id);
    public Resultado<List<Contato>> listar();
    public Resultado<Contato> atualizar(int id, Contato novoContato);
    public Resultado<Boolean> deletar(int id);
}
```
Note que estamos utilizando a classe ``Resultado`` para transportar o retorno dos métodos. Isso é feito para melhorar a legibilidade do código e ao mesmo tempo agregar mais informações aos retornos. Uma explicação detalhada pode ser encontrada [aqui](resultado.md).


### Implementação de ContatoDAO com JDBC

Iremos implementar a interface ContatoDAO para realizar a comunicação com um banco de dados relacional utilizando JDBC. Desta forma, esta versão do DAO será um tradutor de Java para SQL.

```java 
public class JDBCContatoDAO implements ContatoDAO{
    private FabricaConexoes fabrica;

    public JDBCContatoDAO(FabricaConexoes fabrica) {
        this.fabrica = fabrica;
    }

    public Resultado<Contato> salvar(Contato contato){
        String sql = "INSERT INTO contatos (nome, email, telefone) VALUES (?, ?, ?)";
        
        try(Connection connection=fabrica.getConnection()){
            PreparedStatement stmt = connection.prepareStatement(sql);
            stmt.setString(1, contato.getNome());
            stmt.setString(2, contato.getEmail());
            stmt.setString(3, contato.getTelefone());
            stmt.executeUpdate();

            int id = DBUtils.getLastId(stmt);

            contato.setId(id);

            return Resultado.sucesso("Contato cadastrado",contato);

        }catch(SQLException e){
            return Resultado.erro(e.getMessage());
        }
    }

    public Resultado<List<Contato>> listar() {
        List<Contato> contatos = new ArrayList<>();
        String sql = "SELECT * FROM contatos";
        
        try(Connection connection=fabrica.getConnection()){
            PreparedStatement stmt = connection.prepareStatement(sql);
            ResultSet rs = stmt.executeQuery();

            while (rs.next()) {
                int id = rs.getInt("id");
                String nome = rs.getString("nome");
                String email = rs.getString("email");
                String telefone = rs.getString("telefone");
                
                Contato contato = new Contato(id,nome, telefone, email);

                contatos.add(contato);
            }
            return Resultado.sucesso("Contatos carregados!",contatos);
        }catch(SQLException e){
            return Resultado.erro(e.getMessage());
        }      
    }


    public Resultado<Contato> buscar(int id) {
        String sql = "SELECT * FROM contatos WHERE id=?";
        
        try(Connection connection=fabrica.getConnection()){
            PreparedStatement stmt = connection.prepareStatement(sql);
            ResultSet rs = stmt.executeQuery();
            Contato contato=null;
            while (rs.next()) {
                String nome = rs.getString("nome");
                String email = rs.getString("email");
                String telefone = rs.getString("telefone");
                
                contato = new Contato(id,nome, telefone, email);

            }
            return Resultado.sucesso("Contato carregado!",contato);
        }catch(SQLException e){
            return Resultado.erro(e.getMessage());
        }      
    }

    public Resultado<Contato> atualizar(int id, Contato novoContato){
        String sql = "UPDATE contatos SET nome=?, email=?, telefone=? WHERE id=?";
        
        try(Connection connection=fabrica.getConnection()){
            PreparedStatement stmt = connection.prepareStatement(sql);
            stmt.setString(1, novoContato.getNome());
            stmt.setString(2, novoContato.getEmail());
            stmt.setString(3, novoContato.getTelefone());
            stmt.setInt(4, id);
            
            int cont = stmt.executeUpdate();

            if(cont == 1){
                novoContato.setId(id);
                return Resultado.sucesso("Contato atualizado!",novoContato);
            }else{
                return Resultado.erro("Problema ao atualizar o contato!");
            }
            
        }catch(SQLException e){
            return Resultado.erro(e.getMessage());
        }
    }

    public Resultado<Boolean> deletar(int id){
        String sql = "UPDATE contatos SET ativo=false WHERE id=?";
        
        try(Connection connection=fabrica.getConnection()){
            PreparedStatement stmt = connection.prepareStatement(sql);
            stmt.setInt(4, id);
            int cont = stmt.executeUpdate();
            if(cont == 1){
                return Resultado.sucesso("Contato removido",true);
            }else{
                return Resultado.erro("Problema ao remover!");
            }
        }catch(SQLException e){
            return Resultado.erro(e.getMessage());
        }
    }
}

```
**Perceba**: o DAO conhece SQL e JDBC. Ele é uma camada **de
persistência** e não contém lógica de negócio.

**FabricaConexoes**: note que o JDBCContatoDAO utiliza uma classe auxiliar chamada ``FabricaConexoes``. A ideia é que o gerenciamento das conexões com o banco de dados seja feito através de uma classe específica. Uma explicação mais detalhada sobre essa classe pode ser encontrada [aqui](fabrica-conexoes.md).

------------------------------------------------------------------------

# O que é Repository?

Enquanto o DAO entende persistencia de dados, o **Repository entende a
aplicação**.

O Repository representa uma **coleção de objetos** do sistema (por
exemplo, uma coleção de Contatos).
Ele fornece métodos mais intuitivos, como:

-   `adicionarContato(String nome, String email, String telefone)`
-   `buscarContato(int id)`
-   `removerContato(int id)`

Ele **usa** um ou mais DAOs internamente, mas não conhece a lógica de persistência dos dados. Um exemplo de diferentes DAOs em um Repository é no caso de uma aplicação que pode funcionar de maneira *off-line* e *online* de forma transparente para o usuário. Quando existe conexão com a internet, os dados são salvos em um banco de dados na rede. Quando não existe conexão os dados podem ser armazenados em um banco local.

### ✅ Exemplo de Repository

``` java
public class ContatoRepository {

    private ContatoDAO contatoDAO;

    public ContatoRepository(ContatoDAO contatoDAO) {
        this.contatoDAO = contatoDAO;
    }

    public Resultado<Contato> adicionarContato(String nome, String email, String telefone) {
        
        
        Contato c = new Contato(nome,email,telefone);
        

        Resultado<Contato> resultado = contadoDAO.salvar(c);


    }

    public Resultado<List<Contato>> listarContatos() {
        
        Resultado<List<Contato>> resultado = contatoDAO.listar();
        
    }
}
```

**Agora a aplicação não lida mais com SQL**.

Neste exemplo o repositório de contatos ficou muito simples. Porém, em contextos mais complexos, o repositório pode ter uma responsabilidade maior. Um exemplo seria no caso de relações de objetos. O responsável por consultar os diferentes DAOs e carregar as dependências seria o repositório.

------------------------------------------------------------------------

# Como DAO e Repository trabalham juntos?

A arquitetura fica assim:

    Aplicação
       ↓
    Repository  ← lógica da aplicação
       ↓
    DAO         ← acesso ao banco
       ↓
    Banco de dados

-   O Repository resolve problemas de negócio
-   O DAO resolve problemas de banco
-   Cada classe tem uma responsabilidade clara

------------------------------------------------------------------------

# ✅ Diferenças resumidas

  -------------------------------------------------------------------------
  Padrão       Responsabilidade            Conhece SQL?          Nível
  ------------ --------------------------- --------------------- ----------
  DAO          Executar SQL e persistir    ✅ Sim                Baixo
               dados                                             nível

  Repository   Oferecer métodos de alto    ❌ Não                Alto nível
               nível para a aplicação                            
  -------------------------------------------------------------------------


------------------------------------------------------------------------

# Conclusão

-   **DAO** evita SQL espalhado e facilita manutenção da camada de
    dados.
-   **Repository** permite que a aplicação fale em "objetos" e não em
    SQL.
-   Os dois padrões juntos deixam o código mais organizado, limpo e
    profissional.

------------------------------------------------------------------------
