
# Tratamento de Exceções em Java

O tratamento de exceções é uma parte essencial da programação em Java.  
Ele permite que o programa **lide com erros de forma controlada**, sem travar ou encerrar inesperadamente.

Imagine que você está dividindo dois números. E se o divisor for zero?  
Ou se o usuário digitar algo errado em vez de um número?  
Essas situações são chamadas de **exceções** — eventos inesperados que podem acontecer durante a execução do programa.

---

## O que é uma exceção?

Uma **exceção** é um problema que ocorre quando o programa não consegue executar uma operação normalmente.

Quando isso acontece, o Java “lança” um objeto de exceção.  
Se esse objeto não for tratado, o programa **para de funcionar** e exibe uma mensagem de erro.

**Em resumo:** uma exceção é um *aviso* do Java dizendo:  
> “Algo deu errado, o que você quer que eu faça com isso?”

---

## Tipos de exceções

As exceções em Java são divididas em dois grandes grupos:

1. **Checked Exceptions (Verificadas)**  
   - O compilador *obriga* você a tratar ou declarar.  
   - Exemplo: ler um arquivo que pode não existir (`IOException`).

2. **Unchecked Exceptions (Não verificadas)**  
   - O compilador *não exige* tratamento.  
   - Exemplo: divisão por zero (`ArithmeticException`), erro de conversão de tipos (`NumberFormatException`).

As exceções não verificadas geralmente indicam **erros de lógica**,  
enquanto as verificadas indicam **situações esperadas, mas incertas** (como falha de rede ou arquivo ausente).

---

## Tratando exceções com `try-catch`

O bloco `try-catch` é o principal mecanismo de tratamento de exceções em Java.

```java
try {
    int resultado = 10 / 0; // Isso causará uma exceção!
    System.out.println("Resultado: " + resultado);
} catch (ArithmeticException e) {
    System.out.println("Erro: divisão por zero não é permitida.");
}
```

**Como funciona:**

- O código dentro de `try` é executado normalmente.  
- Se ocorrer uma exceção, o Java **pula para o bloco `catch`** correspondente.  
- O programa continua rodando, sem travar.

Você pode ter **vários blocos `catch`** para tratar diferentes tipos de exceção.

---

## O bloco `finally`

O bloco `finally` é **opcional**, mas muito útil.  
Ele é executado **sempre**, independentemente de ter ocorrido uma exceção ou não.

```java
try {
    int[] numeros = {1, 2, 3};
    System.out.println(numeros[3]); // Índice inexistente!
} catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("Erro: índice fora dos limites do array.");
} finally {
    System.out.println("Encerrando o programa...");
}
```

**Uso típico:** fechar arquivos, liberar conexões de banco de dados ou recursos externos.

---

## Lançando exceções com `throws`

Em alguns casos, o método pode **lançar uma exceção** em vez de tratá-la diretamente.  
Isso é feito com a palavra-chave `throws`.

```java
public void lerArquivo(String caminho) throws IOException {
    FileReader arquivo = new FileReader(caminho);
    // ...
}
```

Aqui, o método **informa ao chamador** que ele deve lidar com a possível exceção (`IOException`).

---

## Boas práticas no tratamento de exceções

1. **Trate apenas o que você puder resolver.**  
   Evite capturar exceções sem saber o que fazer com elas.

2. **Não ignore exceções.**  
   Um `catch (Exception e) {}` vazio é um mau sinal.  
   Pelo menos, registre o erro com `e.printStackTrace()` ou uma mensagem clara.

3. **Use mensagens explicativas.**  
   Ajude quem for depurar o código no futuro.

4. **Ordem dos blocos `catch`:**  
   Do mais específico para o mais genérico.

---

## Exemplos práticos

### Exemplo 1 – Divisão segura

```java
import java.util.Scanner;

public class DivisaoSegura {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        try {
            System.out.print("Digite o numerador: ");
            int a = sc.nextInt();
            System.out.print("Digite o denominador: ");
            int b = sc.nextInt();

            int resultado = a / b;
            System.out.println("Resultado: " + resultado);
        } catch (ArithmeticException e) {
            System.out.println("Erro: não é possível dividir por zero!");
        } catch (Exception e) {
            System.out.println("Erro inesperado: " + e.getMessage());
        } finally {
            sc.close();
            System.out.println("Encerrando programa...");
        }
    }
}
```

---

### Exemplo 2 – Conversão de texto para número

```java
public class Conversao {
    public static void main(String[] args) {
        String valor = "abc";

        try {
            int numero = Integer.parseInt(valor);
            System.out.println("Número: " + numero);
        } catch (NumberFormatException e) {
            System.out.println("Erro: valor informado não é um número válido.");
        }
    }
}
```

---

## Conclusão

Tratar exceções é como **colocar capacete na programação**:  
você evita que pequenos acidentes causem grandes estragos.

Com `try`, `catch`, `finally` e `throws`, o Java oferece ferramentas poderosas para deixar seu código **seguro, legível e confiável**.


