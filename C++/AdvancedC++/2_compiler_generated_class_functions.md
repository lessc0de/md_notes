# Compiler Generated Class Functions

## Compiler Generated Class Functions
If you don't declare the following explicitly, they will be implicitly defined/generated.

1. Copy Constructor
2. Copy Assignment Constructor
3. Default Constructor (iff no constructor is defined)
4. Destructor

The following class

	class Dog {};

is equivalent to
```C++
class Dog {
	public:
    	dog(const dog& rhs) {...};  // 1. Copy Constructor (deep-copy)
        
        dog& operator=(const dog& rhs) {...};  // 2. Copy Assignment Constructor (deep-copy, but used with '=')
        
        dog() {...};  // 3. Default Constructor (call base class's constructor)
        
        ~dog() {...};  // 4. Destructor (call base class's destructor)
};
```
If the constructor/destructors cannot be generated, they will not be generated then.

---

Let's look at a less trivial class.
```C++
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
```
The following constructor/destructors are generated:

1. Copy Constructor - no
2. Copy Assignment Constructor - yes
3. Default Constructor - no b/c default already defined
4. Destructor - no

---

Let's look at another case:
```C++
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
```
The compiler will generate a default constructor for `Dog`. In this default constructor, it will call the default constructor of its members - `Collar m_collar` in this case. `Collar` doesn't have a default constructor, and one will not be generated b/c `Collar` already has a constructor. Oops, there is a compile error.

### Note on reference members
If a member of a class is a reference (e.g. `string& m_name`), the compiler cannot generate any constructor. Why? C++ requires any reference variable to be initialized. Compiler generated constructors construct its members, but they don't initialize them.

### "Explicitly" generating constructors/destructors
If you want to explicitly generate a constructor/destructor, use `default`.
```C++
class Dog {
   public:
	dog() = default;
    dog(string name) { ... };
};
```

## Manually Disable Compiler Generated Class Functions
The compiler can generate

1. Copy Constructor
2. Copy Assignment Operator
3. Destructor
4. Default Constructor

You can disallow generated constructors.
```C++
class OpenFile{
public:
    OpenFile(string filename) { cout << "Opening file " << filename << endl; }
};

int main() {
    OpenFile f(string("Bo_file"));
    OpenFile f2(f);
}
```
Uh-oh. Our compiler generated a copy constructor. But we don't want to have 2 handles on the same file! Let's disallow copy-constructor with `delete`.
```C++
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
```
You can also disallow constructors/destructors by declaring them in `private`.
```C++
class OpenFile {
private:
	OpenFile(const &rhs);
}
```
---

Disallowing through the `private` technique is especially useful for reference-counting shared pointers:
```C++
class RefCountPointer {
public:
	...
private:
	~RefCountPointer() {...}
};

int main {
	RefCountPointer r;
}
```
The above will not compile b/c `r` cannot destruct after leaving scope. This is great for reference-counted pointers, where shared pointers should not be able to destruct each other.

The only way to destruct is this:
```C++
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
```
Note that we allocate `RefCountPointer` on the heap, since if it were created on the stack, then `r` will be automatically destructed when the stack unwinds (leave scope). This is bad b/c now, you're attempting to destruct twice. Also, in the 2nd time, `~RefCountPointer` is private, so it wouldn't compile.
