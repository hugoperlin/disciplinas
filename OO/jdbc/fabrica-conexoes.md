
# Fábrica de Conexoes 

A classe `FabricaConexoes` é uma implementação simples de uma **fábrica de conexões JDBC**, utilizada para criar e gerenciar conexões com um banco de dados. Ela aplica o padrão de projeto **Singleton** e funciona como um mini **connection pool**.

Os objetivos da classe

- Centralizar a lógica de conexão com o banco
- Ler credenciais do `.env` e manter o código organizado
- Evitar criar conexões infinitas com o banco
- Reutilizar conexões já abertas

---

```java

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class FabricaConexoes {

    private static int MAX_CONNECTIONS = 5;
    private final String URL_DB;
    private final String DB_NAME;
    private final String USER;
    private final String PASSWORD;
    private final String CON_STRING;

    private Connection[] conexoes;

    private static FabricaConexoes instance;

    private FabricaConexoes() {
        conexoes = new Connection[MAX_CONNECTIONS];
        // carregar dados de conexão do .env
        URL_DB = Env.get("URL_DB");
        DB_NAME = Env.get("DB_NAME");
        CON_STRING = URL_DB+"/"+DB_NAME;
        USER = Env.get("DB_USER");
        PASSWORD = Env.get("DB_PASSWORD");
    }

    public static FabricaConexoes getInstance() {
        if (instance != null) {
            return instance;
        }
        instance = new FabricaConexoes();
        return instance;
    }

    public Connection getConnection() throws SQLException {

        for (int i = 0; i < MAX_CONNECTIONS; i++) {
            if (instance.conexoes[i] == null || instance.conexoes[i].isClosed()) {
                instance.conexoes[i] = DriverManager.getConnection(CON_STRING, USER, PASSWORD);
                return instance.conexoes[i];
            }
        }
        throw new SQLException("Máximo de conexões");
    }
}
```



## Padrão Singleton

O Singleton garante que exista **apenas uma instância** da classe durante toda a aplicação:

```java
private static FabricaConexoes instance;

public static FabricaConexoes getInstance() {
    if (instance != null) {
        return instance;
    }
    instance = new FabricaConexoes();
    return instance;
}
```

Isso significa que qualquer parte do sistema que precisar de conexão chamará:

```java
FabricaConexoes.getInstance().getConnection();
```

E todas usarão **a mesma fábrica**.

---

## Pool de 5 conexões

A classe armazena até 5 conexões abertas:

```java
private static int MAX_CONNECTIONS = 5;
private Connection[] conexoes;
```

Quando alguém pede uma conexão:

1. ela verifica se existe alguma vaga
2. se a conexão antiga está fechada, ela é substituída
3. se todas as 5 estiverem ocupadas, lança exceção

```java
public Connection getConnection() throws SQLException {
    for (int i = 0; i < MAX_CONNECTIONS; i++) {
        if (instance.conexoes[i] == null || instance.conexoes[i].isClosed()) {
            instance.conexoes[i] = DriverManager.getConnection(CON_STRING, USER, PASSWORD);
            return instance.conexoes[i];
        }
    }
    throw new SQLException("Máximo de conexões");
}
```

---

## Uso do `.env`

As credenciais do banco são lidas sem ficar expostas no código:

```java
URL_DB = Env.get("URL_DB");
DB_NAME = Env.get("DB_NAME");
USER = Env.get("DB_USER");
PASSWORD = Env.get("DB_PASSWORD");
```

Isso melhora organização e segurança.

---


