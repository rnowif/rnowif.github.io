---
layout: post
title: "L'AOP démystifié"
date: 2016-04-24 08:55:27 +0200
comments: true
categories: ["Design Pattern", "Java"]
---

L'_Aspect Oriented Programming_ est un paradigme de programmation qui permet de traiter les préoccupations transverses (ou _cross cutting concerns_ en anglais) telles que le logging, le cache ou les transactions séparément du code métier.

L'AOP repose la plupart du temps sur l'utilisation de librairies ou de frameworks qui rendent ce type de programmation assez obscur et parfois difficile à comprendre. L'objectif de cet article est d'expliquer comment cela fonctionne afin de le démystifier.

<!-- more -->

## Pourquoi l'AOP ?

Imaginons un service métier qui doit faire une réservation pour un événement. On souhaite également mesurer le temps pris par la transaction pour s'exécuter, la tracer et envoyer un mail de confirmation à l'utilisateur.

Le code pourrait alors ressembler à ceci :

```java
ReservationBookingResult void bookReservationFor(User user, Event event) {
	long start = System.currentTimeInMillis();
    if (event.isFull()) {
    	return ReservationBookingResult.fullEvent();
    }

	Reservation reservation = new Reservation(user, event);

	long end = System.currentTimeInMillis();
	LOGGER.info("Reservation took " + (end - start) + " ms to complete.");
	return ReservationBookingResult.confirmation(reservation);
}
```

## Le pattern _Proxy_

## JdkProxy, CGLib

## Définitions de l'AOP

Weaving, Pointcut, Aspect, JoinPoint

## AspectJ et Spring AOP