---
layout: post
title:  "Strategy design pattern utilizando o Spring Framework"
date:   2023-09-13 21:00:00 -0300
categories: Spring, Design Patterns, Java, SOLID
---

Padrões de projeto nos auxiliam a desenvolver sistemas flexíveis e expansíveis. Regras de negócio são voláteis, e o software tem que se adequar a essas mudanças, sem perder a qualidade. O que mais vemos são sistemas legados, em que as boas práticas foram ignoradas,  tornando impraticável a manutenção e o desenvolvimento de novas funcionalidades.
O Design Pattern Strategy é um padrão de projeto que pertence à categoria dos padrões comportamentais. Ele oferece uma maneira de extrair regras diferentes de um objeto para outras classes, permitindo alterar esses comportamentos de forma isolada e extensível.

Vou demonstrar duas formas que você pode implementar o Strategy na sua aplicação. Uma sem dependências externas e a segunda utilizando o Spring plugin, que nos permite implementar estratégias com regras mais rebuscadas. Ambas utilizam a injeção de dependências que o Spring nos oferece.

## Exemplo Prático

Para demonstrar vamos supor que temos um sistema de pagamentos. Oferecemos três formas de pagamento: Mercado Pago, Stripe e Pix. Além disso, queremos deixar o sistema extensível para adição, alteração e exclusão, assim não ficamos presos a nenhuma plataforma de pagamentos.
Vamos começar com a nossa classe de tipo de pagamento. Como temos os tipos de pagamento bem definidos e fixos, vamos utilizar um enum para representá-los.

PaymentType.java
```java
public enum PaymentType {


 MERCADO_PAGO, STRIPE, PIX
}
```
## Strategy utilizando o Spring Core

A implementação sem utilizar uma biblioteca externa é feita utilizando named beans. Os componentes implementam a mesma interface, que é quem vai ser usada na injeção de dependências.

PaymentService.java
```java	
public interface PaymentService {


 void process();
}
```

Agora que temos a interface que nossos serviços vão implementar, vamos criar um serviço para cada valor válido do enum PaymentType.

MercadoPagoService.java
```java
@Service("MERCADO_PAGO")
public class MercadoPagoService implements PaymentService {


 private static final Logger logger = LoggerFactory.getLogger(MercadoPagoService.class);


 @Override
 public void process() {
   logger.info("Vanilla strategy: {}", PaymentType.MERCADO_PAGO);
 }
}
```
StripeService.java
```java
@Service("STRIPE")
public class StripeService implements PaymentService {


 private static final Logger logger = LoggerFactory.getLogger(StripeService.class);


 @Override
 public void process() {
   logger.info("Vanilla strategy: {}", PaymentType.STRIPE);
 }
}

```
PixService.java
```java
@Service("PIX")
public class PixService implements PaymentService {


 private static final Logger logger = LoggerFactory.getLogger(PixService.class);


 @Override
 public void process() {
   logger.info("Vanilla strategy: {}", PaymentType.PIX);
 }
}
```

A forma que o Spring nos oferece de conseguir utilizar nossos serviços de acordo com o nome do Bean, é realizando a injeção de dependências em uma interface Map, onde a chave é o nome do Bean e o valor é o Bean. Como todos os serviços implementam a interface PaymentService, devemos utilizá-la como variável que vai ser injetada.

StrategyExampleApplication.java
```java
@SpringBootApplication
public class StrategyExampleApplication implements CommandLineRunner {


 private final Map<String, PaymentService> paymentServices;


 public StrategyExampleApplication(Map<String, PaymentService> paymentServices) {
   this.paymentServices = paymentServices;
 }


 public static void main(String[] args) {
   SpringApplication.run(StrategyExampleApplication.class, args);
 }


 @Override
 public void run(String... args) {
   Arrays.stream(PaymentType.values())
       .forEach(p -> paymentServices.get(p.name()).process());
 }
}
```
Saída:
```
Vanilla strategy: MERCADO_PAGO
Vanilla strategy: STRIPE
Vanilla strategy: PIX
```

## Strategy utilizando o Spring Plugin

O uso dos named beans para o strategy, atende casos em que não precisamos de regras adicionais além do valor ser o mesmo. O Spring Plugin nos permite determinar regras para cada estratégia, utilizando a interface Plugin, que fornece o método booleano supports. Nesse método é determinado se a estratégia vai ser utilizada ou não.
Então para aplicar a estratégia, devemos criar uma interface comum entre os serviços, que estende a interface Plugin<T> que o Spring plugin nos fornece. O genérico do Plugin é a classe que vai ser utilizada como parâmetro no método supports.

PaymentPlugin.java
```java
public interface PaymentPlugin extends Plugin<PaymentType> {


 void process();
}

```

Cada serviço deve implementar a interface PaymentPlugin que criamos.

MercadoPagoPluginService.java
```java
@Service
public class MercadoPagoPluginService implements PaymentPlugin {


 private static final Logger logger = LoggerFactory.getLogger(MercadoPagoPluginService.class);


 @Override
 public void process() {
   logger.info("Plugin strategy: {}", PaymentType.MERCADO_PAGO);
 }


 @Override
 public boolean supports(@Nullable PaymentType paymentType) {
   return paymentType == PaymentType.MERCADO_PAGO;
 }
}
```
StripePluginService.java
```java
@Service
public class StripePluginService implements PaymentPlugin {


 private static final Logger logger = LoggerFactory.getLogger(StripePluginService.class);


 @Override
 public void process() {
   logger.info("Plugin strategy: {}", PaymentType.STRIPE);
 }


 @Override
 public boolean supports(@Nullable PaymentType paymentType) {
   return paymentType == PaymentType.STRIPE;
 }
}
```
PixPluginService.java
```java
@Service
public class PixPluginService implements PaymentPlugin {


 private static final Logger logger = LoggerFactory.getLogger(PixPluginService.class);


 @Override
 public void process() {
   logger.info("Plugin strategy: {}", PaymentType.PIX);
 }


 @Override
 public boolean supports(@Nullable PaymentType paymentType) {
   return paymentType == PaymentType.PIX;
 }
}
```
O PluginRegistry, nos permite pegar um Optional do serviço de acordo com o resultado do supports. Se nenhum serviço é suportado, vai ser retornado um empty. 

StrategyExampleApplication.java
```java
@SpringBootApplication
@EnablePluginRegistries(PaymentPlugin.class)
public class StrategyExampleApplication implements CommandLineRunner {


 private final PluginRegistry<PaymentPlugin, PaymentType> paymentPlugins;


 public StrategyExampleApplication(PluginRegistry<PaymentPlugin, PaymentType> paymentPlugins) {
   this.paymentPlugins = paymentPlugins;
 }
 public static void main(String[] args) {
   SpringApplication.run(StrategyExampleApplication.class, args);
 }


 @Override
 public void run(String... args) {
   Arrays.stream(PaymentType.values())
       .forEach(p -> paymentPlugins.getPluginFor(p).ifPresent(PaymentPlugin::process));
 }
}
```
Saída:
```
Plugin strategy: MERCADO_PAGO
Plugin strategy: STRIPE
Plugin strategy: PIX
```

Pudemos ver duas formas de aplicar o Design Pattern Strategy, um sem dependência externa, mas limitado, e outro que necessitamos do Spring Plugin, entretanto mais poderoso. Existem muitos casos em que precisamos aplicar regras diferentes de acordo com uma condição, e o Strategy nos permite implementar essas regras, respeitando o Open Closed Principle do SOLID, com um código isolado para quantos casos forem necessários.

Link do projeto: https://github.com/luiguip/spring-strategy-example
