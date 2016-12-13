---
layout: post
title: Taking a look at the new FragmentTransaction's commitNow()
description: There is a new commitNow() method added in Android 7.0 and support library version 24 that allows us to commits the transaction synchronously. But if we read the documentation carefully there is one paragraph that tells us that the transactions commited using commitNow() may not be added to the FragmentManager's backstack.
---

So it's all started when I saw this [interesting question](http://stackoverflow.com/questions/38566628/how-is-the-new-fragmenttransaction-commitnow-working-internally) in stackoverflow.  

There is a new commitNow() method added in Android 7.0 and support library version 24 that allows us to commits the transaction synchronously. But if we read the documentation carefully there is one paragraph that tells us that the transactions commited using commitNow() may not be added to the FragmentManager's backstack.

>Transactions committed in this way may not be added to the FragmentManager's back stack, as doing so would break other expected ordering guarantees for other asynchronously committed transactions. This method will throw IllegalStateException if the transaction previously requested to be added to the back stack with addToBackStack(String). 

Then when the question poster tried to use `addToBackStack(String)` and `commitNow()` at the same time, he got this confusing error that tell him that the transaction is already being added to the backstack.


>java.lang.IllegalStateException: This transaction is already being added to the back stack

Why that Exception was thrown? So let's take a look how is `commitNow()` really work!

## FragmentTransaction

Before digging deeper into `commitNow()` let's see how FragmentTransaction works.  
Usually we begin a transaction like this (in this post I will use the support library one) :

```
getSupportFragmentManager().beginTransaction()
```

Which if you take a look at FragmentManagerImpl class implementation [here](https://github.com/android/platform_frameworks_support/blob/2167a9174649d66111f2be5a0b1fc96b15cc48aa/fragment/java/android/support/v4/app/FragmentManager.java#L588) will return a new [BackStackRecord](https://github.com/android/platform_frameworks_support/blob/2167a9174649d66111f2be5a0b1fc96b15cc48aa/fragment/java/android/support/v4/app/BackStackRecord.java#L197). After that you can do transactions like add, remove, replace, etc using the `FragmentTransaction` that returned from `beginTransaction()` like this

```
getSupportFragmentManager()
    .beginTransaction()
    .add(myFragment)
```

Then after that you need to call `commit()` to commit your transaction that if you take a look at the [source](https://github.com/android/platform_frameworks_support/blob/2167a9174649d66111f2be5a0b1fc96b15cc48aa/fragment/java/android/support/v4/app/BackStackRecord.java#L696), it will add the transaction into FragmentManagerImpl's queue and will be executed when the transaction dequeued.

```
getSupportFragmentManager()
    .beginTransaction()
    .add(myFragment)
    .commit()
```

## commitNow()

Sometimes we want to the transaction to be executed immediately instead adding it into the queue, in the past we use executePendingTransactions() to do that. But now we have commitNow() to commits our transaction synchronously, even the documentation favor commitNow() over executePendingTransactions().

>Calling commitNow is preferable to calling commit() followed by FragmentManager.executePendingTransactions() as the latter will have the side effect of attempting to commit all currently pending transactions whether that is the desired behavior or not.

To use it, just replace the call to commit() to commitNow()

```
getSupportFragmentManager()
    .beginTransaction()
    .add(myFragment)
    .commitNow()
```

## commitNow() and backstack

So we back to the question why this exception was thrown?

>java.lang.IllegalStateException: This transaction is already being added to the back stack

So let's take a look at `BackStackRecord.commitNow()` source [here](https://github.com/android/platform_frameworks_support/blob/2167a9174649d66111f2be5a0b1fc96b15cc48aa/fragment/java/android/support/v4/app/BackStackRecord.java#L671)

```
@Override
public void commitNow() {
    disallowAddToBackStack();
    mManager.execSingleAction(this, false);
}

@Override
public FragmentTransaction disallowAddToBackStack() {
    if (mAddToBackStack) {
        throw new IllegalStateException(
                "This transaction is already being added to the back stack");
    }
    mAllowAddToBackStack = false;
    return this;
}
```

As we can see, `commitNow()` will call `disallowAddToBackStack()` and will throw an exception when `mAddToBackStack` is true. If we add a call into `addtoBackStack()` in our transaction `mAddToBackStack` will be set to true and that exception will be thrown.  

What bothers me is the unclear Exception message that thrown by `disallowAddToBackStack()`, I think it will be clearer if the message is something like `You're not allowed to add to backstack when using commitNow() or commitNowAllowingStateLoss()`


## Bonus: executePendingTransactions() vs commitNow()

When I read the documentation of commitNow() for the first time I instantly wondered what's the difference between executePendingTransactions() and commitNow(). I think if you use fragments a lot, there are some situation where you need to call executePendingTransaction() to execute all pending transactions.  

If we dig deeper into `FragmentManager source code`, `commitNow()` (that will call [FragmentManagerImpl.execSingleAction()](https://github.com/android/platform_frameworks_support/blob/2167a9174649d66111f2be5a0b1fc96b15cc48aa/fragment/java/android/support/v4/app/FragmentManager.java#L1629)) actually doing almost same thing as `executePendingTransactions()` (that will call [FragmentManagerImpl.execPendingAction()](https://github.com/android/platform_frameworks_support/blob/2167a9174649d66111f2be5a0b1fc96b15cc48aa/fragment/java/android/support/v4/app/FragmentManager.java#L1652)), but instead executing all previously committed transaction, `commitNow()` will only commit that transaction.

I think that's the main reason why commitNow() isn't allowing addition to the backstack since it cannot guarantee there aren't any other pending transaction. If commitNow() can add to the backstack, there is a possibility that we can break our backstack sequence that will leading into unexpected thing.

## Ending Words

The new FragmentTransaction's commitNow() provide us a new way to commit into transaction, Google even recommend it over calling commit() followed by FragmentManager.executePendingTransactions() as the latter will have the side effect of attempting to commit all currently pending transactions whether that is the desired behavior or not.  

But we need to be careful when committing transaction with commitNow() since the transaction won't be added into backstack.

Thanks for reading and if you have any question you can write comment below or reach me via twitter [@niko_yuwono](https://twitter.com/niko_yuwono).
