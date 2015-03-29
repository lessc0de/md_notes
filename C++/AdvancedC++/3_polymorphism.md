# Polymorphism

## Public, Protected, and Private Inheritance
These 3 types of inheritance specify different access control from the derived class to the base class.

1. `public` - any class/function may access the method/property
2. `protected` - only this class and any subclasses may access the method/property
3. `private` - only this class may access the method/property. It won't be inherited.

.
```C++
class B{};
class D_priv : private B{};
class D_prot : protected B{};
class D_pub : public B{};
```

1. None of the derived classes can access anything that is private in `B`.
2. `D_pub` inherits public members of `B` as public and protected members of `B` as protected.
3. `D_priv` inherits the public and protected members of `B` as private.
4. `D_prot` inherits the public and protected members of `B` as protected.

Casting:

1. Anyone can cast a `D_pub*` to `B*`. `D_pub` **is-a** `B`.
2. `D_priv`'s members and friends can cast a `D_priv*` to `B*`.
3. `D_prot`'s members, friends, and children can cast a `D_prot*` to `B*`

More details:
```C++
class B {
public:
	void f_pub() {};
protected:
	void f_prot() {};
private:
	void f_priv() {};
};

class D_pub : public B {
public:
	void f() {
    	f_pub();	// OK
        f_prot();	// OK
        f_priv();	// Error
    }
};

class D_prot : protected B {
public:
	void f() {
    	f_pub();	// OK
        f_prot();	// OK
        f_priv();	// Error
    }
};

class D_priv : private B {
public:
	void f() {
    	f_pub();	// OK
        f_prot();	// OK
        f_priv();	// Error
    }
};

int main() {
	D_pub D1;
    D1.f_pub();		// OK
    
    D_prot D2;
    D2.f_pub();		// Error. f_pub is D_prot's protected function
    
    B* pB = &D1;	// OK
    pB = &D2;		// Error (not is-a relationship)
}
```
### Public vs Private Inheritance
Public inheritance gives an `is-a` relationship, e.g., `D_pub` **is-a** `B`.  
Private inheritance is similar to a **has-a** relationship.

Private inheritance can satisfy a **has-a** relationship, but it is not recommended. An explicit **has-a** is better:
```C++
class ring {
public:
	void tinkle() { ... };
};

/* Composition (has-a satisfied explicitly) */
class dog {
	ring m_ring;
    public:
    	void tinkle() { m_ring.tinkle(); }  // call-forwarding
};

/* Private Inheritance (has-a satisfied implicitly) */
class dog : private ring {
public:
	using ring::tinkle;  // expose private method to the public
};
```
Only use private inheritance for what it's for: **poly-morphism** (aka virtual function overriding).

## Maintain **is-a** Relation for Public Inheritance
A derived class should be able to do everything the base class can do. This is known as **is-a** relation. In real-world programming, sometimes this relation is hard to maintain. A penguin, for example, is a bird, but it can't do everything birds can do, namely **FLY!**
```C++
class bird {
public:
	void fly();
};

class penguin : public bird {};

int main() {
	penguin p;
    p.fly();
}

We can solve this by:

class bird;
class flyablebird : public bird {
public:
	void fly();
};
class penguin : public bird{};
```
Here is a 2nd example:
```C++
class dog;
class yellowdog : public dog;

int main() {
	yellowdog* py = new yellowdog();
    py->bark();
    dog* pd = py;
    pd->bark();
}
``` 
The above will output:

	Woof! I am a yellowdog. <-- (yellowdog*)pd->bark()
    Woof! I am a dog.		<-- (dog*)pd->bark()

Hmm... this isn't how poly-morphism is supposed to work! The behavior isn't consistent. Never override non-virtual functions. Otherwise, specify the function as `virtual`!
```C++
class dog {
public:
	virtual void bark(string msg = "dog") { cout << "Woof! I am a " << msg << endl; }
};
class yellowdog : public dog {
public:
	virtual void bark(string msg = "yellow dog") { cout << "Woof! I am a " << msg << endl; }
};
```
Output:

	Woof! I am a yellowdog. <-- (yellowdog*)pd->bark()
    Woof! I am a dog.		<-- (dog*)pd->bark()

Why hasn't the above worked? This is b/c we're using default parameters, which are bound during compile-time, not run-time. So in `yellowdog`'s `bark()` call, the default value of `msg` was already bound to "dog" from the base class's default parameter value.

---

In summary, there are 4 lessons to be learned.

1. Precisely define classes.
2. Don't override non-virtual functions.
3. Don't override default parameter values for virtual functions.
4. Force inheritance of shadowed functions.

Here is an example of #4.
```C++
class dog {
public:
	virtual void bark(string msg) {}
    void bark(int i) {}
};

class yellowdog : public dog {
public:
	virtual void bark(string msg) {}
};

int main() {
	yellowdog* py = new yellowdog();
    py->bark("hi");
    py->bark(5);
}
```
`py->bark("hi")` will work. `py->bark(5)` won't b/c `void bark(int i)` is shadowed by `yellowdog`'s `virtual void bark(string msg)` function. We need to re-expose the shadowed function.
```C++
class yellowdog : public dog {
public:
	using dog::bark;
    virtual void bark(string msg) {}
};
 ```   
Now `yellowdog` can use the `int` version of the overloaded `bark` method.


## Static Polymorphism
When we talk about polymorphism, we're by default talking about dynamic polymorphism:
```C++
struct TreeNode { TreeNode *left, *right; };

class Generic_parser {
public:
	void parse_preorder(TreeNode* node) {
    	if (node) {
        	process_node(node);
            parse_preorder(node->left);
            parse_preorder(node->right);
        }
    }
private:
	virtual void process_node(TreeNode* node) { }
};

class EmployeeChart_Parser : public Generic_Parser {
private:
	void process_node(TreeNode* node) {
    	cout << "Customized process_node() for EmployeeChart." << endl;
    }
};

int main() {
	EmployeeChart_Parser ep;
    ep.parse_preorder(root);  // polymorphism!
}
```    
Dynamic Polymorphism comes with a cost. It needs memory for a "virtual" table, and it costs some run-time for dynamic binding (looking up the virtual table). If dynamic casting is a critical portion of your code, you should optimize this with static polymorphism! Static polymorphism "simulates" dynamic polymorphism without paying the virtual price.

Requirements for static polymorphism:

1. **is-a** relationship between class and derived class
2. Base class defines a "generic" algorithm that's used by the derived class
3. The "generic" algorithm is customized by the derived class

Here is the solution:
```C++
template <typename T>
class Generic_Parser {
public:
	void parse_preorder(TreeNode* node) { }
	void process_node(TreeNode* node) {
    	static_cast<T*>(this)->process_node(node);
    }
};

class EmployeeChart_Parser : public Generic_Parser<EmployeeChart_Parser> {
public:
	void process_node(TreeNode* node) { }
};

int main() {
	EmployeeChart_Parser ep;
    ep.parse_preorder(root);
}
```
You can see, instead of using `virtual`, we use `static_cast<T*>(this)` instead! All type bindings are performed during compile time.

_Note: Static polymorphism is also known as simulated polymorphism, curiously recurring template pattern, or template metaprogramming._


### Free Lunch?
There are some costs to static polymorphism. Consider the following:
```C++
int main() {
	EmployeeChart_Parser ep;
    MilitaryChart_Parser mp;
}
```
`EmployeeChart_Parser` and `MilitaryChart_Parser` both take up space, making the program size even larger. Is the tradeoff worth it? It depends on your application.


## Multiple Inheritance: Diamond Shape Hierarchy and Interface Segregation
A class is directly derived from multiple base classes.
```C++
class InputFile {
public:
	void read();
    void open();
};

class OutputFile {
public:
	void write();
    void open();
};

class IOFile : public InputFile, public OutputFile {};

int main() {
	IOFile f;
}
``` 
Both `InputFile` and `OutputFile` have `void open()`. Which one do we use?
```C++
int main() {
	IOFile f;
    f.OutputFile::open();
    // or
    f.InputFile::open();
}
```
We can try avoiding the above by basing `InputFile` and `OutputFile` off of another base class: `File`.
```C++
class File {
public:
	string name;
    void open();
};

class InputFile : public File {};
class OutputFile : public File {};

class IOFile : public InputFile, public OutputFile {};  // diamond shape of hierarchy

int main() {
	IOFile f;
    f.open();  // compiler error
}
```
This will still give a compiler error b/c `IOFile` has 2 copies of `open`, 1 from `InputFile` (which itself got `open` from `File`), and 1 from `OutputFile` (which itself got `open` from `File`).

### C++'s solution to "Diamond Shape Hierarchy"
```C++
class InputFile : virtual public File {};
class OutputFile: virtual public File {};
class IOFile : public InputFile, public OutputFile {};

int main() {
	IOFile f;
    f.open();
}
```
Great! The `virtual` keyword means we want only 1 instance of `File`'s members.  
Wait.. not so great! Virtual inheritance introduces another problem!
```C++
class File {
public:
	File(string fname);
};

class InputFile : virtual public File {
public:
	InputFile(string fname) : File(fname) {}
};

class OutputFile : virtual public File {
public:
	OutputFile(string fname) : File(fname) {}
};

class IOFile : public InputFile, public OutputFile {
public:
	IOFile(string fname) : OutputFile(fname), InputFile(fname) {}
};

int main() {
	IOFile f;
}
```
Above, we have defined a non-default constructor for `File`. That means the compiler will not construct a default constructor for any of `File`'s derived classes. Also, look above and see - we've manually defined constructors. This causes trouble here:

	IOFile(string fname) : OutputFile(fname), InputFile(fname) {}
    
As you can see, we've instantiated 2 objects - 1 `OutputFile` and 1 `InputFile`. This is bad! C++ solves this by defining a rule: **the initialization of the base virtual class is the responsibility of the most derived class. In our case, `IOFile` is the most derived class of the base class `File`**:

	IOFile(string fname) : OutputFile(fname), InputFile(fname), File(fname) {}
    
This solution looks awkward, but we have to live with it if we want multiple inheritance and polymorphism.

### Why Multiple Inheritance? Because of Interface Segregation.
If multiple inheritance is so ugly to use, why bother? The answer lies in what's called the **Interface Segregation Principle**, which states

> Split large interfaces into smaller and more specific ones so that clients only need to know about the methods that are of interest to them.

Here is an example of the above principle:
```C++
class Engineer {
public:
	...; // 40 APIs. Not very difficult.
};

class Son {
public:
	...; // 50 APIs. Not very difficult.
};

class Andy : public Engineer, Son {
public:
	...; // 500 APIs! Very difficult!
};
```  
Here, we see that Andy is an Engineer and a Son. To Andy's co-workers, he is just an Engineer, so people will work with him through the `Engineer` interface/API. To Andy's parents, he is a Son, so his parents will work with him through the `Son` interface/API. This de-coupling makes is great, because we have separate interfaces for different clients who interact with Andy for different purposes.

### Pure Abstract Classes
C++ provides the concept of abstract classes, which are classes that have one or more pure virtual functions.

A Pure Abstract Class is a class which contains ONLY pure virtual functions

* no data members
* no implemented functions

The Pure Abstract Class provides an interface; it provides zero implementation.
```C++
class OutputFile {
public:
    virtual void write() = 0;
    virtual void open() = 0;    
};
```

## The Duality of Public Inheritance: Inheritance of Interface and Implementation
When we talk about public inheritance, we can either refer to inheritance of an interface, or inheritance of an implementation.
```C++
class Dog {
public:
	virtual void bark() = 0;
    void run() { cout << "I am running." << endl; }
    virtual void eat() { cout << "I am eating. " << endl; }
protected:
	void sleep() { cout << "I am sleeping..." << endl; }
};

class Yellowdog : public Dog {
public:
	virtual void bark() { cout << "I am a yellow dog." << endl; }
};
```
`Yellowdog` will inherit the interface AND implementation of `run()`. It will inherit only the interface of `bark()`.  
If `Yellowdog` overrides `eat()`, it inherits the interface of `eat()`. If it doesn't override, it will inherit the interface AND implementation of `eat()`.


Does this mean that if the implementation is inherited, then the interface is automatically inherited? No. Look at `sleep()`, which is a protected method in the base `Dog` class.

`sleep()`'s implementation is inherited. That means `Yellowdog` can _use_ `sleep()`. But clients of `Yellowdog` may not use `sleep()`, since `Yellowdog` protected it: it doesn't provide an interface for `sleep()`. In other words, `Yellowdog` inherits the implementation, not the interface, of `sleep()`. You can, of course, expose `sleep()` to the public:
```C++
class Yellowdog : public Dog {
public:
	void iSleep() { sleep(); }
    // or
    using Dog::sleep;
};
```
---

In summary,

1. Pure virtual public function - inherit interface only.
2. Non-virtual public function - inherit both interface and implementation.
3. Impure virtual public function - inherit interface and default implementation if not overrided
4. Protected function - inherit implementation only

### Interface vs Implementation Inheritance
Use interface inheritance for sub-typing or polymorphism:
```C++
virtual void bark() = 0;
// beware of interface bloat
``` 
Implementation inheritance isn't encouraged because it increases code complexity.

---

Guidelines for Implementation Inheritance:

1. **Do not use inheritance for code reuse; use composition instead!** (See the next topic)
2. Minimize the implementation in base classes. Base classes should be thin.
3. Minimize the level of hierarchies in implementation inheritance.


## Code Reuse: (Private) Inheritance [is-a] vs Composition [has-a]
Here is an example of inheritance for code re-use:
```C++
/* Code Re-use with Inheritance */
class BaseDog {
	// common activities
};

class BullDog : public BaseDog {
	// call the common activities to perform more tasks
};

class SheperdDog : public BaseDog {
	// call the common activities to perform more tasks
};
 ```   
This doesn't look good. `BaseDog` isn't a good name. It reveals implementation details. It reveals that `BaseDog` is used to derive other Dogs. This isn't a problem if `BaseDog` isn't revealed in the public interface. A solution is to just rename `BaseDog` to `Dog`.

Also, it seems to be using inheritance for code-reuse! This is very complex to develop. You have to re-factor code so many times if implementation details change. Also, inheritance should describe **is-a** relations. The above code really doesn't do a good job of that. Wouldn't it be better to use composition instead? Composition emphasizes a **has-a** relationship.
```C++
/* Code Re-use with Composition */
class ActivityManager {
	// common activities
};

class Dog {
	...
};

class BullDog : public Dog {
	ActivityManager* pActMngr;
    // call the common activities through pActMngr
};

class SheperdDog : public Dog {
	ActivityManager* pActMngr;
    // call the common activities through pActMngr
};
```
Why is the above better for code reuse?

1. With inheritance, the code reuser is heavily coupled with the reuseable code.
  * child class automatically inherits ALL parent class's public members (child is heavily coupled to parent)
  * child class can access parent's protected members (child can access parent's internals)
    * this breaks encapsulation (privacy)
2. With composition, you can enable dynamic binding!
  * Inheritance is bound at compile time (relationship can't be changed in run-time)
  * Composition can be bound either at compile time or at run time (relationship can be changed during run-time)
  
More on point #2:
```C++
class OutdoorActivityManager : public ActivityManager { ... }
class IndoorActivityManager : public ActivityManager { ... }
// since each dog has a pointer to ActivityManager, we can take advantage of dynamic binding to specialize
// the activity manager during run-time
```
3. Composition has flexible code construction

.

    Dog		ActivityManager
    BullDog	OutdoorActivityManager
    SheperdDog	IndoorActivityManager
    WeirdDog	OutdoorActivityManager, IndoorActivityManager (can have multiple)
