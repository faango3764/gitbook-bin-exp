# Challenges of Precise CFG

Let's say that we had some C functions

```c
void lt_sort(int *arr, int len){
    sort(arr, len, less_than);
}

​void gt_than(int *arr, int len){
    sort(arr, len, greater_than);
}
```

So these two functions do a similar thing, where they take in a pointer to an array, and a length, and then it'll call the sort function, and it'll sort the array base on that third argument, `less_than` or `greater_than`

If we take a look at the rest of the code, we get this

```c
bool less_than(int x, int y){ ... };

bool greater_than(int x, int y){ ... };​

void sort(int *arr, int len, comp_func_t cmp){
    if(cmp(a[i], a[i+1]))
        { ... }
    ...
}
```

## Compiler enforced static CFI <a id="compiler-enforced-static-cfi"></a>

So what the compiler would do is it would find every instruction that can be a legitimate target for an indirect branch, and assign each one of those a unique ID.

The compiler is also going to associate every indirect branch instruction with a set of valid target IDs. The compiler will then instrument every indirect branch to determine whether the current jump is valid. So in other words, the compiler is essentially going to inject new instructions that the developer didn't put there, to introspect the jump target, and is going to ask "Hey, is this jump tagret fall on the set of valid target IDs?"

If we look at the control flow graph for the C code, you'll see something like what's below

![](https://gblobscdn.gitbook.com/assets%2F-MGOhxJbNhi10jg9Cv-U%2F-MH0UI2AWjOqELeqhYtk%2F-MH0YsKrzH7k1OCKAB9I%2Fcf_graph.PNG?alt=media&token=ea589e25-1460-4ce9-989d-d2b4ea1c39b6)

So as you can see, we get light arrows signifying direct flow, and heavier arrows signifying indirect flow. We can tell, say the top left `lt_sort()` has a direct flow to the top of the `sort()` function because we know, with complete certainty, that we're going to be calling the `sort()` function.

 In contrast, if we look at the top of the `sort()` function, we see an indirect control flow to the `less_than()` boolean. Depending o what function was passed in as an argument, we're going to jump to one of two possible targets, either `less_than()` or `greater_than()`.

So what are some examples of Compiler-inserted CFI checks?

* `less_than()::ret` can only return to `id2`
* `sort()::call *cmp` can only jump to
  * `id3` if control flow originates from `lt_sort()`
  * `id4` if control flow originates from `gt_sort()`

 So we could say the checks are dependant on the control flow history

### Challenge - expensive! <a id="challenge-expensive"></a>

Tracking cross jump control flow is expensive. This is because compiler-injected instrumentation might need to access data structures many times for each jump. So you could imagine that the compiler-injected instrumentation is actually using some type of call stack for example, to determine where a call instruction should go to. Well it could look at the stack, and say "Ok, I came through the `lt_sort()` function, so therefore the only valid target for this call is the `less_than()` boolean"

The problem here is that jumps are very common, so if we were going to check a full blown data structure for every control flow, and maintaining that data structure whenever we transfer the value of `rip` to some different value, that would be very expensive.

###  CFI implemented in the real world? <a id="cfi-implemented-in-the-real-world"></a>

A real compiler-based CFI often relaxes some constraints, so sometimes it will allow a control flow to happen, despite it technically not allowed by the precise control flow graph.

For example, many compiler-based control schemes allow a ret to return to any call-preceded instruction. If we look at `lt_sort()`, there's a call instruction, and then an `id0`, which is just some instruction. The instruction at `id0` is a call-preceded instruction. If we relax the control flow graph that we're trying to enforce, the compiler is only going to inject control flow instrumentation if a `ret` returns to a call-preceded instruction. So if we take a look at `less_than()::ret`, it would not just be able to return to `sort()::id2`, but also `lt_sort()::id0`

Another common relaxation technique is that a `jmp` can go to any function start, or any allowable basic box, where it might be the start of a c level control statement, such as an if... else, while loop etc.

With all these relaxation techniques, it means that the compiler has to do less checks. This ameans that the checks are going to be less process heavy.

### Is relaxed CFI useful? <a id="is-relaxed-cfi-useful"></a>

A lot of people use relaxed CFI as:

* It has no false positives
* It has some false negatives \(eg. `sort()::call *cmp` can jump to the start of any function\)

It's better than nothing!

* For example, `sort()::call *cmp` cannot jump to the middle of any function
* Because most people don't want false positives, they don't want many processing overtime, so they mainly use relaxed CFI

