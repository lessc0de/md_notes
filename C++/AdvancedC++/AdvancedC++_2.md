# Advanced C++
## Compiler Generated Functions
If you don't declare the following explicitly, they will be implicitly defined/generated.

1. Copy Constructor
2. Copy Assignment Constructor
3. Default Constructor (iff no constructor is defined)
4. Destructor

The following class

	class Dog {};

is equivalent to

	class Dog {
    	public:
        	dog(const dog& rhs) {...};  // 1. Copy Constructor (deep-copy)
            
            dog& operator=(const dog& rhs) {...};  // 2. Copy Assignment Constructor (deep-copy, but used with '=')
            
            dog() {...};  // 3. Default Constructor (call base class's constructor)
            
            ~dog() {...};  // 4. Destructor (call base class's destructor)
    };

If the constructor/destructors cannot be generated, they will not be generated then.

---

Let's look at a less trivial class.

	class Dog {
   	public:
    	string m_name;
        
        dog(string name="Bob") {m_name=name}
        ~dog() {cout << "bye" << endl;}
    }
    
    int main() {
    	Dog dog1("Henry");
        Dog dog2;
        dog2 = dog1;
    }

The following constructor/destructors are generated:

1. Copy Constructor - no
2. Copy Assignment Constructor - yes
3. Default Constructor - no b/c default already defined
4. Destructor - no

---

Let's look at another case:

	class Collar {
    	public:
        Collar(string color) { cout << "collar is born." << endl; }
    };
    
    class Dog {
    	Collar m_collar;
    };

	int main() {
    	Dog dog1;
    }

The compiler will generate a default constructor for `Dog`. In this default constructor, it will call the default constructor of its members - `Collar m_collar` in this case. `Collar` doesn't have a default constructor, and one will not be generated b/c `Collar` already has a constructor. Oops, there is a compile error.

### Note on reference members
If a member of a class is a reference (e.g. `string& m_name`), the compiler cannot generate any constructor. Why? C++ requires any reference variable to be initialized. Compiler generated constructors construct its members, but they don't initialize them.

### "Explicitly" generating constructors/destructors
If you want to explicitly generate a constructor/destructor, use `default`.

	class Dog {
   	public:
    	dog() = default;
        dog(string name) { ... };
    };


## Disallow Functions
The compiler can generate

1. Copy Constructor
2. Copy Assignment Operator
3. Destructor
4. Default Constructor

You can disallow generated constructors.

    class OpenFile{
    public:
        OpenFile(string filename) { cout << "Opening file " << filename << endl; }
    };
    
    int main() {
        OpenFile f(string("Bo_file"));
        OpenFile f2(f);
    }

Uh-oh. Our compiler generated a copy constructor. But we don't want to have 2 handles on the same file! Let's disallow copy-constructor with `delete`.

    class OpenFile{
        public:
            OpenFile(string filename) { cout << "Opening file " << filename << endl; }
            OpenFile(OpenFile& rhs) = delete;
            OpenFile& operator=(const OpenFile &rhs) = delete;
        };
        
        int main() {
            OpenFile f(string("Bo_file"));
            OpenFile f2(f);
        }

You can also disallow constructors/destructors by declaring them in `private`.

	class OpenFile {
    private:
    	OpenFile(const &rhs);
    }

---

Disallowing through the `private` technique is especially useful for reference-counting shared pointers:

	class RefCountPointer {
    public:
    	...
    private:
    	~RefCountPointer() {...}
    };

	int main {
    	RefCountPointer r;
    }

The above will not compile b/c `r` cannot destruct after leaving scope. This is great for reference-counted pointers, where shared pointers should not be able to destruct each other.

The only way to destruct is this:

	class RefCountPointer {
    public:
    	...
        void destroyMe() { delete this; }
    private:
    	~RefCountPointer() { cout << "destructed..." << endl; }
    };

	int main {
    	RefCountPointer *r = new RefCountPointer();
        r->destroyMe();
    }

Note that we allocate `RefCountPointer` on the heap, since if it were created on the stack, then `r` will be automatically destructed when the stack unwinds (leave scope). This is bad b/c now, you're attempting to destruct twice. Also, in the 2nd time, `~RefCountPointer` is private, so it wouldn't compile.

## Virtual Destructor and Smart Destructor
Let's use "virtual" destructors in polymorphic base classes.

	class Dog {
    public:
    	~Dog() { cout << "Dog destroyed" << endl; }
    };
    
    class Yellowdog : public Dog {
    public:
    	~Yellowdog() { cout << "Yellow dog destroyed." << endl; }
    };
    
    class DogFactory {
    public:
    	static Dog* createYellowDog() { return (new Yellowdog()); }
        // create other dogs
    };
    
    int main() {
    	Dog* pd = DogFactor::createYellowDog();
        // Do something with pd
        
        delete pd;
        return 0;
    }

When the above is called, only `Dog`'s destructor is invoked, since the type of `createYellowDog` is `Dog*`, not `Yellowdog*`.  
**Thus, if a class is meant to be used in a polymorphic way, then the base class MUST have a virtual destructor.**

	class Dog {
    public:
    	virtual ~Dog() { ... }
    }

---

Instead of virtualizing the base destructor, we can use `shared_ptr`:

	class Dog {
    public:
    	~Dog() { ... }
    };

	class DogFactory {
    public:
    	static shared_ptr<Dog> createYellowDog() {
        	return shared_ptr<Yellowdog>(new Yellowdog());
        }
    };

	int main() {
    	shared_ptr<Dog> pd = DogFactor::createYellowDog();
        //delete pd;
        return 0;
    }

We can also used shared pointers. With `shared_ptr`, the shared pointer itself is responsible for destruction, and since we wrapped `new Yellowdog()` with `shared_ptr<Yellowdog>`, the shared pointer knows to use `Yellowdog`'s destructor when it is time to be destroyed.

Note that we casted `shared_ptr<Yellowdog>` to `shared_ptr<Dog>`. This is allowed b/c shared pointers, like all C++ classes, support polymorphism.

_Note: Use shared pointers. Unique pointers won't work._

_Note: All classes in STL have no virtual destructors. Be careful!_


## Exceptions in Destructors
We need to prevent exceptions from leaving destructors! We don't want memory leak when exceptions (run-time) occur!

	class Dog {
    public:
    	string m_name;
        Dog(string name) {m_name = name;}
        void bark() {}
    };
    
    int main() {
    	try {
        	Dog dog1("Henry");
            Dog dog2("Bob");
            throw 20;
        } catch (int e) {
        	cout << e << " is caught" << endl;
        }
    }
    
OUTPUT:

	Henry is born.
    Bob is born.
    Bob is destroyed.
    Henry is destroyed.
    20 is caught.

You can see the `Dog`s are destroyed when the `try` block is exited.

What if we threw an exception in the destructor?

	~Dog() { throw 20; }
    
OUTPUT:

	Henry is born.
    Bob is born.
    CRASH!

### Solution 1: Destructor swallow the exception.

	~dog() {
    	try {
        	// Enclose all exception-prone code here
        } catch (MYEXCEPTION e) {
        	// Catch exception
        } catch (...) {
        	// Beware: default catch is dangerous
        }
    }
	
### Solution 2: Move exception-prone code to different function

	void prepareToDestruct() { throw 20; }
    ~Dog() = default;
    
Now in `main`:

	try {
        Dog dog1("Henry");
        Dog dog2("Bob");
        dog1.bark();
        dog2.bark();
        dog1.prepareToDestruct();
        dog2.prepareToDestruct();
    } except (int e) {
    	...
    }


## Virtual Function in Constructor or Destructor
The `virtual` keyword invokes the method of the immediate scope, not the scope it was defined at:

	class Dog [
    public:
    	...
        void bark() { cout << "bark!" << endl; }
        void seeCat() { bark(); }
    };
    
    class Yellowdog : public Dog {
    public:
    	...
        void bark() { cout << "woof!" << endl; }
    };
    
vs

	class Dog [
    public:
    	...
        virtual void bark() { cout << "bark!" << endl; }
        void seeCat() { bark(); }
    };
    
    class Yellowdog : public Dog {
    public:
    	...
        virtual void bark() { cout << "woof!" << endl; }
    };

In `main`:

	Yellowdog d;
    d.seeCat();
    
`seeCat` in the former scenario will print "bark!", while in the latter scenario, it will print "woof!".

`virtual` enables polymorphism aka dynamic binding.

### Virtual Function inside Constructor or Destructor
Virtual functions don't follow the immediate scope when called within constructor or destructor. Why? For instance, in `Dog`:

	Dog() { bark(); }
    
and in `main`:

	Yellowdog d;

When `d` is constructed, it first calls its parent's constructor. When in the parent's constructor, it sees `bark()`, but `bark()`, even though it's virtual, hasn't been defined yet for `YellowDog`, so C++ defaults to `Dog`'s `bark()` method.

**Solution**: Don't call virtual functions inside constructors/destructors.


## Assignment to Self in Assignment Operator
What if we assign to itself?

	dog dd;
    dd = dd;  // looks silly
    dogs[i] = dogs[j];  // not as silly

Let's implement the assignment operator

	class collar;
    class dog {
    	collar* pCollar;
        dog& operator=(const dog& rhs) {
        	collar* pOrigCollar = pCollar;
            pCollar = new collar(*rhs.pCollar);
            delete pOrigCollar;
            return *this;
        }
    };

This will not work for self assignment b/c `*rhs.pCollar` references a deleted `pCollar`.

Here is the fix:

	class collar;
    class dog {
    	collar* pCollar;
        dog& operator=(const dog& rhs) {
        	if (this == &rhs)
            	return *this;
            
        	collar* pOrigCollar = pCollar;
            pCollar = new collar(*rhs.pCollar);
            delete pOrigCollar;
            return *this;
        }
    };
    
Here is another solution:

	class dog {
    	collar* pCollar;
        dog& operator=(const dog& rhs) {
        	*pCollar = *rhs.pCollar;  // deep copy or collar's operator=
            
            return *this;
        }
    };

## Resource Acquisition is Initialization (RAII)
Resources are memory, hardware device, network handle, etc.

Consider the following:

	Mutex_t mu = MUTEX_INITIALIZER;
    
    void functionA() {
    	Mutex_lock( &mu );
        ... // Do a bunch of things
        Mutex_unlock( &mu );
    }
    
This code looks fine, except what if after mutex locking, an exception happens? Then the mutex unlocking will never occur! How should we solve this?

Let's make sure the unlocking will always occur when exiting scope.

	Mutex_t mu = MUTEX_INITIALIZER;

	class Lock {
  	private:
    	Mutex_t* m_pm;
    public:
    	explicit Lock(Mutex_t *pm) { Mutex_lock(pm); m_pm = pm; };
        ~Lock() { Mutex_unlock(m_pm); };
    };
    
    void functionA() {
    	Lock mylock(&mu);
        ... // do a bunch of things
        // the mutex will be released once stack unwinds due to function exit or exception
    }

Notice that the resource management is tied to the lifespan of suitable objects.

---

Here is a good example of RAII: `shared_ptr`:

	int function_A() {
    	shared_ptr<dog> pd(new dog());
        // the dog is destructed when pd goes out of scope (i.e. no more pointer points to pd)
    }

Here is another example:

	class dog;
    class trick;
    void train(shared_ptr<dog> pd, trick dogtrick);
    trick getTrick();
    
    int main() {
    	train(shared_ptr<dog> pd(new dog()), getTrick());
    }

What is wrong with the above code?

In C++, when passing expressions as parameters to functions, the order of evaluating which expression first is undeterministic. This means that `getTrick()` may be evaluated before `new dog()` or it may not. And `shared_ptr<dog>(...)` may be evaluated after the former 2! In the case that `new dog()` gets evaluated before `getTrick()` but not yet `shared_ptr`, if `getTrick()` throws an exception, then `shared_ptr` never gets reached, so now we have a dangling pointer.

1. `new dog();`
2. `getTrick();`
3. construct `shared_ptr<dog>`

The solution is to not combine storing objects in shared pointer with other statements:

	int main() {
    	shared_ptr<dog> pd(new dog());
        train(pd, getTrick());
    }

Now our code is thread-safe. Treat shared pointer storage as atomic!!!

---

Here is a final example. What happens when resource management object is copied?

	Lock L1(&mu);
    Lock L2(L1);
    
There are 2 solutions

1. Prohibit copying.
2. Reference count the underlying resource with `shared_ptr`

Here is solution 2 more in-depth:

	template<Class Other, class D> shared_ptr(Other *ptr, D deleter);
    
    int main() {
    	shared_ptr<dog> pd(new dog());
    }

Normally, the `deleter` is just `delete`. Let's customize it to `Mutex_unlock`:

	class Lock {
    private:
    	shared_ptr<Mutex_t> pMutex;
    public:
    	// the second param of shared_ptr constructor is "deleter" function.
    	explicit Lock(Mutex_t *pm) : pMutex(pm, Mutex_unlock) {
        	Mutex_lock(pm);
        };
    };
    
    int main() {
    	Lock L1(&mu);
        Lock L2(L1);
    }

## Static Initialization Fiasco
What is this problem? Here is a scenario:

	class Dog {
    	string _name;
    public:
    	void bark();
        Dog(char *name);
    };
    
	class Cat {
    	string _name;
    public:
    	void meow();
        Cat(char *name);
    };
    
    void Dog::bark() {
    	cout << "Dog rules! My name is " << _name << endl;
        c.meow(); 
    }
    
    void Cat::meow() {
    	cout << "Cat rules! My name is " << _name << endl;
    }
    
    Dog d("Gunner");  // <-- GLOBAL/STATIC INITIALIZED DOG
    Cat c("Smokey");  // <-- GLOBAL/STATIC INITIALIZED DOG
    
    int main() {
    	d.bark();
        return 0;
    }

Here we see `d` calling `c.meow()` in its `d.bark()`. If the above source code is split into multiple files (source/header), then if it happens that `c` is constructed _after_ `d` calls `bark()`, then the problem will crash.

How do you solve this problem? The most common solution is the Singleton Design Pattern.

	class Dog;
    class Cat;
    
    class Singleton {
    	static Dog* pd;
        static Cat* pc;
    public:
    	~Singleton();
        
        static Dog* getDog();
        static Cat* getCat();
    };
    
    Dog* Singleton::pd = 0;
    Cat* Singleton::pc = 0;
    
    Dog* Singleton::getDog() {
    	if (!pd)
        	pd = new Dog("Gunner");  // Initialize Upon First Usage Idiom
        return pd;
    }

    Cat* Singleton::getCat() {
    	if (!pc)
        	pc = new Cat("Smokey");
        return pc;
    }
    
    Singleton::~Singleton() {
    	if (pd) delete pd;
        if (pc) delete pc;
        pd = 0;
        pc = 0;
    }

Singleton means treat 2 objects as 1 object, aka a single object:

	int main() {
    	Singleton s;  // make sure Singleton's destructor is called by instantiating it on the stack
        
    	Singleton::getDog()->bark();
        Singleton::getCat()->meow();
    }

