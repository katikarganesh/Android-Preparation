# Android-Preparation

## Core building blocks of Android
- Activity
- Fragment
- Views
- Intent
- Service
- Content Providers
- Android Manifest

## Activity
#### Activity is a class that represents a single screen.
#### Activity has its own lifecycle method mentioned below.
- onCreate() - Called when an activity is first created.
- onStart() - Called when an activity becomes visible to the user.
- onResume() - Called when an activity starts interacting with the user.
- onPause() - Called when an activity is not visible to the user.
- onStop() - Called when an activity is no longer visible to the user.
- onRestart() - Called when an activity stopped, prior to restart.
- onDestroy() - Called when an activity is destroyed.

## Flow in Kotin
#### Flow is a reactive stream processing library that provides a way to emit and consumes streams of data asynchronously and efficiently.
![flow1.png](assets/flow1.png)
<img align="center" src="file:///assets/flow1.png" height="80" />

There are two types of Streams.
#### Hot Stream
Hot Stream produces data no matter consumer is consuming it or not.
#### Cold Stream
Cold Stream produces data only if consumer is consuming at the other side.

#### Flow are by default in Cold nature
Example:
```kotlin
GlobalScope.launch {
    val result = producer()
        result.collect {
            delay(1000)
            Log.d("TAG", "==> collect $it")
        }
}
        
fun producer(): Flow<Int> {
    val list = listOf<Int>(1, 2, 3, 4, 5)
    return flow<Int> {
       list.forEach {
          emit(it)
       }
    }
 }
```
Output:
```kotlin
==> collect 1
==> collect 2
==> collect 3
==> collect 4
==> collect 5
```

#### In case of multiple consumer how the Cold Stream behave?
In case of Cold Stream all the consumers will get the data from starting even if they joined late, but in case of Hot Stream whoever join the late will get the data from that point of state only.

Example:

In the below example we are having two consumers which will consumes data which are emitted from producer function.

```kotlin
//Consumer A
GlobalScope.launch {
    val result = producer()
        result.collect {
            delay(1000)
            Log.d("TAG", "==> collect1 $it")
        }
}

//Consumer B
GlobalScope.launch {
    val result = producer()
    delay(2500)
    result.collect {
        delay(1000)
        Log.d("TAG", "==> collect2 $it")
    }
}
    
fun producer(): Flow<Int> {
    val list = listOf<Int>(1, 2, 3, 4, 5)
    return flow<Int> {
        list.forEach {
            emit(it)
        } 
    }
}
```
In the above example there are two consumers Consumer A and Consumer B. Both are consuming data produced from producer function, but Consumer B has started consuming late by 2.5 seconds still Consumer B will get data from initial state.

Below is the output:
```kotlin
==> collect1 1
==> collect1 2
==> collect1 3
==> collect2 1
==> collect1 4
==> collect2 2
==> collect1 5
==> collect2 3
==> collect2 4
==> collect2 5
```

#### How to cancel flow?
Flow are cancellable by default.
```kotlin
val job=scope.launch{ flow.cancellable().collect{ }}
job.cancel()
```
cancellable() will ensure the flow is terminated before new items are emitted to collect {}.  if job is cancelled though flow builder then all its implementation are cancellable() by default.

#### Events in flow?
##### onStart() :
onStart(){} blocks will gets executed before consuming of any item at the start.
##### onCompletion()
onCompletion(){} blocks will gets executed after all the items are consumed.
##### onEach()
onEach(){} blocks will gets executed before each item is about to emit.

Example:
```kotlin
GlobalScope.launch {
            val result = producer()
            result.onStart {
                Log.d("TAG", "==> on start")
            }.onCompletion {
                Log.d("TAG", "==> on completion")
            }.onEach {
                Log.d("TAG", "==> about to emit $it")
            }.collect {
                delay(1000)
                Log.d("TAG", "==> collect $it")
            }
}

fun producer(): Flow<Int> {
        val list = listOf<Int>(1, 2, 3, 4, 5)
        return flow<Int> {
            list.forEach {
                emit(it)
            }
        }
}
```
Output:
```kotlin
==> on start
==> about to emit 1
==> collect 1
==> about to emit 2
==> collect 2
==> about to emit 3
==> collect 3
==> about to emit 4
==> collect 4
==> about to emit 5
==> collect 5
==> on completion
```
#### Operators in Flow

Two types of operators presents in kotlin flow. **Terminal operators** and **Non-terminal operators**.

- Terminal operators are responsible for starting a flow. It is a suspend function.
e.g first(), toList(), collect() etc

- Non-terminal operators are not responsible for starting a flow.
e.g map{}, filter{}, buffer() etc

##### first(): Returns the first value

##### last(): Returns the last value

##### toList(): Returns the entire object as a result.

Example:
```kotlin
GlobalScope.launch {
   val result = producer()
   Log.d("TAG, ","==> Result first(): ${result.first()}")
   Log.d("TAG, ","==> Result toList(): ${result.toList()}")
   Log.d("TAG, ","==> Result last(): ${result.last()}")
}

fun producer(): Flow<Int> {
        val list = listOf<Int>(1, 2, 3, 4, 5)
        return flow<Int> {
            list.forEach {
                emit(it)
            }
        }
}
```

```kotlin
==> Result first(): 1
==> Result toList(): [1, 2, 3, 4, 5]
==> Result last(): 5
```

##### map{}: Transform the one object into another.

##### filter{}: Filter the object based on condition.

```kotlin
GlobalScope.launch {
            val result = producer()
            result.map {
                it*2
            }.filter {
                it%2==0
            }.collect{
                Log.d("TAG", "==> ${it}")
            }
}

fun producer(): Flow<Int> {
        val list = listOf<Int>(1, 2, 3, 4, 5)
        return flow<Int> {
            list.forEach {
                emit(it)
            }
        }
}
```

```kotlin
==> 2
==> 4
==> 6
==> 8
==> 10
```

##### buffer
Buffers flow emissions via channel of a specified capacity and runs collector in a separate coroutine.

```kotlin
flowOf("A", "B", "C")
    .onEach  { println("1$it") }
    .collect { println("2$it") }
```
It is going to be executed in the following order by the coroutine Q that calls this code:

```kotlin
Q : -->-- [1A] -- [2A] -- [1B] -- [2B] -- [1C] -- [2C] -->--
```

So if the consumer's code takes considerable time to execute, then the total execution time is going to be the sum of execution times for all consumer's.

The buffer consumer's creates a separate coroutine during execution for the flow it applies to. Consider the following code:

```kotlin
flowOf("A", "B", "C")
    .onEach  { println("1$it") }
    .buffer()  // <--------------- buffer between onEach and collect
    .collect { println("2$it") }

```

It will use two coroutines for execution of the code. A coroutine Q that calls this code is going to execute collect, and the code before buffer will be executed in a separate new coroutine P concurrently with Q:

```kotlin
P : -->-- [1A] -- [1B] -- [1C] ---------->--  // flowOf(...).onEach { ... }

                      |
                      | channel               // buffer()
                      V

Q : -->---------- [2A] -- [2B] -- [2C] -->--  // collect

```

When the consumer's code takes some time to execute, this decreases the total execution time of the flow

##### flowOn()
Using flowOn operator we can change the context of the flow

```kotlin
withContext(Dispatchers.Main) {
    val singleValue = intFlow // will be executed on IO if context wasn't specified before
        .map { ... } // Will be executed in IO
        .flowOn(Dispatchers.IO)
        .filter { ... } // Will be executed in Default
        .flowOn(Dispatchers.Default)
        .single() // Will be executed in the Main
}
```

##### catch
Catches exception in the flow.
```kotlin
flow { 
    emitData()
}.map {
    computeOne(it) 
}.catch { 
... // catches exceptions in emitData and computeOne 
}.map { 
computeTwo(it) 
}.collect { 
process(it) 
} // throws exceptions from process and computeTwo

```

##### zip
Zips values from the current flow (this) with other flow
Example:
```kotlin
GlobalScope.launch {
            val flow = flowOf(1, 2, 3).onEach { delay(10) }
            val flow2 = flowOf("a", "b", "c", "d").onEach { delay(15) }
            flow.zip(flow2) { i, s -> i.toString() + s }.collect {
                println("==> $it") // Will print "1a 2b 3c"
            }
        }
```
Output
```kotlin
==> 1a
==> 2b
==> 3c
```

#### Shared Flow is Hot Stream in nature

Example:
```kotlin
//ConsumerA
GlobalScope.launch(Dispatchers.Main) {
            val consumerA = producer()
            consumerA.collect {
                Log.d("TAG", "==> Consumer A: $it")
            }
}

//ConsumerB
GlobalScope.launch(Dispatchers.Main) {
            val consumberB = producer()
            delay(2500)//Will add some delay so that Consumer B will start collecting values after 2.5 seconds
            consumberB.collect {
                Log.d("TAG", "==> Consumer B: $it")
            }
}

fun producer(): MutableSharedFlow<Int> {
        val mutableSharedFlow = MutableSharedFlow<Int>()
        GlobalScope.launch {
            val list = listOf<Int>(1, 2, 3, 4, 5)
            list.forEach {
                mutableSharedFlow.emit(it)
                delay(1000)
            }
        }
        return mutableSharedFlow
}
```

Output
```kotlin
==> Consumer A: 1
==> Consumer A: 2
==> Consumer A: 3
==> Consumer B: 4
==> Consumer A: 4
==> Consumer B: 5
==> Consumer A: 5
```

In the above example consumer A started initially but consumer B started after 2.5 seconds hence after that time the current emission value is 4 in abvoe case hence consumer B will get value from 4. Shared Flow are Hot Stream in nature and hence it is not independent of consumer.


#### StateFlow
Similar to Shared Flow but it will maintain last emitted value as a state.

In the previous example if we consider then output will start from 3 for consumer B. Because after 2.5 seconds the last emmited value was 3 hence in case of Shared Flow it starts with 4 but in case of State Flow it will start from last stored value.

LiveData Vs Flow

| LiveData    | Flow |
| -------- | ------- |
| All the operations and filters are done on main thread  | All the operations and filters are done on any thread. Using flowOn operater we can change the thread for flow.    |
| We have limited operators in LiveData | More operators as compared to LiveData     |
| LiveData is lifecycle aware    | Flow is independent, and require coroutine.    |














