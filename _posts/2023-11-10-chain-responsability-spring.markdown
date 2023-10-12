---
layout: post
title:  "Design Pattern Chain of Responsibility utilizando o Spring Framework"
date:   2023-11-10 21:00:00 -0300
categories: Spring, Design Patterns, Java, SOLID

---

Chain of Responsibility é um Design Pattern que pertence à categoria de padrões comportamentais. Ele nos oferece uma forma de tratar regras sequenciais em que caso se encaixe em uma regra, o fluxo é interrompido.

## Aplicação

Um bom exemplo é o envio de mensagem para um cliente.

Se a mensagem é importante, é necessário ligar para o cliente;
Se a mensagem possui menos de 500 caracteres, ela é enviada por WhatsApp;
Se não se encaixa nas regras anteriores, é enviada por email.

Quando pensamos na implementação, cada classe da chain tem que ser capaz de verificar se ela é responsável por processar o dado e deve ter a lógica que vai ser implementada.
O Java já nos oferece interfaces para implementarmos as regras do nosso Chain of Responsibility.

* Predicate<T>: É uma interface funcional em que recebe um argumento genérico T e retorna um booleano. Justamente o que precisamos para validar se o handler deve atuar.
* Consumer<T>: Interface funcional que recebe um argumento genérico e não tem retorno, o que nos atende. O handler vai logar a atividade que está realizando e não vai retornar nada.

Adicionalmente vamos implementar uma interface que o Spring nos oferece para ordenar nossos Beans, é o que utilizaremos para determinar a ordem dos nossos handlers.

* Ordered: oferece o método getOrder que retorna um int. Com os valores retornados, o Spring nos permite injetar uma lista ordenada de acordo com os valores desse método.

## Implementação

Começamos nossa implementação definindo a interface do nosso Handler.

Handler.java
```java
import java.util.function.Consumer;
import java.util.function.Predicate;
import org.springframework.core.Ordered;


public interface Handler extends Consumer<Message>, Predicate<Message>, Ordered {
}
```

Agora implementamos um Handler para cada regra de negócio. A ordem que definimos pode ser configurada no nosso application.properties.

application.properties
```java
text-handler.important.order: 1
text-handler.short.order: 2
```
ImportantTextHandler.java
```java
@Component
public class ImportantTextHandler implements Handler{

    private static final Logger log = LoggerFactory.getLogger(ImportantTextHandler.class);

    @Value("${text-handler.important.order}")
    private int order;

    @Override
    public void accept(Message message) {
        log.info("Call client. message: {}", message.text());
    }

    @Override
    public boolean test(Message message) {
        return message.important();
    }

    @Override
    public int getOrder() {
        return order;
    }
}
```

ShortTextHandler.java
```java
@Component
public class ShortTextHandler implements Handler{

    private static final Logger log = LoggerFactory.getLogger(ShortTextHandler.class);

    @Value("${text-handler.short.order}")
    private int order;


    @Override
    public void accept(Message message) {
        log.info("Send a message via WhatsApp. message: {}", message.text());
    }


    @Override
    public boolean test(Message message) {
        return message.text().length() < 500;
    }


    @Override
    public int getOrder() {
        return order;
    }
}
```
DefaultTextHandler.java
```java
@Component
public class DefaultTextHandler implements Handler {

    private static final Logger log = LoggerFactory.getLogger(DefaultTextHandler.class);

    @Override
    public void accept(Message message) {
        log.info("Send message via email. message: {}", message.text());
    }

    @Override
    public boolean test(Message message) {
        return true;
    }

    @Override
    public int getOrder() {
        return LOWEST_PRECEDENCE;
    }
}
```
Para implementar a corrente dos handlers vamos utilizar umas lista onde a ordem, é a ordem de prioridade dos handlers. Como estamos utilizando a interface Ordered que o Spring nos oferece, basta realizarmos a injeção de dependências com uma Lista da interface Handler.

TextHandlerChain.java
```java
@Component
public class TextHandlerChain {

    private final List<Handler> textHandlers;

    public TextHandlerChain(List<Handler> textHandlers) {
        this.textHandlers = textHandlers;
    }

    public void send(Message message) {
        textHandlers.stream()
        .filter(h -> h.test(message))
        .findFirst()
        .ifPresent(h -> h.accept(message));
    }
}
```

Na nossa application injetamos o handler e implementamos a interface CommandLineRunner que nos permite adicionar lógica para rodar quando a aplicação começar.
Crio três mensagens, uma que se encaixa em cada handler e aplicamos nosso chain of responsiblity.

```java
@SpringBootApplication
public class ChainOfResponsibilityExampleApplication implements CommandLineRunner {

    private final TextHandlerChain textHandlerChain;

    public ChainOfResponsibilityExampleApplication(TextHandlerChain textHandlerChain) {
        this.textHandlerChain = textHandlerChain;
    }

    public static void main(String[] args) {
        SpringApplication.run(ChainOfResponsibilityExampleApplication.class, args);
    }

    @Override
    public void run(String... args) {
        var messages = List.of(
        new Message("Important text", true),
        new Message("Short text", false),
        new Message("Long text!".repeat(50), false)
        );
        messages.forEach(textHandlerChain::send);
    }
}
```

E nos logs da aplicação, podemos ver que os handlers foram aplicados de acordo com as condições que definimos.

```
2023-10-08T16:07:47.514-03:00  INFO 23008 --- [  restartedMain] c.g.l.c.freight.ImportantTextHandler     : Call client. message: Important text
2023-10-08T16:07:47.515-03:00  INFO 23008 --- [  restartedMain] c.g.l.c.freight.ShortTextHandler         : Send a message via WhatsApp. message: Short text
2023-10-08T16:07:47.515-03:00  INFO 23008 --- [  restartedMain] c.g.l.c.freight.DefaultTextHandler       : Send message via email. message: Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!Long text!
```
Pudemos observar que o Spring nos oferece várias ferramentas para utilizarmos sua injeção de dependências de acordo com nossa necessidade, o que é bem útil nos prazos curtos e nas regras de negócio complexas que lidamos.
O código fonte se encontra no meu [github](https://github.com/luiguip/chain-of-responsibility-example). 
