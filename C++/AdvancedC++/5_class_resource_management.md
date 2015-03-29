# Class Resource Management

## Assignment to Self in Assignment Operator
What if we assign to itself?
```C++
dog dd;
dd = dd;  // looks silly
dogs[i] = dogs[j];  // not as silly
```
Let's implement the assignment operator
```C++
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
```
This will not work for self assignment b/c `*rhs.pCollar` references a deleted `pCollar`.

Here is the fix:
```C++
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
```
Here is another solution:
```C++
class dog {
	collar* pCollar;
    dog& operator=(const dog& rhs) {
    	*pCollar = *rhs.pCollar;  // deep copy or collar's operator=
        
        return *this;
    }
};
```
## Resource Acquisition is Initialization (RAII)
Resources are memory, hardware device, network handle, etc.

Consider the following:
```C++
Mutex_t mu = MUTEX_INITIALIZER;

void functionA() {
	Mutex_lock( &mu );
    ... // Do a bunch of things
    Mutex_unlock( &mu );
}
``` 
This code looks fine, except what if after mutex locking, an exception happens? Then the mutex unlocking will never occur! How should we solve this?

Let's make sure the unlocking will always occur when exiting scope.
```C++
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
```
Notice that the resource management is tied to the lifespan of suitable objects.

---

Here is a good example of RAII: `shared_ptr`:
```C++
int function_A() {
	shared_ptr<dog> pd(new dog());
    // the dog is destructed when pd goes out of scope (i.e. no more pointer points to pd)
}
```
Here is another example:
```C++
class dog;
class trick;
void train(shared_ptr<dog> pd, trick dogtrick);
trick getTrick();

int main() {
	train(shared_ptr<dog> pd(new dog()), getTrick());
}
```
What is wrong with the above code?

In C++, when passing expressions as parameters to functions, the order of evaluating which expression first is undeterministic. This means that `getTrick()` may be evaluated before `new dog()` or it may not. And `shared_ptr<dog>(...)` may be evaluated after the former 2! In the case that `new dog()` gets evaluated before `getTrick()` but not yet `shared_ptr`, if `getTrick()` throws an exception, then `shared_ptr` never gets reached, so now we have a dangling pointer.

1. `new dog();`
2. `getTrick();`
3. construct `shared_ptr<dog>`

The solution is to not combine storing objects in shared pointer with other statements:
```C++
int main() {
	shared_ptr<dog> pd(new dog());
    train(pd, getTrick());
}
```
Now our code is thread-safe. Treat shared pointer storage as atomic!!!

---

Here is a final example. What happens when resource management object is copied?
```C++
Lock L1(&mu);
Lock L2(L1);
```
There are 2 solutions

1. Prohibit copying.
2. Reference count the underlying resource with `shared_ptr`

Here is solution 2 more in-depth:
```C++
template<Class Other, class D> shared_ptr(Other *ptr, D deleter);

int main() {
	shared_ptr<dog> pd(new dog());
}
```
Normally, the `deleter` is just `delete`. Let's customize it to `Mutex_unlock`:
```C++
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
```
## Static Initialization Fiasco: Singleton design pattern
What is this problem? Here is a scenario:
```C++
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
```
Here we see `d` calling `c.meow()` in its `d.bark()`. If the above source code is split into multiple files (source/header), then if it happens that `c` is constructed _after_ `d` calls `bark()`, then the problem will crash.

How do you solve this problem? The most common solution is the Singleton Design Pattern.
```C++
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
```
Singleton means treat 2 objects as 1 object, aka a single object:
```C++
int main() {
	Singleton s;  // make sure Singleton's destructor is called by instantiating it on the stack
    
	Singleton::getDog()->bark();
    Singleton::getCat()->meow();
}
```

## Struct vs Class
What are their differences?
```C++
struct Person_t {
	string name;
    unsigned age;
};

class Person {
	string name_;
    unsigned age_;
};
```
1. `struct` members are public; `class` members are private
2. Semantically, `struct` is usually used for _passive_ objects that carry public data (aka a data container). Classes are for active objects that carry private data. Clients use classes for storing complex data structures.

## Resource Managing Classes: deep-copy vs shallow-copy
Here is a pitfall of a class that owns another object through a pointer.
```C++
class Person {
public:
	Person(string name) { pName_ = new string(name); };
    ~Person() { delete pName_; };
    void printName() { cout << *pName_ };
private:
	string *pName_;
};

int main() {
	vector<Person> persons;
    persons.push_back(Person("George"));
    
    persons.back().printName();
    
    cout << "Goodbye" << endl;
    return 0;
}
```
Note that `Person` owns `pName_`.

The above code at `printName()`... why? Let's look at `push_back()`:

1. "George" is constructed
2. **A copy of "George" is saved in the vector persons.**
3. **"George" is destroyed.**

You can see that the "George" inside the vector is NOT the "George" created in `main` scope. The "George" inside the vector is a **shallow copy** of the "George" in `main` scope. Since shallow copy doesn't create a copy of the pointees of pointers, the (shallow) copy of `pName_` still points to the same exact memory address. So when the original `pName_` is deleted, the copied `pName_` is pointing to an invalid memory address! The program will crash. To solve this, we need to manually perform deep-copy:
```C++
class Person {
public:
	Person(string name) { pName_ = new string(name); };
    ~Person() { delete pName_; };
    Person(const Person &rhs) { pName_ = new string(*(rhs.pName())); };
    Person& operator=(const Person &rhs) { pName_ = new string(*(rhs.pName())); };
    void printName() { cout << *pName_ };
    string *pName() { return pName_; }
private:
	string *pName_;
};
```
Another solution is to just delete the copy constructor and copy assignment operator:
```C++
private:
	Person(const Person &rhs);
    Person &operator=(const Person &rhs);
```
---

**Note**: STL containers require that stored objects be copy-constructable and copy-assignable. If you don't want to implement these constructors, you can just `push_back` pointers to objects instead. Just make sure to delete the objects yourself later on.

Implicit copying is where bugs are most likely introduced. So be careful! To be safe, just disable compiler generated copy constructor or copy assignment operator.


## (Virtual) `clone()` Function
You can implement a `clone()` function to copy manually and explicitly (instead of implicitly).
```C++
class Dog;
class Yellowdog : public Dog;

void foo(Dog *d) {  // d is a Yellowdog
	Dog *c = new Dog(*d);  // c is a Dog
}

int main() {
	Yellowdog d;
    foo(&d);
}
```
Poly-morphism "broke" our code! How to solve? Let's use `virtual` to preserve poly-morphism:
```C++
class Dog {
public:
	virtual Dog* clone() { return (new Dog(*this)); };  // co-variant return type
};

class Yellowdog : public Dog {
public:
	virtual Yellowdog* clone() { return (new Yellowdog(*this)); };
};

void foo(Dog *d) {  // d is a Yellowdog
	Dog* c = d->clone();  // c is a Yellowdog
}

int main() {
	Yellowdog d;
    foo(&d);
}
```
`virtual` makes sure that the original type's function is called, no matter what type the object was later casted into. This technique of preserving poly-morphism is called "co-variannce".
