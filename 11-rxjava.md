# RxJava

* [Reaktivní programování](http://reactivex.io/)
* [Repozitář projektu](https://github.com/ReactiveX/RxJava)

  Rozšíření umožňující deklarativní reaktivní programování v Javě. Celé je postaveno na návrhovém vzoru [Observer](https://en.wikipedia.org/wiki/Observer_pattern) od GoF. Existují ekvivalenty pro jiné jazyky - C\#, JavaScript...

## Vše je Observable

Reaktivní programování znamená vytvářet proudy událostí. Observable vytváří události - na požádání, při splnění určitých podmínek, nebo třeba vůbec. Protipólem je Subscriber - konzument - který se váže k Observable a odebírá její události. Každý Subscriber obsahuje trojici metod _onNext_, _onComplete_ a _onError_. Metoda _onNext_ může být zavolána alespoň 0x a je následována právě 1 voláním _onComplete_ nebo _onError_. Tím život streamu končí.

```java
Observable<String> myObservable = Observable.create(
    new Observable.OnSubscribe<String>() {
        @Override
        public void call(Subscriber<? super String> sub) {
            sub.onNext("Hello, world!");
            sub.onCompleted();
        }
    }
);
```

Tento kód předá Subscriberovi po subscribnutí řetězec "Hello, world!" a skončí. Vytvoříme Subscribera, který bude umět řetězec zpracovat:

```java
Subscriber<String> mySubscriber = new Subscriber<String>() {
    @Override
    public void onNext(String s) { System.out.println(s); }

    @Override
    public void onCompleted() { }

    @Override
    public void onError(Throwable e) { }
};
```

Vše spojíme:

```java
myObservable.subscribe(mySubscriber);
```

Pro vztváření Observable axistuje řada factory metod. Nejsnažší k použití je.

```java
Observable<String> myObservable =
    Observable.just("Hello, world!");
```

Subscriber je tvořen 3 akcemi - ekvivalenty k onNext, onComplete, onError. Toho se dá využít a zadefinovat každou akci zvlášť:

```java
myObservable.subscribe(onNextAction, onErrorAction, onCompleteAction);
```

Nebo použít jen tu, co se nám hodí:

```java
Action1<String> onNextAction = new Action1<String>() {
    @Override
    public void call(String s) {
        System.out.println(s);
    }
};

myObservable.subscribe(onNextAction);
```

> V praxi je ale vždy nutné reagovat na onError, byť bychom měli na daném místě použít jen log. Když cokoli selže stream se routuje do onError metody, tak když není implementována, skončí program s výjimkou.

Když zrušíme proměnné, jde daný kód zjednodušit následovně:

```java
Observable.just("Hello, world!")
    .subscribe(new Action1<String>() {
        @Override
        public void call(String s) {
              System.out.println(s);
        }
    });
```

A to vše jde pomocí lambda výrazu zkrátit na:

```java
Observable.just("Hello, world!")
    .subscribe(s -> System.out.println(s));
```

Tady vše teprve začíná. Streamy jdou pomocí různých operátorů modifikovat a kombinovat.

```java
Observable.just("Hello")
    .map(s -> s + ", world!")
    .subscribe(s -> System.out.println(s));
```

```java
Observable.just("Hello")
    .map(s -> s + ", ")
    .map(s -> s + "world!")
    .subscribe(s -> System.out.println(s));
```

```java
Observable.just("Hello, world!")
    .map(s -> s.hashCode())
    .subscribe(i -> System.out.println(Integer.toString(i)));
```

Další způsob jak vytvořit Observable:

```java
Observable.from("url1", "url2", "url3")
    .subscribe(url -> System.out.println(url)); //Co vypíše tento kód?
```

Observable se dají kombinovat:

```java
Observable.from("url1", "url2", "url3")
    .flatMap(url -> getTitle(url)) //getTitle je dalsi Observable vracejici titulek Url
    .subscribe(title -> System.out.println(title));
```

A zase můžeme na stream aplikovat některé z [operátorů](https://github.com/ReactiveX/RxJava/wiki/Alphabetical-List-of-Observable-Operators).

```java
    Observable.from("url1", "url2", "url3")
    .flatMap(url -> getTitle(url))
    .filter(title -> title != null)
    .take(5)
    .doOnNext(title -> saveTitle(title))
    .subscribe(title -> System.out.println(title));
```

## Scheduler

Plánovače představují mechanizmus pro určení typu vlákna, které bude stream obsluhovat.

```java
myObservableServices.retrieveImage(url)
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(bitmap -> myImageView.setImageBitmap(bitmap));
```

## Subscription

Spojení mezi Observable a Subscriberem. Slouží k držení reference na stream a jeho případné ukončení. Nejčastěji se používá s CompositeSubscription.

### Užitečné pro Android:

* [RxBindings](https://github.com/JakeWharton/RxBinding)
* [RetroLambda](https://github.com/evant/gradle-retrolambda)
* [SQLBrite](https://github.com/square/sqlbrite)

