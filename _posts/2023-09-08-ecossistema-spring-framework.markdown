---
layout: post
title:  "Ecossistema Spring Framework"
date:   2023-09-08 16:00:00 -0300
categories: Spring
---

O Spring Framework é um dos frameworks mais utilizados no mundo do desenvolvimento Java. Uma alternativa ao EJB, que na época era extremamente verboso e desnecessariamente complexo, caiu na graça dos desenvolvedores, pela sua simplicidade e robustez. Desde sua criação, foram surgindo módulos, que atendem às mais diversas necessidades, como acesso a banco de dados, web e cloud. Vamos dar uma olhada geral nos principais projetos, e entender um pouco mais como funciona esse ecossistema. 


## Spring Framework

O Spring Framework disponibiliza uma gama de ferramentas que se revelam incrivelmente valiosas durante o desenvolvimento de aplicações, incluindo bibliotecas para testes, acesso a bancos de dados, desenvolvimento web, programação reativa, integração, e até mesmo suporte para outras linguagens, como o Kotlin.
O Spring Core é núcleo do projeto, todos os módulos dependem dele. Ele oferece o container de  inversão de controle, que é responsável por realizar a injeção de dependências e o gerenciamento do ciclo de vida dos objetos. Esses objetos são nomeados Beans, e permitem o desenvolvimento de um código mais limpo e testável.

## Spring Boot

Quando não existia o Spring Boot, para fazer um “Hello World!”, era necessário configurar o projeto todo, algo que necessita muito tempo e conhecimento, depois de toda a configuração era gerado um jar que devia ser implantado em um servidor Tomcat, que fornece a implementação das apis necessárias para realizar comunicação HTTP.
Hoje com o Spring Boot, o projeto já vem configurado, seguindo as melhores práticas e o Tomcat já está contigo no JAR executável, onde é só rodar e o projeto está de pé. Ele oferece configurações e suporte para os outros módulos do Spring, melhorando bastante a experiência do desenvolvedor, e diminuindo a complexidade de configuração e deploy.

## Spring Data

O Spring Data simplifica o acesso a banco de dados, permitindo que a complexidade com a comunicação de um banco de dados SQL e NoSQL seja abstraída. É oferecido uma interface que o próprio Spring oferece uma implementação pronta, permitindo um processo de desenvolvimento mais eficiente e consistente.

## Spring Security

O Spring Security nos permite realizar o fluxo de autenticação e autorização da aplicação. Não conheço uma aplicação que não tenha necessidade de restrição de acessos de alguma forma, então é um módulo essencial em qualquer aplicação Spring.
A autenticação nos permite determinar quem pode acessar nossa aplicação enquanto a Autorização nos permite verificar se um usuário autenticado pode acessar um determinado recurso. Além disso, oferece proteção contra ataques de injeção de SQL e CSRF.


## Spring Batch

Com o crescimento do volume de dados, os sistemas hoje precisam ser capazes de processá-los em larga escala. O Spring Batch nos permite realizar processamento em lote robusto e escalável, ele fornece formas de escalar horizontalmente, tratar erros durante a execução, agendar tarefas e criar fluxos complexos de regras de negócios.

## Spring Cloud

A nuvem mudou a forma como desenvolvemos software, com isso surgiram novas necessidades. Com a adoção de microsserviços, as aplicações passaram a ter necessidade de balanceamento de carga, padrões de resiliência, descoberta  de serviços, configuração distribuída e é aí que o Spring Cloud entra, ele nos permite trabalhar com sistemas distribuídos, fornecendo abstrações para problemas complexos e reduzindo o boilerplate no código das nossas aplicações.

## Conclusão

Esses são alguns módulos que o Spring oferece, dentre muitos outros. Caso se interesse em conhecer os outros módulos, no [site  têm a lista completa](https://spring.io/projects). Podemos perceber que existem muitas ferramentas que resolvem problemas comuns nas aplicações atuais, e isso auxilia muito a vida do desenvolvedor. Muitas vezes o prazo não nos permite reinventar a pedra.
Para quem está começando, pode ficar perdido com o tanto de termos e buzzwords que nossa área tem, espero ter esclarecido do que se trata o Spring framework, o Spring boot e os módulos que fazem sinergia com ele.
