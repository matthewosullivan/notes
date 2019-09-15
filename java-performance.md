# Java application performance and memory management

## What happens when the Java Virtual Machine runs our code?

Just In Time (JIT) compilation

Code runs faster the longer it is left to run

Which of the methods or code blocks in an application are being compiled.

There's a JVM flag we can use to find that out.

```
-XX:+PrintCompilation
```

```-XX``` Means this is an advanced option, then there's a ```:```. Then we have a ```+``` or a ```-``` which indicates if we 
want the option to be switched on or off, then the option name.

```
$ java -XX:+PrintCompilation main.Main 10
```

We're seeing every bit of compiling that is going on within the virtual machine as our application runs.
This is split into columns.

1. Number of milliseconds since the virtual machine started
2. Order in which the method of code block was compiled

- One column that has an N in - N here means native method
- One column that has an S in - S means it's a synchronized method
- Exclamation mark means there's some exception handling going on
- Percentage symbol would mean that the code has been natively compiled and is now running in a special part of memory called
the code cache, a percentage symbol in here means that the method is now running in the most optimal way possible.
- Next column has a number from 0 to 4 - tells us what kind of compiling has taken place
  - 0 no compilation the code has been interpreted
  - 1 to 4 means that a progressively deeper level of compilation has happened
- Line of code that has actually compiled

## JVM JIT Compilers

Two compilers built into the Java Virtual Machine called C1 and C2.

- C1 is able to do the first three levels of compilation - each is progressively more complex than the last
- C2 compiler can do the fourth level

The virtual machine decides which level of compilation to apply to a particular block of code

The flag ```-XX:+PrintCompilation``` prints compilation will output this information to the console. There is an alternative
which will output to a file and gives a little bit more information. Useful if you're wanting to know what's happening on a
remote machine where you can't see the console. Use a flag called log compilation but in order to use that we actually need
another flag called unlock diagnostic VM options and that flag must come first in the list.

```
$ java -XX:+UnlockDiagnosticVMOptions -XX:+LogCompilation main.Main 5000
```
Look in our folder there is a file produced by running Java Virtual Machine with this log compilation flag.

## Tuning Code Cache size

Code cache has a limited size
Increasing the size of the code cache can lead to an improvement in our application's performance.

```VM warning: CodeCache is full. Compiler has been disabled```

Find out about the size of the code cache by using a JVM flag

```
$ java -XX:+PrintCodeCache main.Main 5000
CodeHeap 'non-profiled nmethods': size=120032Kb used=27Kb max_used=27Kb free=120004Kb
 bounds [0x000000011ee19000, 0x000000011f089000, 0x0000000126351000]
CodeHeap 'profiled nmethods': size=120028Kb used=136Kb max_used=136Kb free=119891Kb
 bounds [0x00000001178e2000, 0x0000000117b52000, 0x000000011ee19000]
CodeHeap 'non-nmethods': size=5700Kb used=989Kb max_used=1002Kb free=4710Kb
 bounds [0x0000000117351000, 0x00000001175c1000, 0x00000001178e2000]
 total_blobs=401 nmethods=99 adapters=158
 compilation: enabled
              stopped_count=0, restarted_count=0
 full_count=0
```

Code cache is about 15 megabytes

Maximum size of the code cache is dependent on which version of java you are using.

- Java 7 or below
  - 48 megabytes 64 bit JVM
  - 32 megabytes 32 bit JVM
- Java 8 or above
  - 240 megabytes 64 bit JVM
  
Can change the code cache size with three different flags:

- InitialCodeCacheSize - When the application starts
- ReservedCodeCacheSize - Maximum size
- CodeCacheExpansionSize - How quickly the code cache should grow

## Remotely monitoring the code cache with JConsole

Start jconsole

```
$ jconsole
```

## The differences between the 32 bit and 64 bit JVM

The flag to force the use of the Client Compiler is ```-client```

```-server```

```-d64```

## Turning off tiered compilation

```
 java -XX:-TieredCompilation -XX:+PrintCompilation main.Main 5000
```

## Tuning native compilation with the Virtual Machine

Native Compilation Tuning

```
$ java -XX:+PrintFlagsFinal
intx CICompilerCount                          = 4                                         {product} {ergonomic}
```
4 threads available for compiling our code

Find java process ID using jps command

```
$ jps
55025 Jps
54561 
```
J info command tool

```
$ jinfo -flag CICompilerCount 54561
-XX:CICompilerCount=4
```
Increase CICompilerCount
```
 java -XX:CICompilerCount=8 -XX:+PrintCompilation main.Main 15000
```

Threshold the number of times a method needs to run before it is natively compiled

```
$ jinfo -flag CompileThreshold 54561
-XX:CompileThreshold=10000
```
Ten thousand - this is the number of times the method has to be run 

## Structure of Java Memory













