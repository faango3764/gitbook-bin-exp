# Type Confusion

To find this table of function pointers that are going to tell us "here's the concrete implementation of what we need to execute". Let's first take a look at _vptrs_

## Vptrs <a id="vptrs"></a>

### What is it? <a id="what-is-it"></a>

In a modern language, such as C++, GO, C\#, Rust, all these languages allow the binding of an abstract method deceleration to multiple concrete implementations of that abstract contrasted expressed by the method deceleration.

In C++, there's polymorphism. In languages like Go, C\#, there's this notion of inheritance - a single inheritance method can be implemented by multiple different instances of a particular class or a particular sort of interface-satisfying-type. At runtime, the system has to determine which of these concrete implementation should be executed.

These types of abstraction layers are enabled via indirection. For example, in Go, you can have a type that implements the inheritance. The code which uses an instance of some object is going to interact clearly with that abstract interface but through a layer of indirection. At runtime, we figure out what type of concrete set of instruction should be executed.

This is all very convenient for attackers because by corrupting those indirection tables that help implement the interaction, the attacker can create memory exploits, both read and write exploits.

### Example <a id="example"></a>

Let's take a look at polymorphism in C++

```cpp
class Animal{
public:
    int id;
    virtual char *getNoise(){};
};

class Octupus: public Animal {
public:
    char *getNoise(){
        return "blub";
    }
};

class Seahorse: public Animal{
public:
    char *getNoise(){
        return "wtf";  
    }
};

void f(){
    Octopus *o = new Octupus();
    Animal *a = o;
    a->getNoise(); // Implicity becomes a->vptr->getNoise()
}
```

So here we can see that we have our superclass, `Animal`, which has an int field of `id` and a function called `getNoise()`. The class `Octopus` inherits from the superclass, and defines what it should return for `getNoise()`. This is the same with the class `Seahorse` \(and if anyone wants to say what noise a seahorse makes, that'll be great :\) \)

So if we take a look at the virtual address space of the program, we'll see something along the lines of this

![](https://gblobscdn.gitbook.com/assets%2F-MGOhxJbNhi10jg9Cv-U%2F-MH_H88FEBUVsJF6UEAK%2F-MH_NtzKbfC5QKW-ke9r%2Fvas_type_confusion.png?alt=media&token=2df5c3ec-af06-459d-81ab-0529143210ca)

Virtual address space of the polymorphism code shown above

We see there is the `code` segment, and somewhere in there we're going to have the implementation for `getNoise()` as provided with the `Seahorse` and `Octopus` class. Somewhere in the `static data` segment we're going to have the `vtable` for the two classes. You can think of the `vtable` as being this level of indirection. Each `vtable` is going to point to the functions that are associated with a particular class. So there's one `Octopus vtable`, which is going to point to all the functions that are implemented by `Octopus`. If we look at the `stack` at the top, we can see two variables: `o` and `a`, which we declared in function `f()`. Those pointers point to somewhere in the `heap` to that instance of the `Octopus`. That is the memory that was allocated by the call to `new`. We see that the `heap` representation of the `Octopus` has two things: the `id` which was defined by the superclass, then we have this vpointer that's inside the memory representation of `Octopus`.

How is that helpful? That vpointer is going to point to the vtable for `Octopus`. This means that when `getNoise()` gets called, at runtime, the way which we can tell which `getNoise()` implementation we need to execute is going through the `vpointer`

Let's image that the attacker has an overflowable buffer that is _immediately_ before the `Octopus` object on the `heap`. Imagine that there's a BOF, and now the attacker can corrupt the vpointer and make it point to the `Seahorse vtable`. This is a corruption, as the object that's in the heap that's actually an `Octopus` object, but now it's been corrupted so that it points to a `Seahorse` object. This is known as a _type confusion_.

## Type confusion <a id="type-confusion"></a>

\(Following on from the example\)

The reason that this is a type confusion is because we're corrupted the runtime information to make polymorphism to work correctly. In this particular case, when we look at this polymorphic method here, `a->getNoise()`, then this is going to result in the `Octopus` object invoking the `Seahorse` method. The reason that this'll happen is because the `vpointer` for the `Octopus` object has become corrupt. So in this case the `Octopus` is gonna make the `Seahorse` noise.

### Usage <a id="usage"></a>

These can help generate out-of-bounds memory access - if you were to type confuse to a new class which had a larger buffer than the class you're currently in, you can get a memory read disclosure, ie. you're able to read memory that you shouldn't be able to.

You can also generate a memory write vulnerability - ie. assigning to an object field via a type-confused `this` pointer.

Say that we had this setup on the stack or the heap

```c
+--------+              
| field3 |
+--------+
| field2 |
+--------+
| field1 |
+--------+           +--------+
| field0 |           | field0 |
+--------+           +--------+
|  vptr  |           |  vptr  |
+--------+           +--------+
  Foo                  Bar
   instance              instance
```

We then had some code:

```cpp
void Foo::set_f3(int v){
    this->field3 = v;
}

Bar *b = new Bar();
Foo *f = typeConfuse(b); // function which provides attacker with TC exploit
// Exploit: f actually points to Bar
f->set_f3(42); //this is a non-contigous write vuln
```

Where `set_f3()` sets `field3` to the value that was passed into it. Later in the program, `f` calls `set_f3(42)` which this is a non-contiguous write vulnerability, if the attacker has exploited the TC. Therefore, the stack / heap will look something like this:

​

```c
+--------+           +---------+   
| field3 |           |  field3 |
+--------+           +---------+
| field2 |           | Skipped |
+--------+           +---------+
| field1 |           | Skipped |
+--------+           +---------+
| field0 |           |  field0 |
+--------+           +---------+
|  vptr  |           |   vptr  |
+--------+           +---------+
  Foo                  Bar
   instance              instance
```

So as we can see, we've skipped both `field1` and `field2` in the Bar instance. This is a non-contiguous vulnerability as the attacker doesn't have to overwrite any values in `field1` or `field2` allowing them to just skip past them and write to `field3` directly. Now imagine in `field2` was a canary. This allows the attacker to just completely bypass the canary value, and carry on with their exploit.

