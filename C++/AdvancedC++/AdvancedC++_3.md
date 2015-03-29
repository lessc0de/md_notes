# Advanced C++
## Struct vs Class
What are their differences?

	struct Person_t {
    	string name;
        unsigned age;
    };
    
    class Person {
    	string name_;
        unsigned age_;
    };

1. `struct` members are public; `class` members are private
2. Semantically, `struct` is usually used for _passive_ objects that carry public data (aka a data container). Classes are for active objects that carry private data. Clients use classes for storing complex data structures.

## Resource Managing Classes
Here is a pitfall of a class that owns another object through a pointer.

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
    
Note that `Person` owns `pName_`.

The above code at `printName()`... why? Let's look at `push_back()`:

1. "George" is constructed
2. **A copy of "George" is saved in the vector persons.**
3. **"George" is destroyed.**

You can see that the "George" inside the vector is NOT the "George" created in `main` scope. The "George" inside the vector is a **shallow copy** of the "George" in `main` scope. Since shallow copy doesn't create a copy of the pointees of pointers, the (shallow) copy of `pName_` still points to the same exact memory address. So when the original `pName_` is deleted, the copied `pName_` is pointing to an invalid memory address! The program will crash. To solve this, we need to manually perform deep-copy:

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

Another solution is to just delete the copy constructor and copy assignment operator:

	private:
    	Person(const Person &rhs);
        Person &operator=(const Person &rhs);

---

**Note**: STL containers require that stored objects be copy-constructable and copy-assignable. If you don't want to implement these constructors, you can just `push_back` pointers to objects instead. Just make sure to delete the objects yourself later on.

Implicit copying is where bugs are most likely introduced. So be careful! To be safe, just disable compiler generated copy constructor or copy assignment operator.


## Clone() Function
You can implement a `clone()` function to copy manually and explicitly (instead of implicitly).

	class Dog;
    class Yellowdog : public Dog;
    
    void foo(Dog *d) {  // d is a Yellowdog
    	Dog *c = new Dog(*d);  // c is a Dog
    }
    
    int main() {
    	Yellowdog d;
        foo(&d);
    }

Poly-morphism "broke" our code! How to solve? Let's use `virtual` to preserve poly-morphism:

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

`virtual` makes sure that the original type's function is called, no matter what type the object was later casted into. This technique of preserving poly-morphism is called "co-variannce".


## Define Implicit Type Conversion
In C++, there are 2 types of type conversion:

1. Standard Type Conversion
  * Implicit: A, Explicit: B
2. User Defined Type Conversion
  * Implicit: C, Explicit: D

_Standard or User Defined **Explicit** Type Conversion_ is more commonly known as _casting_.

### Category A: Implicit Standard Type Conversion

    char c = 'A';
    int i = c;  // integral promotion
    
    char *pc = 0;  // int -> null pointer
    
    void f(int i);
    f(c);  // c is implicitly promoted to integer
    
    dog *pd = new yellowdog();  // pointer type conversion (yellowdog* -> dog*)

### Category C: Implicit User-defined Type Conversion
There are 2 ways to do so:


1. Use constructor that can accept a single parameter
  * convert other types of object into your class
2. Use the type conversion function
  * convert an object of your class into other types

For instance,

    class dog {
    public:
    	// "convert" a string to a dog (if no explicit keyword is used)
    	dog(string name) {m_name = name;}  // NO explicit
        string getName() { return m_name; }
    private:
    	string m_name;
    };
    
    int main() {
    	string dogname = "Bob";
        dog dog1 = dogname;
        printf("My name is %s. \n", dog1);  // but how do we convert dog -> string?
    }

Here is method 2:

	public:
    	operator string () const { return m_name; }

The above method "converts" `dog` to a `string`. The below also works:

	string dog2 = dog1;


---

Principles for user-defined implicit type conversion:

1. Avoid defining seemingly unexpected conversion.
2. Avoid defining two-way implicit conversion.
  * If you define a conversion from A -> B, you probably don't want to define a conversion from B -> A.


## All Castings Considered
Casting is an explicit type conversion. There are 5 types of casts:

1. `static_cast`
2. `dynamic_cast`
3. `const_cast`
4. `reinterpret_cast`
5. C-style casting


### 1. `static_cast`

	int i = 9;
    float f = static_cast<float>(i);  // convert object from one type to another
    dog d1 = static_cast<dog>(string("Bob"));  // type conversion needs to be defined
    dog* pd = static_cast<dog*>(new yellowdog());  // convert ptr/ref to a related tyupe (down/up cast)

### 2. `dynamic_cast`

	dog* pd = new yellowdog();
    yellowdog* py = dynamic_cast<yellowdog*>(pd);

1. Convert ptr/ref to related type (down cast)
2. Run-time check. If succeed, `py==pd`; if fail, `py==0`
3. Requires the 2 types to be polymorphic (have virtual function)

The above code succeeds. The below code fails the run-time check:

	class dog {
    public:
    	virtual dog() = default;
    };

	class yellowdog : dog;

	dog* pd = new dog();
    yellowdog* py = dynamic_cast<yellowdog*>(pd);
    if (py)  // py == 0 b/c it failed dynamic_cast
    	py.bark();

### 3. `const_cast`

	const char* str = "Hello, world.";
    char* modifiable = const_cast<char*>(str);  // only work on ptr/ref of the same type

### 4. `reinterpret_cast`
`reinterpret_cast` re-interprets the bits of an objected pointed to

	long p = 5111902;
    dog *pd = reinterpret_cast<dog*>(p);  // p is interpreted as a ptr

### 5. C-style casting
C-style casting is a mixture of `static_cast`, `const_cast`, and `reinterpret_cast`. Use with caution!

	short a = 2000;
    int i = (int)a;  // c-like cast notation
    int j = int(a);  // functional notation
    
---

One more thing about `const_cast`: it can be used in a "const" function! In a "const" function, `*this` is const!

	class dog {
    private:
    	string m_str;
    public:
    	void bark() const {
        	cout << "bark!" << endl;
            const_cast<dog*>(this)->m_str = "hi";
        };
    }


## rvalue and lvalue
Why should I care about this topic?

1. Understand C++ construct, and decypher compiler errors or warnings.
2. C++ 11 introduced "rvalue reference"

### Simplified Definition

* lvalue - an object that occupies some identifiable location in memory
  * the address can be pointed to by a memory handle (there is a variable that can point to this address)
* rvalue - anything that is NOT an lvalue
  * the object does not have a address handle (no variable can be made to point to this address)


### Lvalue examples:

	int i;			// i is an lvalue
    int *p = &i;	// b/c i's address is identifiable
    i = 2;			// memory content is modifiable
    
    dog d1;			// this is an lvalue of a user-defined type (class/struct)
    
### Rvalue examples:

	int x = 2;			// 2 is an rvalue
    int x = i+2;		// (i+2) is an rvalue
    int *p = &(i+2);	// Err: you can't get the mem addr of (i+2)
    i+2 = 4;			// Err: you can't modify the memory content of (i+2)
    2 = i;				// Err: you can't modify the memory content of 2
	
	dog d1;
    d1 = dog();			// dog() is rvalue of user defined type (class/struct)
    
    int sum(int x, int y) { return x+y; }
    int i = sum(3, 4);  // sum(3, 4) is rvalue

### Lvalue and Rvalue references

	// Reference (aka lvalue reference)
    int i;
    int& r = i;			// OK: you're referencing an lvalue
    
    int& r = 5;			// Err: you're referencing an rvalue
    
    const int& r = 5;	// Gotcha: const lvalue can reference an rvalue
	
    
    int square(int& x) { return x*x; }
    square(i);		// OK
    square(40);		// Err
    
    
    int square(const int& x) { return x*x; }
    square(i);		// OK
    square(40);		// OK
    
### Converting between lvalue and rvalue

	/*
     * lvalue can be used to create an rvalue
     */
    int i = 1;		// 1 is an rvalue
    int x = i + 2;	// i is used to create i + 2
    
    int x = i;		// i is implicitly transformed into an rvalue
    
    /*
     * rvalue can be used to create an lvalue
     */
    int v[3];
    *(v+2) = 4;	// (v+2) is an rvalue. Dereferencing an rvalue converts it to an lvalue: *(v+2) is an lvalue
    // aka, if a pointer is an rvalue, the pointee (the dereferenced pointer) is an lvalue

### Misconceptions

	/*
     * Misconception 1: function or operator always yields rvalues
     */
    array[3] = 50;  // operator [] is a function, yet it yields an lvalue
    
    /*
     * Misconception 2: lvalues are always modifiable
     */
    const int c = 1;  // c is an lvalue, but not modifiable
    
    /*
     * Misconception 3: rvalues are never modifiable
     */
    // built-in types like functions or ints are never modifiable if rvalue'd
    i + 3 = 6;		// Yep, won't work
    sum(3,4) = 7;	// Yep, won't work
    
    // this is NOT true for user defined types (class/struct)
    class dog;
    dog().bark();	// this can work! bark() can modify dog()

### Summary

1. Every C++ expression yields either an lvalue or rvalue.
2. If the expression has an identifiable memory address, it's an lvalue; otherwise, it's an rvalue.

