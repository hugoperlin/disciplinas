
# Interfaces em Java

## O que é uma interface?

Uma **interface** em Java é um tipo especial de contrato. Ela define **métodos** que uma classe deve implementar, mas **não contém a implementação** desses métodos. Em outras palavras: a interface diz o que deve ser feito, e a classe decide como fazer.

---

## Por que interfaces existem?

Interfaces existem para permitir:

- **Padronização** — várias classes podem seguir o mesmo "contrato".
- **Polimorfismo** — objetos diferentes podem ser tratados de forma igual.
- **Desacoplamento** — o código fica menos dependente de implementações específicas.

---

## Um exemplo simples

Imagine um sistema que trabalha com vários tipos de arquivos: PDF, TXT, DOCX. Cada um deles pode ser salvo, mas cada um salva de uma forma diferente. Criamos uma interface:

```java
public interface Salvavel {
    void salvar();
}
```

Agora qualquer classe pode implementar o contrato:

```java
public class ArquivoPDF implements Salvavel {
    @Override
    public void salvar() {
        System.out.println("Salvando arquivo PDF...");
    }
}
```

```java
public class ArquivoTXT implements Salvavel {
    @Override
    public void salvar() {
        System.out.println("Salvando arquivo TXT...");
    }
}
```

E o código pode tratar todos de forma igual:

```java
public class Sistema {
    public void salvarArquivo(Salvavel arquivo) {
        arquivo.salvar();
    }
}
```

---

## Interfaces são úteis quando:

✔ Múltiplas classes precisam ter o mesmo comportamento  
✔ Você quer programar pensando em **contratos**, não implementações  
✔ O sistema deve aceitar novas implementações sem mudar código existente

---

## Diferença entre interface e classe abstrata

| Interface | Classe Abstrata |
|-----------|-----------------|
| Todos os métodos são abstratos por padrão (Java moderno permite métodos default) | Pode ter métodos abstratos e métodos implementados |
| Uma classe pode implementar várias interfaces | Uma classe só pode herdar de uma única classe abstrata |
| Não possui atributos de instância (somente constantes) | Pode possuir atributos |

---

## Conclusão

Interfaces ajudam a manter o código organizado, polimórfico e fácil de expandir. 

Sempre que você quiser garantir que várias classes diferentes sigam o mesmo comportamento, **uma interface é a ferramenta ideal**.
