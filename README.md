# What is a Thread?

## Learning Goals

- Explain the differences between a program, process, and thread.
- Define the relationship between a thread and a stack.
- Demonstrate single threaded versus multithreaded program execution.

## Introduction

In order to understand concurrency and multithreading we have to first
understand some basic terms. We’ll look at the following terms:

- Program
- Process
- Thread
- Stack

A program is a set of instructions and data to perform a specialized task.

A process is an instance of an executing program.
Each process is an environment that allocates the necessary system resources
to run the instructions in the program. 

A thread is the smallest unit of execution within a process.  Every
process executes at least one thread. However, a process can
run multiple threads to carry out the program instructions.
Each thread has its own stack.

A stack holds information about the methods that are executing in
that thread. When a method is called, a "stack frame" is pushed onto the stack
to store the details of the method call. The frame stores the method's local
variables, including parameters and possibly the `this` reference if the method
is called on an object, along with other data like the return address
so the thread knows where to go when the method finishes executing.

## Single Threaded Program Execution

So far the Java programs we've written were single-threaded.
When we run a program, Java automatically create a thread and then executes
the `main` method within the thread.

Let's look at a single-threaded program to understand how the call stack works.
Consider the `v1.Dog` class shown below, which has instance variables to
store the dog's name, whether their tail is wagging, and if they like baths.
The constructor does not assign `isWaggingTail`, so the instance variable will
have the default value `false`.

```java
public class Dog {
    private String name;
    private boolean isWaggingTail;
    private boolean likesBaths;

    public Dog(String name, boolean likesBaths) {
        this.name = name;
        this.likesBaths = likesBaths;
    }

    public void eatTreats(int numTreats) {
        for (int i=1; i<= numTreats; i++)
            System.out.println(name + " eats treat #" + i + " (" + Thread.currentThread().getName() + ")");
        isWaggingTail = true;
        System.out.println("After treats " + toString() + " (" + Thread.currentThread().getName() + ")");
    }

    public void takeBath() {
        isWaggingTail = likesBaths;
        System.out.println("After bath " + toString() + " (" + Thread.currentThread().getName() + ")");
    }

    @Override
    public String toString() {
        return String.format("%s wagging is %b" , name, isWaggingTail);
    }

    public static void main(String[] args) {
        System.out.println("Start " + " (" + Thread.currentThread().getName() + ")");
        int treats = 4;
        Dog snoopy = new Dog("Snoopy", false);       
        Dog fifi = new Dog("Fifi", true);           
        snoopy.eatTreats(treats);   
        fifi.eatTreats(treats);     
        snoopy.takeBath();          
        System.out.println("Finish " + " (" + Thread.currentThread().getName() + ")");
    }
}
```

The `main` method creates two dogs and then calls the `eatTreats` and `takeBath` methods,
which reassign the `isWaggingTail` variable and print the resulting object state.
The print statements append the name of the current thread
by calling `Thread.currentThread().getName()`.
In a single threaded program, the default thread is named "main".

The output will be the same every time the program runs.

```text
Start  (main)
Snoopy eats treat #1 (main)
Snoopy eats treat #2 (main)
Snoopy eats treat #3 (main)
Snoopy eats treat #4 (main)
After treats Snoopy wagging is true (main)
Fifi eats treat #1 (main)
Fifi eats treat #2 (main)
Fifi eats treat #3 (main)
Fifi eats treat #4 (main)
After treats Fifi wagging is true (main)
After bath Snoopy wagging is false (main)
Finish  (main)
```

Let's use the debugger and visualizer to explore the program and understand the call stack.
We'll set a breakpoint at the first print statement in the `eatTreats` method.
The breakpoint is reached after the `main` method calls `snoopy.eatTreats(numTreats)`:

![Single thread stack](https://curriculum-content.s3.amazonaws.com/java-mod-4-threads-intro/introduction-to-threads/singlethread.png)

The `main` method is automatically called when the program starts.
The call stack contains a frame to store data for the `main` method:

- A local variable that holds a primitive value such as an integer
  is stored directly in the frame.
- Objects are not stored in the stack, they are
  stored in a separate area of memory called the heap. 
  A local variable that holds an object reference stores the heap address. 
- Data about the current line of execution is stored in the frame. For example,
  the `main` method is at line `33`, which corresponds to `snoopy.eatTreats(treats)`.

The `eatTreats` frame will be popped off the stack when the method finishes
executing its statements. The `main` method can then proceed with its
next line of code, which will add a new frame to the stack
corresponding to the call `fifi.eatTreats(treats)`.

Because the single threaded program has just one stack,
the method calls always execute in the exact order listed:

```java
snoopy.eatTreats(treats);  
fifi.eatTreats(treats);    
snoopy.takeBath();  
```

## Multithreaded program execution

Let’s briefly look at how we can use multiple threads in Java to support concurrent execution.

Java's `Thread` class provides constructors and methods
to create and perform operations on a thread.

- The `Thread` class has a constructor that takes a `Runnable` object as an argument.
- `Runnable` is a functional interface that defines a single abstract method:  `void	run()`
  - The object passed into the `Thread` constructor must implement the `run()` method.
- We call the `start()` method on the new `Thread` object.
  - Starting a thread automatically calls the `run()` method on the `Runnable` object.
  - The `run()` method executes on the thread's stack.

We will adapt the `main` method to create 3 separate threads to execute
the `eatTreats` and `takeBath` method calls.  Each thread has its
own stack, which means the methods can execute is a concurrent rather than
sequential order.

We will pass a lambda expression as an argument to the `Thread` constructor.
The lambda expression implements the abstract `run()` method.
The lambda body is executed when the `start()`
method is called on the corresponding thread object.

```java
public static void main(String[] args) {
    System.out.println("Start " + " (" + Thread.currentThread().getName() + ")" );
    int treats = 4;
    Dog snoopy = new Dog("Snoopy", false);       
    Dog fifi = new Dog("Fifi", true);            
        
    Thread thread0 = new Thread( () -> snoopy.eatTreats(treats) );
    Thread thread1 = new Thread( () -> fifi.eatTreats(treats) );
    Thread thread2 = new Thread( () -> snoopy.takeBath() );

    thread0.start();
    thread1.start();
    thread2.start();
    
    System.out.println("Finish " + " (" + Thread.currentThread().getName() + ")" );
}  
```

By default, Java will name the threads main, Thread-0, Thread-1, and Thread-2.
Because there are 4 threads, the order of the print statements may differ each time the program executes.

For example, we might see this output:

```text
Start  (main)
Finish  (main)
After bath Snoopy wagging is false (Thread-2)
Fifi eats treat #1 (Thread-1)
Fifi eats treat #2 (Thread-1)
Snoopy eats treat #1 (Thread-0)
Fifi eats treat #3 (Thread-1)
Fifi eats treat #4 (Thread-1)
Snoopy eats treat #2 (Thread-0)
Snoopy eats treat #3 (Thread-0)
Snoopy eats treat #4 (Thread-0)
After treats Fifi wagging is true (Thread-1)
After treats Snoopy wagging is true (Thread-0)
```

The sample output corresponds to the shared timeline shown below.
We can see the print statements in the `main` thread both happen before
`Thread-2` executes the print statement from the call `snoopy.takeBath()`.
`Thread-0` and `Thread-1` then alternate executing the code
from the method calls `snoopy.eatTreats(treats)` and `fifi.eatTreats(treats)`.

```text
main       - -  
Thread-0                -      - - -    -  
Thread-1           - -     - -        -  
Thread-2        -  
```

How does this happen?  Recall that each thread has its own stack.
The image below depicts the heap and stacks for the sample program.
Note the heap would also contain objects for each thread
and lambda implementation, but only the dog objects are shown.


![Multithreaded stacks](https://curriculum-content.s3.amazonaws.com/java-mod-4-threads-intro/introduction-to-threads/multithreaded.png)

The Java runtime system allows the threads to take turns executing
the methods on their stack. The order and duration of each turn
may differ every time the program executes.

## Conclusion

This lesson introduced the concept of concurrency by demonstrating a program with
multiple threads. 

Multithreading is the reason the browser can react to mouse clicks while loading the page,
and why a game can handle user input from the keyboard, mouse, and touch screen while
showing animation on the monitor screen.

Once we take a deeper dive into threads, you’ll be able to handle multiple threads in your program!


## Resources

[Java 11 Thread](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Thread.html)  
[Java 11 Runnable](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Runnable.html)  
