# Tutorial Didático de Freemarker com Java

O **Freemarker** é uma biblioteca de templates para Java que ajuda a separar **lógica da aplicação** da **apresentação**. Com ele, você pode gerar páginas HTML, e-mails, relatórios e outros conteúdos dinâmicos de forma organizada.

Utilizaremos esta biblioteca de forma integrada ao Javalin.

---

## 1. Configuração do Freemarker

Adicione a dependência Maven:

```xml
<dependency>
  <groupId>org.freemarker</groupId>
  <artifactId>freemarker</artifactId>
  <version>2.3.34</version>
</dependency>
```

---

## 2. Template HTML simples

A biblioteca possui uma sintaxe de marcação para definir porções do arquivo html que serão substituídos pelo conteúdo gerado pela aplicação Java.

Verifique no arquivo html abaixo, existe a marcação `${nome}`, isso significa que no momento em que esse arquivo for renderizado, essa marcação será substituída pelo conteúdo enviado pelo Java.

```html
<!DOCTYPE html>
<html>
<head>
    <title>Exemplo Freemarker</title>
</head>
<body>
    <h1>Olá, ${nome}!</h1>
    <p>Bem-vindo ao Freemarker em Java.</p>
</body>
</html>
```

O código Java utilizado para renderizar o arquivo é o seguinte:

```java

Map<String, Object> data = new HashMap<>();
data.put("nome", "Maria");

ctx.render("hello.html",data);

```
Note que os dados são passados para o método render através de um objeto do tipo Map, também conhecidos como dicionários. A chave do dicionário corresponde ao valor utilizado dentro do template.

**Saída gerada:**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Exemplo Freemarker</title>
</head>
<body>
    <h1>Olá, Maria!</h1>
    <p>Bem-vindo ao Freemarker em Java.</p>
</body>
</html>
```

É possível incluir no dicionário valores mais complexos, como por exemplo, um objeto com diversos atributos. 

Veja o seguinte exemplo, onde foi declarada a classe Pessoa, e iremos renderizar os dados em um template.

Arquivo `Pessoa.java`

```java
public class Pessoa{
    private String nome;
    private int idade;

    //...
    public String getNome(){
        return nome;
    }

    public int getIdade(){
        return idade;
    }
    //...
}
```

Arquivo `dados.html`.
```html
<!DOCTYPE html>
<html>
<head>
    <title>Exemplo Freemarker - Objetos</title>
</head>
<body>
    <h1>Olá, ${pessoa.nome}!</h1>
    <p>Você tem ${pessoa.idade} de idade</p>
</body>
</html>
```

O código Java para renderizar o template.

```java

Pessoa pessoa = new Pessoa("João",20);

Map<String,Object> data = new HashMap<>();
data.put("pessoa",pessoa);

ctx.render("dados.html",data);
```

O código html gerado.
```html
<!DOCTYPE html>
<html>
<head>
    <title>Exemplo Freemarker - Objetos</title>
</head>
<body>
    <h1>Olá, João!</h1>
    <p>Você tem 20 anos de idade.</p>
</body>
</html>
```

---

## 3. Usando Condicionais

O Freemarker possui o operador condicional <#if>, que permite renderizar de acordo com alguma condição.

Arquivo `idade.html`:

```html
```html
<!DOCTYPE html>
<html>
<head>
    <title>Exemplo Freemarker - Objetos</title>
</head>
<body>
    <h1>Olá, ${pessoa.nome}!</h1>
    <p>Você tem ${pessoa.idade} de idade</p>
    <p>
    <#if pessoa.idade >= 18>
        Você é maior de idade.
    <#else>
        Você é menor de idade.
    </#if>
    </p>
</body>
</html>
```

**Saída:**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Exemplo Freemarker - Objetos</title>
</head>
<body>
    <h1>Olá, João!</h1>
    <p>Você tem 20 anos de idade.</p>
    <p>Você é maior de idade.</p>
</body>
</html>
```

---

## 4. Usando Loops

Arquivo `frutas.html.ftl`:

```html
<h2>Lista de Frutas</h2>
<ul>
<#list frutas as fruta>
    <li>${fruta}</li>
</#list>
</ul>
```

Java:

```java
List<String> frutas = Arrays.asList("Maçã", "Banana", "Laranja");
data.put("frutas", frutas);

template.process(data, out);
```

**Saída:**

```html
<h2>Lista de Frutas</h2>
<ul>
    <li>Maçã</li>
    <li>Banana</li>
    <li>Laranja</li>
</ul>
```

---

## 5. Subtemplates (Inclusão de Templates)

**`header.html.ftl`**:

```html
<header>
    <h1>Bem-vindo ao nosso site</h1>
</header>
```

**`pagina.html.ftl`**:

```html
<#include "header.html.ftl">
<main>
    <p>Conteúdo principal da página.</p>
</main>
```

Java:

```java
Template template = cfg.getTemplate("pagina.html.ftl");
template.process(data, out);
```

**Saída:**

```html
<header>
    <h1>Bem-vindo ao nosso site</h1>
</header>
<main>
    <p>Conteúdo principal da página.</p>
</main>
```

---

## 6. Fluxo Didático

1. **Java** cria dados (Map, List, objetos).  
2. **Freemarker** lê o template HTML.  
3. **Freemarker** substitui placeholders (`${...}`) pelos dados.  
4. Resultado final é HTML pronto para exibir no navegador.

---
