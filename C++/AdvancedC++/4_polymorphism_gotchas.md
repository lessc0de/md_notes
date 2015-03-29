# Polymorphism Gotchas!

## Polymorphism REQUIRES Virtual Destructor or Smart Destructor
Let's use "virtual" destructors in polymorphic base classes.
```C++
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
```
When the above is called, only `Dog`'s destructor is invoked, since the type of `createYellowDog` is `Dog*`, not `Yellowdog*`.  
**Thus, if a class is meant to be used in a polymorphic way, then the base class MUST have a virtual destructor.**
```C++
class Dog {
public:
	virtual ~Dog() { ... }
}
```
---

Instead of virtualizing the base destructor, we can use `shared_ptr`:
```C++
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
```
We can also used shared pointers. With `shared_ptr`, the shared pointer itself is responsible for destruction, and since we wrapped `new Yellowdog()` with `shared_ptr<Yellowdog>`, the shared pointer knows to use `Yellowdog`'s destructor when it is time to be destroyed.

Note that we casted `shared_ptr<Yellowdog>` to `shared_ptr<Dog>`. This is allowed b/c shared pointers, like all C++ classes, support polymorphism.

_Note: Use shared pointers. Unique pointers won't work._

_Note: All classes in STL have no virtual destructors. Be careful!_


## Exceptions in Destructors
We need to prevent exceptions from leaving destructors! We don't want memory leak when exceptions (run-time) occur!
```C++
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
```  
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
```C++
~dog() {
	try {
    	// Enclose all exception-prone code here
    } catch (MYEXCEPTION e) {
    	// Catch exception
    } catch (...) {
    	// Beware: default catch is dangerous
    }
}
```
### Solution 2: Move exception-prone code to different function

	void prepareToDestruct() { throw 20; }
    ~Dog() = default;
    
Now in `main`:
```C++
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
```

## DON'T Call Virtual Functions in Constructor/Destructor
The `virtual` keyword invokes the method of the immediate scope, not the scope it was defined at:
```C++
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
 ```   
vs
```C++
class Dog {
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
```
In `main`:
```C++
Yellowdog d;
d.seeCat();
``` 
`seeCat` in the former scenario will print "bark!", while in the latter scenario, it will print "woof!".

`virtual` enables polymorphism aka dynamic binding.

### Virtual Function inside Constructor or Destructor
Virtual functions don't follow the immediate scope when called within constructor or destructor. Why? For instance, in `Dog`:

	Dog() { bark(); }
    
and in `main`:

	Yellowdog d;

When `d` is constructed, it first calls its parent's constructor. When in the parent's constructor, it sees `bark()`, but `bark()`, even though it's virtual, hasn't been defined yet for `YellowDog`, so C++ defaults to `Dog`'s `bark()` method.

**Solution**: Don't call virtual functions inside constructors/destructors.
