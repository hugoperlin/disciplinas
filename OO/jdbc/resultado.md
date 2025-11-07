# Resultados mais Informativos

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