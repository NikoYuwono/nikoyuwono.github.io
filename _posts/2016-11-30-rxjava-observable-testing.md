---
layout: post
title: Testing RxJava Observable with TestSubscriber
description: Testing RxJava code can be intimidating because all operations in RxJava is asynchronous by default. Let's check several ways how we can test it.
---

RxJava gained so much popularity this year, I think it's kinda obvious because it helps us to make our code composable. I won't talk much about how to use it but, if you haven't heard/tried it yet I recommend you to check it out [here](https://github.com/ReactiveX/RxJava) or you can check really nice tutorial by Dan Lew [here](http://blog.danlew.net/2014/09/15/grokking-rxjava-part-1/) (For RxJava 2.0 you can check vogella's [tutorial](http://www.vogella.com/tutorials/RxJava/article.html)).

Testing RxJava code can be intimidating because all operations in RxJava is asynchronous by default. Let's check several ways how we can test it.

## Naive way

We can declare a global variable and set it inside the subscriber and then assert that the variable is not null.  

```
Definition wordDefinition;

@Test
public void searchWordDefinition() {
    final Definition definition = new Definition(FAKE_TAGS, FAKE_WORDS);
    when(wordsRemoteDataSource.getWordDefinition(TEST_QUERY)).thenReturn(Observable.just(definition));
    
    wordsRepository.getWordDefinition(TEST_QUERY).subscribe(definition -> {
        wordDefinition = definition;
    });
    assertNotNull(wordDefinition);
}
```

This will works because it runs on the same thread, but what happen if we set it to run on different thread? or maybe we introduce delay to it?

```
Definition wordDefinition;

@Test
public void searchWordDefinition() {
    final Definition definition = new Definition(FAKE_TAGS, FAKE_WORDS);
    when(wordsRemoteDataSource.getWordDefinition(TEST_QUERY)).thenReturn(Observable.just(definition));
    
    wordsRepository.getWordDefinition(TEST_QUERY)
            .delay(1000, TimeUnit.MILLISECONDS) // A delay can broke your test!
            .subscribe(newDefinition -> {
                wordDefinition = newDefinition;
            });
    assertNotNull(wordDefinition);
}
```
And this one will fail. But then, how to test it? Let's move to the next way

## Blocking way

RxJava provide an operator called [toBlocking()](https://github.com/ReactiveX/RxJava/wiki/Blocking-Observable-Operators) (blockingY() in [RxJava 2](https://github.com/ReactiveX/RxJava/wiki/What's-different-in-2.0)) to provide us a way to convert Observable to BlockingObservable which can be used as a synchronous operation.

When using toBlocking() our code will become like this

```
@Test
public void searchWordDefinition() {
    final Definition definition = new Definition(FAKE_TAGS, FAKE_WORDS);
    when(wordsRemoteDataSource.getWordDefinition(TEST_QUERY)).thenReturn(Observable.just(definition));
    
    final Definition wordDefinition = 
            wordsRepository.getWordDefinition(TEST_QUERY)
                    .toBlocking()
                    .first();
    assertNotNull(wordDefinition);
}
```

As you can see, our test become simpler and cleaner! No global variable and even if you add delay it will still works!

```
@Test
public void searchWordDefinition() {
    final Definition definition = new Definition(FAKE_TAGS, FAKE_WORDS);
    when(wordsRemoteDataSource.getWordDefinition(TEST_QUERY)).thenReturn(Observable.just(definition));
    
    final Definition wordDefinition =
            wordsRepository.getWordDefinition(TEST_QUERY)
                    .delay(1000, TimeUnit.MILLISECONDS)
                    .toBlocking()
                    .first();
    assertNotNull(wordDefinition);
}
```
But what if I want to test if onCompleted is called or not? or what if I want to assert what received in onNext?  
In that case let's check the next way

## TestSubscriber

TestSubscriber is a helper class provided by RxJava that we can use for unit testing, to perform assertions, inspect received events, or wrap a mocked Subscriber.

With TestSubscriber, our code will become like this

```
@Test
public void searchWordDefinition() {
    final Definition definition = new Definition(FAKE_TAGS, FAKE_WORDS);
    when(wordsRemoteDataSource.getWordDefinition(TEST_QUERY)).thenReturn(Observable.just(definition));
    
    wordsRepository.getWordDefinition(TEST_QUERY).subscribe(wordDefinitionTestSubscriber);
    wordDefinitionTestSubscriber.assertValue(definition);
}
```
Not only become simpler, we can also check there is an error or not

```
@Test
public void searchWordDefinition() {
    when(wordsRemoteDataSource.getWordDefinition(TEST_QUERY)).thenReturn(Observable.error(new IllegalArgumentException()));
        
    wordsRepository.getWordDefinition(TEST_QUERY).subscribe(wordDefinitionTestSubscriber);
    wordDefinitionTestSubscriber.assertError(IllegalArgumentException.class);
}
```
or maybe number of items that emitted by onNext

```
@Test
public void testValueCont() {
    Observable.just("Hello", "RxJava", "Testing").subscribe(wordListTestSubscriber);
    wordListTestSubscriber.assertValueCount(3);
}
```
And another nice feature of TestSubscriber is human-readable AssertionError message. For example if we change above test value count to 2, it will produce error message like this

```
java.lang.AssertionError: Number of onNext events differ; expected: 2, actual: 3 (1 completion)
```
and if we change the exception type above, it will produce

```
java.lang.AssertionError: Exceptions differ; expected: class java.lang.NullPointerException, actual: java.lang.IllegalArgumentException
```

Which can really helpful for us to fix the error.

### Ending Words

There are several ways to do testing in RxJava, but I recommend you to use TestSubscriber because it will give you more feature, flexibility, better error message and it's also the official way to test an Observable in RxJava.  
Thanks for reading and if you have any question you can write comment below or reach me via twitter [@niko_yuwono](https://twitter.com/niko_yuwono).
