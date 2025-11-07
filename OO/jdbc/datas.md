# Trabalhando com Datas

Uma forma para trabalhar com registro de tempo em sistema é utilizar as classes [`LocalDate`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/LocalDate.html), para representar uma data, e [`LocalDateTime`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/LocalDateTime.html), para representar uma data e hora.


```java
public class Compromisso {
    private int id;
    private LocalDateTime dataHora;
    private String descricao;
}
```

Para inserir os valores de data e hora em um banco de dados utilizando JDBC, utilize a classe `TimeStamp` do pacote `java.sql`, que permite converter um objeto do tipo `LocalDateTime` ou `LocalDate` para `TimeStamp`. 

```java

public String cadastrarCompromisso(LocalDateTime data, String descricao){
        String sql = "INSERT INTO oo_compromissos(dataHora,descricao) VALUES (?,?)";
           //... 
            PreparedStatement pstm = con.prepareStatement(sql);

            pstm.setTimestamp(1, Timestamp.valueOf(data));
 
          //...
```

Para recuperar os valores de data e hora do banco, utilize o método auxiliar `toLocalDateTime` (ou `toLocalDate`) da classe `TimeStamp`.

```java

public List<Compromisso> listarCompromissos(){

        String sql = "SELECT * FROM oo_compromissos";
        //...

        LocalDateTime dataHora = result.getTimestamp("dataHora").toLocalDateTime();
        //...
```

Para manipular data e hora em um banco de dados, sugere-se que a coluna seja definida com o tipo `TimeStamp`.


