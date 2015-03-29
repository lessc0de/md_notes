# Modern C++

## User Defined Literals
User Defined Literals are a convenient way to define custom units.
```C++
constexpr long double operator"" _cm (long double x) {return x * 10;}
constexpr long double operator"" _m (long double x) {return x * 1000;}
constexpr long double operator"" _mm (long double x) {return x;}

int main() {
	long double height = 3.4_cm;
    cout << height << "mm" << endl;
    cout << (height + 13.0_m) << endl;
    cout << (130.0_mm / 13.0_m) << endl;
}
```
`operator""` signifies a user defined literal. Notice we use `constexpr`, which makes the function be performed at compile-time, not run-time. This eliminates run-time previously associated with user defined literals.

Here is an example a user defined literal for binary numbers:
```C++
int operator"" _bin (const char* str, size_t l) {
	int ret = 0;
    for (int i=0; i<i; i++) {
    	ret = ret << 1;
        if(str[i] == '1')
        	ret += 1;
    }
    return ret;
}

int main() {
	cout << "110"_bin << endl;
    cout << "1100110"_bin << endl;
    cout << "110100010001001110001"_bin << endl;
}
```
### Restrictions
User defined literals can only work with the following parameters:
```C++
char const*
unsigned long long
long double
char const*, std::size_t
wchar_t const*, std::size_t
char16_t const*, std::size_t
char32_t const*, std::size_t
```
The return value, however, can be of any type.


## Compiler Generated Functions
Recall that the compiler will generate up to 4 member functions of a class if you don't declare them by yourself. For example:
```C++
class Dog {};

/*
 * C++ 03:
 * 1. default constructor
 * 2. copy constructor
 * 3. copy assignment operator
 * 4. destructor
 */
```
C++11 generates two more special functions by the compiler:
```C++
/*
 * C++ 11:
 * 5. move constructor
 * 6. move assignment operator
 */
```
Here are the rules for compiler-generated functions:

* default constructor: gen'd iff no user-def constructor
* copy constructor: gen'd iff no user-def move constructor/`operator=`
* copy `operator=`: gen'd iff no user-def move constructor/`operator=`
* destructor: gen'd iff no user-def destructor
* move constructor: gen'd iff none of the following are user-def: copy constructor, copy `operator=`, destructor, move `operator=`
* move `operator=`: gen'd iff none of the following are user-def: copy constructor, copy `operator=`, destructor, move constructor

Notes:

* gen'd copy constructor/`operator=` mutually exclusive of user-def move constructor/`operator=`
* move constructor is gen'd iff the only user-def function, if defined, is a non-copy/move constructor/`operator=`
* move `operator=` is gen'd iff the only user-def function, if defined, is a non-copy/move constructor/`operator=`


### C++03 rules deprecated by C++11
One more thing... In C++11, the copy constructor will only be generated if, in addition to the move constructor/`operator=` not user-defined, the copy assignment operator or the destructor isn't user-defined. Similarly, the copy assignment operator will not be generated vice versa. In other words, change the following

1. copy constructor: gen'd iff no user-def move constructor/`operator=`
2. copy `operator=`: gen'd iff no user-def move constructor/`operator=`

to these - 

1. copy constructor: gen'd iff none of the following are user-def: move constructor/`operator=`, copy `operator=`, destructor
2. copy `operator=`: gen'd iff none of the following are user-def: move constructor/`operator=`, copy constructor, destructor

### Examples
An empty class `Dog` is equivalent to the following:
```
class Dog {
public:
	Dog();
    Dog(const Dog&);
    Dog& operator=(const Dog&);
    ~Dog();
    
    Dog(Dog&&);
    Dog& operator=(Dog&&);
};
```
---

Here is another example:
```C++
class Cat {
public:
	Cat(const Cat&) {}  // copy constructor
};
```
* _<strike>copy assignment operator generated</strike> (deprecated)_
* destructor generated

---
```C++
class Duck {
public:
	Duck(Duck&&) {}  // move constructor
};
```
* destructor generated

Mutexes are a great example of this Duck class. They should only be moved and never be copied.

---
```C++
class Fish {
public:
	~Fish() {}  // destructor
};
```
* default constructor generated
* _<strike>copy constructor generated</strike> (deprecated)_
* _<strike>copy assignment operator generated</strike> (deprecated)_

---
Here is a tricky one.
```C++
class Cow {
public:
	Cow& operator= (const Cow&) = delete;  // technically user-defined
    //Cow& operator= (const Cow&) = default;  // also user-defined
    
    //Cow (const Cow&) = default;
    // gen'd copy constructor deprecated, so bring
    // it back by assigning it to "default"
};
```
* default constructor generated
* _<strike>copy constructor generated</strike> (deprecated)_
* destructor generated


## Shared Pointer
Pointers are often trouble makers. Consider the following:
```C++
class Dog {
string name_;
public:
	Dog(string name) { cout << "Dog is created: " << name << endl; name_ = name; }
    Dog() { cout << "Nameless dog is created." << endl; name_ = "nameless"; }
    ~Dog() { cout << "dog is destroyed: " << name_ << endl; }
    void bark() { cout << "Dog " << name_ << " rules!" << endl; }
};

void foo() {
	Dog* p = new Dog("Gunner");
    // ... do stuff
    delete p;
    // ... do more stuff
    p->bark();  // what...? p is a dangling pointer - undefined behavior
}  // if p isn't deleted, we'll have a memory leak
```
There are three problems with the above:

1. If you don't delete the Dog before exiting function scope, you have a memory leak.
2. If you do delete the Dog but accidentally call its member function, you have a dangling pointer which results in undefined behavior. Since it is undefined behavior, it is very difficult to catch this bug later on.
3. Keeping track of when to delete the object at the right time is hard and tedious work.

How should we solve this problem in C++11? Let's use "smart" pointers. Let's look at the shared pointer from the `<memory>` module.
```C++
#include <memory>

class Dog {
	...
};

void foo() {
	shared_ptr<Dog> p(new Dog("Gunner"));  // Count = 1
    p->bark();
}  // Count = 0
```
The shared pointer keeps track of the memory-allocated object through reference counting. Above, the Gunner has a count equal to 1. By the end of `foo()`, `p` will go out of scope, so the count of Gunner goes to 0. When it reaches 0, the memory will be automatically de-allocated.

But why is it called "shared" pointer? Here's why:
```C++
void foo() {
	shared_ptr<Dog> p(new Dog("Gunner"));  // count = 1
    
    {
    	shared_ptr<Dog> p2 = pl;  // count = 2
        p2->bark();
        
        cout << p.use_count();  // prints 2
        // notice we use '.' notation for use_count()
    }  // count = 1
    p->bark();  // notice we use '->' notation for bark()
    (*p).bark();  // this works too
}  // count = 0
```
We can copy the shared pointer so that now we have 2 counts of Gunner. Hence "shared". Anyways, with this construct, there is no `delete` operator seen anywhere, even though we have two scopes in the code! Previously, we had to think about where to put the `delete`. Not anymore.

### Gotchas
Consider the following:
```C++
int main() {
	Dog *d = new Dog("Tank");
    shared_ptr<Dog> p(d);  // p.use_count() == 1
    shared_ptr<Dog> p2(d);  // p2.use_count() == 1
}
```
Uh-oh... we're supposed to have `count = 2`, but instead, we have 2 `count = 1`s. This is a bad way to use `shared_ptr`.

An object should be assigned to a smart pointer as soon as it is created. Raw pointers should not be used again. Here is a fix.
```C++
int main() {
	Dog *d = new Dog("Tank");
    shared_ptr<Dog> p(d);
    shared_ptr<Dog> p2 = p;
}
``` 
Here is another fix:
```C++
int main() { 
	shared_ptr<Dog> p = make_shared<Dog>("Tank");  // faster and safer
}
```
`make_shared` is faster because it does 2 steps in 1 step. It is safer because it handles memory exceptions automatically. The previous way doesn't. If previously in `shared_ptr<Dog> p(d)`, it throws an exception, then "Tank" is not managed by a smart pointer, so it will not be automatically deleted when scope exits.

---

Can we convert a raw pointer to a different type of pointer? Yes we can with:

* `static_pointer_cast`: pointer cast
* `dynamic_pointer_cast`: pointer cast with runtime error handling
* `const_pointer_cast`: pointer cast as a constant pointer

---

Note the following as well:
```C++
void foo() {
	shared_ptr<Dog> p1 = make_shared<Dog>("Gunner");
    shared_ptr<Dog> p2 = make_shared<Dog>("Tank");
    p1 = p2;  // Gunner is deleted, Tank's count == 2
    //p1 = nullptr;  // this works too: Gunner is deleted
    //p1.reset();  // this works too: Gunner is deleted
}
```
We have 3 ways to delete an object. The smart pointer uses the default deleter. What if we need to use a custom deleter? First of all, what is a custom deleter?
```C++
void foo() {
	shared_ptr<Dog> p1 = make_shared<Dog>("Gunner");
    shared_ptr<Dog> p2 = shared_ptr<Dog>(new Dog("Tank"),
    	[](Dog* p) {cout << "Custom deleting. "; delete p;});
    
    shared_ptr<Dog> p3(new Dog[3],
    	[](Dog* p) {delete[] p;});
}
```
---

You can fetch the raw pointer out of the shared pointer through `get`:

	Dog *d = p1.get();

Make sure not to `delete d`. Also make sure not to make a new shared pointer from this `d`. Also don't save the pointer `d` in a container; when the shared pointer deletes the dog, the container now has a dangling pointer - undefined behavior. We want `shared_ptr` to manage the memory. Use `get()` with caution!


## Weak Pointers
Shared pointers provide shared ownership of objects. When all shared pointers go out of scope, that object will be deleted. This takes care of memory leaks and dangling pointers. But not all cases of memory leaks will be solved just by shared pointers.
```C++
class Dog {
	shared_ptr<Dog> m_pFriend;
public:
	string m_name;
    void bark() { cout << "Dog " << m_name << " rules!" << endl;
    Dog(string name) { cout << "Dog is created: " << name << endl; m_name = name;
    ~Dog() { cout << "dog is destroyed! " << m_name << endl; }
    void makeFriend(shared_ptr<Dog> f) { m_pFriend = f; }
};

int main() {
	shared_ptr<Dog> pD(new Dog("Gunner"));
    shared_ptr<Dog> pD2(new Dog("Smokey"));
    pD->makeFriend(pD2);
    pD2->makeFriend(pD);
}
```
Uh-oh.. two dogs making friends with each other.. with shared pointers.. doesn't this look like some type of race condition? This is known as **cyclic reference**. None of the shared pointers will go out of scope, so none of the dogs will be deleted.

Instead of using shared pointers for friending, let's use weak pointers:
```C++
class Dog {
	weak_ptr<Dog> m_pFriend;
	...
};
```
We just changed the type of `m_pFriend` from `shared_ptr<Dog>` to `weak_ptr<Dog>`. Note that the type of the parameter to `makeFriend` is still `shared_ptr<Dog>`. No more memory leak occurs.

A weak pointer only wants to use an object. It doesn't want to "own" the object. In other words, it doesn't want to manage memory for the object. A weak pointer is like a raw pointer, but it has one layer of protection: you CANNOT perform `delete` on `weak_ptr`. A weak pointer is thusly not always valid. If the object is deleted, the weak pointer is known as an empty pointer.

The weak pointer has one more layer of protection. You must call `lock()` first to use the object.
```C++
void showFriend() {
	cout << "My friend is: " << m_pFriend.lock()->m_name << endl;
}
```
The `lock()` function creates a shared pointer out of the weak pointer. It does that to ensure the weak pointer is still pointing to a valid object and the object will not be deleted while it is being accessed.

You can check the validity of the object:

	if(! m_pFriend.expired()) cout << "My friend is: " << m_pFriend.lock()->m_name << endl;

You can use `use_count()`, just as in `shared_ptr`:

	cout << m_pFriend.lock()->m_name << "has" << m_pFriend.use_count() << " friends." << endl;


## Unique Pointers
Another type of smart pointer is the unique pointer, which represents the exclusive ownership of an object. It is a light weight smart pointer b/c it is less expensive than `shared_ptr`.
```C++
class Dog {
	...
};

void test() {
	unique_ptr<Dog> pD(new Dog("Gunner"));
    
    pD->bark();
    // do some things
    
    Dog *p = pD.release();  // get raw pointer from unique_ptr; pD no longer responsible
    // p is now responsible
    
    if (!pD)
    	cout << "pD is empty.\n";
}

int main() {
	test();
}
```
Above we used `release()` to "release" ownership. Below, we "reset" ownership to a new Dog. The previous Dog is automatically de-allocated:
```C++
void test() {
	unique_ptr<Dog> pD(new Dog("Gunner"));
    
    pD.reset(new Dog("Smokey"));  // Gunner is destroyed
    
}  // Smokey is destroyed
```
Note that calling `pD.reset()` just sets `pD` to the null pointer, so it is empty:
```C++
void test() {
	unique_ptr<Dog> pD(new Dog("Gunner"));
    
    pD.reset();  // Gunner is destroyed
    
    if (!pD)
    	cout << "pD is empty.\n";   
}
```
### Move Semantics

Two unique pointers cannot point to the same object at the same time, but they can point to the same object at different times:
```C++
void test() {
	unique_ptr<Dog> pD(new Dog("Gunner"));
    unique_ptr<Dog> pD2(new Dog("Smokey"));
    pD2 = move(pD);
    // 1. Smokey is destroyed
    // 2. pD is empty
    // 3. pD2 owns Gunner
    
    pD2->bark();
}
```
Move works because we're "moving" or "transferring" ownership from one unique pointer to the other unique pointer.

---
```C++
void f(unique_ptr<Dog> p) {
	p->bark();
}

void test() {
	unique_ptr<Dog> pD(new Dog("Gunner"));
    f(move(pD));
}

int main() {
	test();
}
```
Here's a good question: where is Gunner destroyed? Is it destroyed in `move`, or in `test`? Gunner is destroyed after the function `f` leaves scope. `move` gave ownership of Gunner from `pD` to `p` (inside function `f`).

Similarly,
```C++
unique_ptr<Dog> getDog() {
	unique_ptr<Dog> p(new Dog("Smokey"));
    return p;
}

void test() {
	unique_ptr<Dog> pD2 = getDog();
}
```
Functions return by value, so move semantics kick in. `p` moves ownership from itself to `pD2`.

### Unique Pointers for has-a composition
```C++
class Dog {
	//Bone* pB;
	unique_ptr<Bone> pB;  // prevent memory leak if constructor throws exception
public:
	...
};
```
In the traditional has-a compositional structure, we use a pointer to represent resource ownership. Now with C++11, we can use smart pointers, namely unique pointers, to represent ownership. We could have used shared pointers, but they're more expensive.


### One more thing
Remember when you use shared pointers for arrays, you have to define a custom deleter? For unique pointers, you don't have to! Just do this:

	unique_ptr<Dog[]> dogs(new Dog[3]);


## Resource Managing Class
Recall this code from before:
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
This illustrated the problem of relying on compiler generated copy constructor/`operator=` to perform copies, but these copies are shallow, not deep. The solution, of course, was to defined our own copy constructor/`operator=` to implement deep copy that follows through pointers. You could also delete the constructor/`operator=`.

C++11 has additional solutions:
```C++
vector<Person> persons;
//persons.push_back(Person("George"));
persons.emplace_back("George");
```
We use `emplace_back` instead of `push_back`. `Person`, however, still must be movable or copyable.

Even better, why don't we use smart pointers?
```C++
class Person {
public:
	Person(string name) : pName_(new string(name)) { }
    ~Person() { delete pName_; }  // disable copy constructor
    Person(Person&&) = default;  // enable move constructor
    Person& operator= (Person&&) = default;  // enable move assignment
    void printName() { cout << *pName_; }
private:
	unique_ptr<string> pName_;  // uncopyable
};

int main() {
	vector<Person> persons;
    Person p("George");
    persons.push_back(std::move(p));
    persons.front().printName();
}
```
