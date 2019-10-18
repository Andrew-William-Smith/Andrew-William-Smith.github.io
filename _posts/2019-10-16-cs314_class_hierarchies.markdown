---
layout: post
title:  "Class Hierarchy Questions in CS 314"
date:   2019-10-16 18:52:00
categories: code java cs314
---

One of the most difficult aspects of working with code in object-oriented languages like Java is navigating and tracing deep hierarchies of inherited classes.
In [CS 314 (Data Structures)](https://www.cs.utexas.edu/~scottm/cs314/index.htm) at the University of Texas at Austin, a large emphasis is placed on this skill as we expect students to learn to work with increasingly complex hierarchies of data structures and the major principles of object-oriented program design.
As such, the exams in all semesters include at least one section on handling inheritance and polymorphism; this portion of each exam proves difficult for many students, as the precise rules of inheritance in Java can be difficult to trace through, especially in a time-constrained setting.
In this post, I shall walk through one such set of problems from the first exam of the Fall 2019 semester, explaining each method and variable resolution as well as the overall class hierarchy.

# Laying the groundwork: Class definitions
On this exam, students were presented with the following series of class definitions, along with the implementations of some simple methods, describing common living arrangements:

{% highlight java %}
public class Living {
    private int sp;
    public Living() { sp = 5; }
    public Living(int s) { sp = s; }

    public int get() { return sp; }
    public void add(int s) { sp += s; }
    public void inc() { sp += 10; }
    public String toString() { return "L" + get(); }
}

public class House extends Living {
    public House(int x) { add(x); }
    public void add() { inc(); }
    public int space() { return 100; }
}

public class Split extends House {
    private int num;
    public Split() { super(20); }
    public Split(int n) {
        this();
        num = n;
    }

    public void add(int n) { num += n; }
    public int get() { return num; }
}

public class Apt extends Living {
    private int fs;

    public String toString() { return super.toString() + " " + fs; }
    public void set(int f) { fs = f; }
    public void add (int f, int s) {
        fs += f;
        add(s);
    }
}
{% endhighlight %}

*A note to CS 314 students:* The above code is not an example of acceptable code style, and you should not format your assignments in this manner; however, this formatting saves a significant amount of whitespace and is how problems like this are presented on exams, so I have duplicated the exam style here.

## Step 1: Drawing a class diagramme
The first step in solving any of these inheritance problems is to draw a diagramme of the class hierarchy.  Recalling that all classes derive from `Object`, the class hierarchy comprised by these definitions ([Graphviz source](/assets/2019-10-16/class-hierarchy.gv)) is as follows:

![Class diagramme for this problem](/assets/2019-10-16/class-hierarchy.png){:.thin-center}

Diagrammes like this can give us a good initial impression of the problem and are immensely helpful when it comes to determining whether an invocation is legal; nevertheless, there are still better techniques we can use to determine which methods are run for a given call.
While the full version of this approach is not effective for exams since it is a somewhat lengthy process, partial applications of it can be used to ensure full correctness in your manual method resolutions.

## Step 2: Fully resolving methods
*Note:* This section describes in detail how inherited methods are resolved in Java.  If you'd rather not read through this information, feel free to skip ahead to [the actual sample problems](#solving-some-sample-problems){:.internal}.

Using the provided code and the diagramme we constructed above, we can rewrite all methods of these classes in terms of the actual methods executed, essentially percolating higher levels of the inheritance tree down to lower levels to determine the exact method run for a given call.

An important behaviour to note is that if a constructor does not ever explicitly invoke a `super()` constructor in its constructor, the zero-argument `super()` constructor of its parent will be invoked automatically.
This behaviour continues all the way up the inheritance tree until `Object` is reached, ensuring that all of a class's instance variables will be initialised any time an instance of that class is created.
However, any explicit call to `super()` overrides this behaviour for a given level of the hierarchy: for example, since the constructor for `Split` explicitly invokes `super(20)`, `super()` itself will not be called in that constructor.
Nevertheless, since the resolved version of that call, `House(20)`, does not invoke a superclass constructor explicitly, the zero-argument constructor `Living()` will be invoked automatically as well.
This behaviour is often called *constructor chaining* and can be one of the more difficult parts of the Java object model to work with in an exam setting.

The fully-resolved versions of all of the classes we have defined can be found below.
Note that the symbol `~~` means *is inherited from*, meaning that when the method before the sign is called, the method after it is invoked.
This is not standard Java syntax, but is an economical way to indicate which methods are actually called for a given method invocation.

{% highlight java %}
public class Living {
    private int sp;
    public Living() { sp = 5; }
    public Living(int s) { sp = s; }

    public int get() { return sp; }
    public void add(int s) { sp += s; }
    public void inc() { sp += 10; }
    public String toString() { return "L" + get(); }
}

public class House extends Living {
    private int sp ~~ Living.sp;
    public House(int x) {
        super() ~~ Living();    // Implicit super constructor
        add(x) ~~ House.add(x) ~~ Living.add(x);
    }

    public void add() { inc() ~~ Living.inc(); }
    public int space() { return 100; }
    // Inherited methods
    public int get() ~~ Living.get()
    public void add(int s) ~~ Living.add(int s)
    public void inc() ~~ Living.inc()
    public String toString() ~~ Living.toString()
}

public class Split extends House {
    private int num;
    private int sp ~~ House.sp ~~ Living.sp;

    public Split() { super(20) ~~ House(20); }
    public Split(int n) {
        this() ~~ Split() ~~ House(20);
        num = n;
    }

    public void add(int n) { num += n; }
    public int get() { return num; }
    // Inherited methods
    public void add() ~~ House.add()
    public int space() ~~ House.space()
    public void inc() ~~ House.inc() ~~ Living.inc()
    public String toString() ~~ House.toString() ~~ Living.toString()
}

public class Apt extends Living {
    private int fs;
    private int sp ~~ Living.sp;

    // Automatically created zero-argument constructor
    public Apt() { super() ~~ Living(); }

    public String toString() {
        return (super.toString() ~~ Living.toString()) + " " + fs;
    }
    public void set(int f) { fs = f; }
    public void add(int f, int s) {
        fs += f;
        add(s) ~~ Living.add(s);
    }
    // Inherited methods
    public int get() ~~ Living.get()
    public void add(int s) ~~ Living.add(int s)
    public void inc() ~~ Living.inc()
}
{% endhighlight %}

# Solving some sample problems
Now we shall put our hard work into action by using it to solve some sample problems from the first Fall 2019 CS 314 exam.
Problem letters are included for reference in case you'd like to follow along with the practice exam posted on Professor Scott's website or check your work as you practice.

## Problem N.  Validating polymorphism
> For each line of code, write `valid` if the line will compile without error or `invalid` if it causes a compile error.  
> ```java
Apt a2 = new Split();
House h2 = new Object();
```

The first example is attempting to create an object with a static type `Apt` and a dynamic type `Split`.
Looking at our [inheritance diagramme](#step-1-drawing-a-class-diagramme){:.internal}, we can see that while both classes inherit from `Living`, they are in different branches of the tree.
This indicates to us that a `Split` can never be cast to `Apt`, so this code sample is **invalid**.

The second example is attempting to create an object with a static type `House` and a dynamic type `Object`.
Since `Object` is higher up in our inheritance tree than `House`, we know that an `Object` will never be able to fulfil the full contract of `House`: that is, `House` may define methods or instance variables that an instance of `Object` cannot provide.
Thus, the second line will also result in a compile error, making it **invalid**.

## Problem O
> What is output by the following code?  
> ```java
House h1 = new House(10);
h1.inc();
System.out.print(h1);
```

This is our first program analysis problem: that is, in order to solve it fully, we have to execute some code in our heads or on paper.
While this may seem like a somewhat daunting task at first (we're not compilers after all!), the [method resolutions](#step-2-fully-resolving-methods){:.internal} we walked through earlier will be very useful to us now.
We can use these traces to write out the full sequences of code that are executed for each line of the problem, showing exactly what will happen with each method call and variable access.
We shall step through each line of code to track exactly what our object looks like after each line executes, explaining which functions are called at all steps along the way.

{% highlight java %}
House h1 = new House(10) {
    super() ~~ Living() {
        sp = 5;    // h1.sp = 5
    };
    add(x = 10) ~~ House.add(10) ~~ Living.add(s = 10) {
        sp += (s = 10);  // h1.sp = 15
    };
};
{% endhighlight %}

The first call to `House(10)` will call not only the `House(int)` constructor, but also implicitly the `Living()` constructor as discussed previously.
This behaviour may result in some unexpected instance variable values, demonstrating once again how important it is to keep constructor chaining in mind when we're dealing with deep inheritance hierarchies.

Within the `House` constructor, we also have to follow the chain of inheritance for the `add(int)` method, which takes us back to `Living#add(int)`.
While this isn't the most complex example of method inheritance, it's a good reminder that subclasses can inherit the methods of their superclasses.
At the end of this constructor, only one instance variable has been set: `h1.sp`, with a value of `15`.
Let's move on to the next line of code.

{% highlight java %}
h1.inc() ~~ Living.inc() {
    sp += 10;    // h1.sp = 25
};
{% endhighlight %}

This is a simple inherited method call, incrementing the value of the inherited instance variable `sp` by 10.
At the end of this method call, `h1.sp` has yet again changed its value, now to `25`.

{% highlight java %}
System.out.print(h1)
~~ System.out.print(h1.toString())
~~ System.out.print(Living.toString() {
       return "L" + get() ~~ Living.get() {
           return sp;    // h1.sp = 25
       };
});
{% endhighlight %}

At the core of this line is the requirement that we remember that `System.out.print()` implicitly calls `.toString()` on all of its arguments.
`House` also inherits its `toString()` method from `Living`, so the method that is called here is `Living#toString()`.

Now that we've traced through the whole function, we can see that the final value of instance variable `sp` is `25`.  Since `System.out.print()` will call `Living#toString()`, an `L` will be prepended to that variable for a final result of **`L25`**.

## Problem P
> What is output by the following code?
> ```java
Split s3 = new Split(50);
System.out.print(s3);
```

As before, let's begin by tracing through the constructor call.
The constructor chaining here is somewhat more complicated, so it'll take a bit more effort to work through; however, the same principles apply as in the previous problem.

{% highlight java %}
Split s3 = new Split(50) {
    this() ~~ Split() {
        super(20) ~~ House(x = 20) {
            super() ~~ Living() {
                sp = 5;    // s3.sp = 5
            };
            add(x = 20) ~~ Split.add(n = 20) {
                num += (n = 20);    // s3.num = 20
            };
        };
    };
    num = (n = 50);    // s3.num = 50
};
{% endhighlight %}

What a deep call tree!
Using the same method resolution rules we've applied thus far and with a bit of help from our expanded methods, however, we were able to successfully navigate it.
Of particular interest is the resolution of the call to `add(int)` from the `House(int)` constructor, which resolves to `Split#add(int)` instead of `House#add(int) ~~ Living#add(int)`.
Since `Split` overrides this method, `s3.add(int)` refers to that overriding method `Split#add(int)`, regardless of in which class the method that calls it resides.

Now that we've gotten the hairy part of the problem out of the way, we can move on to the next line of code.

{% highlight java %}
System.out.print(s3)
~~ System.out.print(s3.toString())
~~ System.out.print(House.toString())
~~ System.out.print(Living.toString() {
       return "L" + get() ~~ Split.get() {
           return num;    // s3.num = 50
       };
});
{% endhighlight %}

Again, the call stack is somewhat deeper since we have to traverse through not only `House` but also `Living` in order to find the `toString()` method, but the basic idea is roughly the same as the previous problem.
The call to `get()` follows the same rule as the call to `add(int)`: since `Split` overrides the method from `Living`, it is `Split`'s method that is called.
This method returns the value of instance variable `num` instead of `sp`, so our final output for this problem is **`L50`**.

## Problem Q
> What is output by the following code?
> ```java
Living g5 = new Apt();
g5.add(10);
System.out.println(g5.toString());
```

By now, you know the drill: let's begin with our constructor.
Class `Apt` does not explicitly define any constructor; however, Java automatically creates a zero-argument constructor for the class, which will be called when we attempt to instantiate `g5`.

{% highlight java %}
Living g5 = new Apt() {
    // Autogenerated constructor
    super() ~~ Living() {
        sp = 5;    // g5.sp = 5
    };
};
{% endhighlight %}

Inside the autogenerated constructor, we have a further aspect of constructor chaining to keep in mind: in the absence of an explicit call to `super()` within a constructor, Java will add a call to the superclass's zero-argument constructor for you.
Since the autogenerated default constructor is empty, a call to `super()`&mdash;which in this case resolves to `Living()`&mdash;is inserted as well.

Note also that it is important to consider both the static and dynamic types of objects we instantiate, as some (statistically, most) combinations of static and dynamic type will result in a compile error.
In this problem, the static type `Living` is a direct ancestor of the dynamic type `Apt`, meaning that every instance of `Apt` can also be considered to be an instance of `Living` by the standard rules of polymorphism.
As a result, this declaration is legal, meaning that we can proceed forth to the next line of code.

{% highlight java %}
g5.add(10) ~~ Living.add(s = 10) {
    sp += (s = 10);    // g5.sp = 15
};
{% endhighlight %}

In this line, we have a simple inheritance of the `add(int)` method from `Living`, as seen in both of the previous problems.
Note that the overloaded `Apt#add(int, int)` method was not called since we passed only a single argument in our invocation.

{% highlight java %}
System.out.println(g5.toString())
~~ System.out.println(Apt.toString() {
       return (super.toString() ~~ Living.toString() {
           return "L" + get() ~~ Living.get() {
               return sp;    // g5.sp = 15
           };
       }) + " " + fs;    // g5.fs = 0
});
{% endhighlight %}

As opposed to the previous few problems, the `Apt` class actually overrides `toString()`, so we don't have to trace that method through the inheritance tree in this instance.
By now, we can likely anticipate the outcome of the first call to `super.toString()`.
Since the superclass of `Apt` is `Living`, we determine that the method `Living#toString()` is called, printing a string `L` followed by the value of `g5.sp`, which in this case is `15`.
But whence came the instance variable `fs`, and how does it have a value of `0`?
Since `fs` is an instance variable defined in `Apt`, we would expect its value to be initialised in the `Apt()` constructor; however, we find no such line of code.
To finish solving this problem, we must remember that any uninitialised numeric primitives have a default value of `0`, a rule which applies to `g5.fs` in this problem.
Now that we know the sources of all of our data, we can perform the string concatenation to find our final answer of **`L15 0`**.

## Problem R
> What is output by the following code?
> ```java
Split s5 = new Split();
s5.add();
s5.add(10);
System.out.print(s5.get());
```

At this point, we've already seen what happens when we instantiate a `Split` using the zero-argument constructor; in the interest of brevity, I shall avoid reproducing the full call trace for that constructor invocation here.
If you'd like to review the entire trace, refer back to the instantiation of `Split s3` in [Problem P](#problem-p){:.internal}.
At the end of the instantiation, the state of `s5` is as follows:

{% highlight java %}
Split s5 = new Split();
// s5.sp = 5
// s5.num = 20
{% endhighlight %}

Now that we have `s5` in memory, we can advance to the next line, a call to the inherited method `add()`.

{% highlight java %}
s5.add() ~~ House.add() {
    inc() ~~ Living.inc() {
        sp += 10;    // s5.sp = 15
    };
};
{% endhighlight %}

Here, we have a simple call to the inherited method `Living#inc()`, which modifies the value of `s5.sp`.
Having some experience with these inherited calls and an inheritance tree to help us along, cases like these should be fairly easy to traverse by now.
Let's move on to the next line.

{% highlight java %}
s5.add(10) ~~ Split.add(n = 10) {
    num += (n = 10);    // s5.num = 30
};
{% endhighlight %}

There's not much to discuss here: `Split` overrides the `add(int)` method from `Living`, so that's the method we call in this instance.
Only one more line left!

{% highlight java %}
System.out.print(s5.get())
~~ System.out.print(Split.get() {
       return num;    // s5.num = 30
});
{% endhighlight %}

What a pleasant surprise&mdash;we didn't have to traverse a `toString()` method this time around.
As we've already seen, `Split` provides its own implementation of the `get()` method, so we return instance variable `num` from this call.
Thus, the output of this snippet is just **`30`**.

## Problem S
> What is output by the following code?
> ```java
Apt a8 = new Apt();
a8.fs += 10;
a8.add(10, 20);
System.out.print(a8);
```

Having worked through the last few problems, you may already be putting pencil to paper to begin resolving the call traces for this snippet.
However, it is always useful to briefly glance over these problems to ensure that the operations that they perform are actually valid: if you're the compiler, that means you also have to validate syntax!
Look at the second line of the problem, which attempts to set the value of instance variable `fs` from outside the `Apt` class.
Recalling the definition of `Apt`, we notice that `fs` is declared as a *private* instance variable, meaning that we cannot access it from a context outside of `Apt` or its descendants.
Thus, this example results in a **compile error**, so we don't have to perform any call tracing after all.

## Problem T
> What is output by the following code?
> ```java
House h9 = new House(20);
h9.inc();
h9.add(h9.space());
String s9 = h9.toString();
System.out.print(s9.equals(h9));
```

This is another problem that, on the surface, looks deeply unpleasant to trace through.
Upon deeper analysis, though, we can use some simple intuition about common Java methods to make our lives significantly easier and complete this problem in a short timeframe.
Notice the last line: here, we compare variable `s9`, an instance of `java.lang.String`, to `h9`, an instance of `House`.
Recalling what we know about implementations of `.equals(Object)` in Java classes, one of the first checks that is usually performed is to determine whether the argument is an instance of the class of the calling object.
If this condition does not hold, then the method should return `false` by standard.
This same rule applies to `String`s, as evidenced by the [source code](http://hg.openjdk.java.net/jdk8/jdk8/jdk/file/687fd7c7986d/src/share/classes/java/lang/String.java#l964) for the `String` class as implemented in OpenJDK.
Thus, the result of the call to `s9.equals(h9)` is simply `false`, meaning that the output of this problem is **`false`**.

# Conclusion
I hope you found these sample inheritance problems helpful and learned a mote or two about the Java object model or how to improve your performance on future CS 314 exams.
Working with class hierarchies can be challenging at first, as there are a lot of variables&mdash;and methods&mdash;to consider, but as you gain more experience and begin to work with such structures yourself, you'll find that doing so becomes more natural and rules like constructor chaining become so deeply ingrained in your code-reading process that they seem automatic.
As always, if you're confused about anything you've read here, feel free to reach out and I'll do my best to clarify.
Until then, fair fortune and happy code tracing!
