# RxJava

* [Reaktivní programování](http://reactivex.io/)
* [Repozitář projektu](https://github.com/ReactiveX/RxJava)
Rozšíření umožňující deklarativní reaktivní programování v Javě. Celé je postaveno na návrhovém vzoru [Observer](https://en.wikipedia.org/wiki/Observer_pattern) od GoF. Existují ekvivalenty pro jiné jazyky - C#, JavaScript...

## Vše je Observable
Reaktivní programování znamená vytvářet proudy událostí. Observable vytváří události - na požádání, při splnění určitých podmínek, nebo třeba vůbec. Protipólem je Subscriber - konzument - který se váže k Observable a odebírá její události. Každý Subscriber obsahuje trojici metod *onNext*, *onComplete* a *onError*. Metoda *onNext* může být zavolána alespoň 0x a je následována právě 1 voláním *onComplete* nebo *onError*. Tím život streamu končí.


