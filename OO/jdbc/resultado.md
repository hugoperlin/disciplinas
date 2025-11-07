# Resultados mais Informativos

A classe [`Resultado`](https://github.com/hugoperlin/results) é uma estrutura auxiliar usada para representar o resultado de uma operação.  
Ela permite que um método retorne um **sucesso** ou um **erro**, sem necessariamente lançar exceções.

Isso deixa o código mais claro, seguro e fácil de tratar, especialmente em camadas de serviço, validação ou repositório.

Os objetivos com o uso da classe são:

- Padronizar retornos de métodos
- Evitar tratamento de exceções quando não são necessárias
- Saber claramente se a operação funcionou ou falhou

## Como funciona

A classe possui dois métodos estáticos para criar resultados:

```java
public static <S> Resultado<S> sucesso(String msg, S obj)
public static <S> Resultado<S> erro(String msg)
```

Eles retornam objetos concretos:

- **Sucesso<S>** → contém mensagem e um objeto gerado pela operação
- **Erro** → contém apenas a mensagem de erro

Também existem métodos de verificação:

```java
foiSucesso()   // retorna true se o resultado for sucesso
foiErro()      // retorna true se o resultado for erro
```

E métodos para acessar o tipo correto:

```java
comoSucesso()  // converte para Sucesso<S>
comoErro()     // converte para Erro
```

---

## Exemplo de uso

### Criando e retornando resultados

```java
public Resultado<Usuario> buscarUsuario(String email){
    Usuario u = dao.buscar(email);

    if(u == null){
        return Resultado.erro("Usuário não encontrado");
    }

    return Resultado.sucesso("Usuário encontrado", u);
}
```

### Consumindo o resultado

```java
Resultado<Usuario> r = buscarUsuario("aaa@bbb.com");

if(r.foiSucesso()){
    Usuario u = r.comoSucesso().getValor();
    System.out.println("Bem vindo, " + u.getNome());
}else{
    System.out.println("Erro: " + r.getMsg());
}
```

## Usando a classe Resultado

No arquivo `pom.xml` antes da tag `<dependencies>` inclua essa seção:

```xml
  
    <repositories>
		<repository>
		    <id>jitpack.io</id>
		    <url>https://jitpack.io</url>
		</repository>
	</repositories>
```

Dentro das dependencias inclua essa dependencia:

```xml

      <dependency>
            <groupId>com.github.hugoperlin</groupId>
            <artifactId>results</artifactId>
            <version>1.0.1</version>
        </dependency>
```