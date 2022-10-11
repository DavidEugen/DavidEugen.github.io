---
layout: post
title:  "Spring4Shell Issue"
date:   2022-04-01 03:05:53 +0900
categories: tool IDE

---

# Spring4Shell Issue

## Impacted Situation

https://spring.io/blog/2022/03/31/spring-framework-rce-early-announcement

- JDK 9 or higher

- Apache Tomcat as the Servlet container

- Packaged as a traditional WAR (in contrast to a Spring Boot executable jar)

- spring-webmvc or spring-webflux dependency

- Spring Framework versions 5.3.0 to 5.3.17, 5.2.0 to 5.2.19, and older versions

\* However, the nature of the vulnerability is more general, and there may be other ways to exploit it that have not been reported yet

## Following notes.

[https://www.lunasec.io/docs/blog/spring-rce-vulnerabilities/](https://l.facebook.com/l.php?u=https%3A%2F%2Fwww.lunasec.io%2Fdocs%2Fblog%2Fspring-rce-vulnerabilities%2F%3Ffbclid%3DIwAR065IOkWKK3Ho47r4XToc-qrqtAi-q3HD4PLaB_aQ19pVFgDkSSDG_jHa0&h=AT0To2fim1A3Xyqj_7IBdqQSiBj8X0niYWNyFFU5nnESUFiS9PW5xeF9bGbLsKZMwHxL_rBdQXOjmEmIud6vjpmLUMXRw88noG0yu4l0ivHHrIg0e84JYyxiX9I7jiaq83OU2nfZ1g&__tn__=-UK-R&c[0]=AT1WB-r49m1Vju0bgBJSoRrRc8t5KNmBGicclt4SmD9dWVDGBfYw6Fb-MqDKhjW0c2lDKLqaeQrwInSipjHvLL1txQrYamjMrxsylUm1_V__XB_b5sq1lFn4IwTAze5LBHov-GUT3iiEwFbzPh1BU9KIUQ)

