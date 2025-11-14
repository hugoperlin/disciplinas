## Recuperando o último ID

Geralmente as tabelas dos bancos de dados possuem uma chave primária do tipo inteiro e autoincremente. Isso significa que a cada inserção de dados feita na tabela, o próprio SGBD se encarrega de gerenciar qual será o valor da chave primárica da nova linha.

É importante para a aplicação que está utilizando o banco de dados, saber qual é a chave primária (id) do dado que foi inserido. Para isso, foi criada uma classe auxiliar que permite recuperar o último id gerado.

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class DBUtils {
    
    public static int getLastId(PreparedStatement pstm) throws SQLException{

        Connection con = pstm.getConnection();
        int id = -1;

        if(con.getMetaData().getDatabaseProductName().equals("SQLite")){
            ResultSet rs2 = con.prepareStatement("select last_insert_rowid();").executeQuery();
            rs2.next();
            id = rs2.getInt(1);
            rs2.close();
        }else{
            ResultSet rs = pstm.getGeneratedKeys();
            rs.next();
            id = rs.getInt(1);
        }
        return id;
    }
}
```

O método ``getLastId`` permite recuperar qual foi o último id gerado pelo banco de dados ao fazer um *insert*. Note que o método recebe como parâmetro um objeto da classe ``PreparedStatement`` que foi utilizado para executar o comando SQL ``INSERT``. O método suporta a recupeção do id de diferentes bancos de dados.

**IMPORTANTE**: Ao criar o objeto da classe ``PreparedStatement``, deve ser passado o parâmetro ``Statement.RETURN_GENERATED_KEYS``.

```java
PreparedStatement stmt = connection.prepareStatement(sql,Statement.RETURN_GENERATED_KEYS);
```