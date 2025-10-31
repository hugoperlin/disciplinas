# Armazenamento Permanente - Banco de Dados  

## Introdução  

O armazenamento permanente de dados é um elemento essencial no desenvolvimento de sistemas. Usualmente utiliza-se **bancos de dados**, que organizam os dados de forma estruturada, permitindo o acesso, consulta e atualização conforme as necessidades da aplicação.  

Um **banco de dados** é, portanto, uma organização estruturada de informações que permite armazenar e acessar dados de maneira eficiente. O armazenamento pode ser feito de diversas formas, seja por meio de arquivos ou através de tabelas e relações em sistemas gerenciadores de banco de dados (SGBDs).  

Os **Sistemas Gerenciadores de Banco de Dados (SGBDs)** são responsáveis por implementar o armazenamento e o acesso aos dados, oferecendo mecanismos de controle, segurança e consistência. Entre os SGBDs mais comuns, destacam-se **MySQL, MariaDB, Oracle, PostgreSQL e SQLite**.

---

## JDBC – Java DataBase Connectivity  

Para que sistemas desenvolvidos em Java possam se comunicar com bancos de dados, utiliza-se a **API JDBC (Java DataBase Connectivity)**. Essa API fornece um modelo uniforme de conexão entre o sistema e diferentes SGBDs, de modo que a aplicação possa manipular os dados sem depender diretamente do tipo de SGBD utilizado.  

O JDBC permite que a relação entre o sistema e o tipo de banco de dados seja transparente, bastando apenas utilizar o driver apropriado para o SGBD escolhido. Cada sistema gerenciador implementa seu próprio driver JDBC, como **MySQL, SQLite, Firebird, SQLServer, PostgreSQL**, entre outros.  

---

## Tipos de Driver JDBC  

Existem diferentes tipos de drivers JDBC. O mais comum e recomendado é o **Driver Nativo (Type 4)**. Esse tipo converte as chamadas JDBC diretamente no protocolo do banco de dados, sendo totalmente implementado em Java e independente de plataforma.  

Os drivers nativos são geralmente desenvolvidos pelos próprios fornecedores dos bancos de dados e são os mais flexíveis e eficientes, sendo amplamente utilizados no mercado.  

Para a conexão com o MySQL/MariaDB pode-se utilizar o driver [Conector-J](https://mvnrepository.com/artifact/com.mysql/mysql-connector-j).

Para a conexão com o SGBD SQLite, pode-se utilizar o driver [SQLite JDBC](https://mvnrepository.com/artifact/org.xerial/sqlite-jdbc).

---

## Incluindo a Dependência (Exemplo MySQL)  

Em projetos Maven, é necessário incluir a dependência do driver JDBC correspondente ao banco de dados. No caso do MySQL/MariaDB, adiciona-se ao arquivo `pom.xml` o seguinte trecho:

```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>9.5.0</version>
</dependency>
```

---

## Criando a Tabela  

Antes de interagir com o banco via JDBC, é preciso garantir que o banco de dados e suas tabelas existam. Abaixo está um exemplo de criação de tabela chamada `oo_contatos`, para armazenar os dados relativos da aplicação de lista de contatos.

```sql
CREATE TABLE IF NOT EXISTS oo_contatos(
    id INT NOT NULL AUTO_INCREMENT,
    nome VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL,
    telefone VARCHAR(255) NOT NULL,
    PRIMARY KEY (id)
);
```

Acesse um banco de dados MySQL e execute a DDL acima, para que seja possível criar a tabela de contatos.

---

## Pacote `java.sql`  

A API JDBC está contida no pacote `java.sql`, que fornece classes e interfaces para acesso e processamento de dados em SGBDs relacionais.  

As principais classes e interfaces são:  

- **DriverManager**: responsável por gerenciar os drivers e estabelecer conexões com o banco.  
- **Connection**: mantém uma conexão ativa com um banco de dados.   
- **PreparedStatement**: executa instruções SQL com parâmetros, prevenindo ataques de SQL Injection.  
- **ResultSet**: armazena e gerencia o resultado de consultas SQL.  

---

## Passos para uma Consulta  

O uso do JDBC geralmente segue as seguintes etapas:  

1. Estabelecer uma conexão (`Connection`).  
2. Criar um `PreparedStatement`.  
3. Executar a instrução SQL desejada.  
4. Processar os resultados, caso haja retorno (`ResultSet`).  
5. Fechar a conexão.  

---

## 1. Estabelecer uma conexão  

A classe `DriverManager` é responsável por gerenciar os drivers JDBC e estabelecer conexões com bancos de dados. 
Invocamos o método estático `getConnection` para que seja criado um novo objeto do tipo `Connection`. 

Os parâmetros a serem utilizados dependerá do SGBD empregado. Geralmente são enviados `url`, que indica a localização do banco de dados, que sempre começa com jdbc seguido do nome do SGDB e o endereço do servidor; e as credenciais de acesso `user` e `password`. 

Exemplo de código para abrir uma conexão:

```java
Connection conn = DriverManager.getConnection(
    "jdbc:mysql://URL/BANCO", "USUARIO", "SENHA"
);
```

O método `getConnection` retorna um objeto que implementa a interface `Connection`. A partir desse objeto, é possível realizar todas as operações sobre o banco de dados.

A interface `Connection` representa uma conexão ativa com o banco de dados. Ela permite enviar e receber instruções SQL e também gerenciar transações.  

Principais métodos utilizados:  
- **commit()**: confirma todas as alterações realizadas na transação atual.  
- **rollback()**: desfaz as alterações não confirmadas.  
- **close()**: encerra a conexão e libera os recursos utilizados.  

Além desses, a interface oferece métodos para execução de consultas, chamadas de Stored Procedures e controle de propriedades da conexão.  

### Gerenciamento de Credenciais

Escrever as credenciais de acesso ao banco de dados dentro do código-fonte é considerado um grave erro, pois abre possibilidades para diferentes vulnerabilidades de segurança.

Uma boa prática, é armanzenar as credenciais em um arquivo de variáveis de ambiente, geralmente chamado de `.env`. Esse tipo de arquivo possui uma estrutura de `CHAVE=VALOR`, geralmente em texto, onde cada linha define o valor de uma variável diferente.

No caso de um banco de dados, geralmente temos as seguintes variáveis:

```env
URL_DB=LINK PARA O BANCO
DB_NAME=NOME DO BANCO
DB_USER=USUARIO DO BANCO
DB_PASSWORD=SENHA DO BANCO
```

Para manipular arquivos `.env` no Java, uma das possibilidades é utilizar a seguinte biblioteca:

```xml
        <dependency>
            <groupId>io.github.cdimascio</groupId>
            <artifactId>dotenv-java</artifactId>
            <version>2.2.4</version>
        </dependency>
```

Para facilitar a leitura das variáveis, a sugestão é definir uma classe que seja responsável pelo acesso aos dados.

Arquivo `Env.java`.
```java
import io.github.cdimascio.dotenv.Dotenv;

public class Env {
    
    private static Dotenv dotenv;
    private Env(){}
    public static String get(String key){
        if(dotenv == null){
            dotenv = Dotenv.load();
        }
        return dotenv.get(key);
    }
 
}

```


---

## 2. Criar um `PreparedStatement`. 

A interface `PreparedStatement` permite a execução de instruções SQL parametrizadas, o que traz mais segurança e eficiência. O uso de parâmetros evita a concatenação direta de strings e protege contra ataques de [SQL Injection](https://www.w3schools.com/sql/sql_injection.asp).  

Exemplo de uso:

```java
String sql = "INSERT INTO contatos(nome,email,telefone) VALUES (?,?,?)";
PreparedStatement pstmt = conn.prepareStatement(sql);
pstmt.setString(1, contato.getNome());
pstmt.setString(2, contato.getEmail());
pstmt.setString(3, contato.getTelefone());
```

Os parâmetros na instrução SQL são representados pelo símbolo `?`. Cada um deles é configurado por meio dos métodos `setTipo(posição, valor)`, como `setString`, `setInt`, etc.  



## 3. Executar a instrução SQL desejada.

Para executar a instrução SQL no banco de dados, temos duas possibilidades.

O método `executeUpdate()` é utilizado para comandos `INSERT`, `UPDATE` e `DELETE`, ou seja, comandos que geralmente geram algum tipo de modificação no banco de dados. Ele retorna o número de linhas que foram afetadas pela instrução SQL executada.

```java
int linhas = pstm.executeUpdate();
```

O método `executeQuery()` é utilizado para o comando `SELECT`, ou seja, o comando de seleção que irá retornar linhas do banco de dados. Ele retorna um objeto do tipo `ResultSet`, que permite manipular os dados selecionados.

```java
ResultSet result = pstm.executeQuery();
```

---

### 4. Processar os resultados, caso haja retorno (`ResultSet`). 

A interface `ResultSet` é utilizada para armazenar e percorrer os dados retornados por uma consulta SQL. O resultado é estruturado em forma de tabela, onde o cursor se movimenta linha a linha.  

Principais métodos:  
- **next()**: move o cursor para a próxima linha.  
- **close()**: libera os recursos utilizados.  
- **getTipo(String columnName)**: retorna o valor da coluna especificada, onde `Tipo` pode ser `Int`, `String`, `Double`, etc.  

Exemplo de uso:

```java
ResultSet rs = stmt.executeQuery("SELECT * FROM contatos");
ArrayList<Contato> lista = new ArrayList<>();

while (rs.next()) {
    int id = rs.getInt("id");
    String nome = rs.getString("nome");
    String email = rs.getString("email");
    String telefone = rs.getString("telefone");
    Contato contato = new Contato(id, nome, email, telefone);
    lista.add(contato);
}
```

Com os métodos `getInt()` e `getString()`, recuperam-se os valores das colunas de cada linha do resultado.

---

## Conclusão  

O uso do JDBC é fundamental para integrar aplicações Java com bancos de dados relacionais. Ele fornece uma API padronizada que abstrai as diferenças entre os diversos SGBDs, permitindo ao desenvolvedor criar aplicações que armazenam, manipulam e consultam dados de forma segura e eficiente.  

Ao dominar as classes `Connection`, `PreparedStatement` e `ResultSet`, é possível implementar todas as operações de persistência necessárias para um sistema robusto e escalável.  
