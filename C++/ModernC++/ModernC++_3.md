# Modern C++

## Rvalue Reference - Move Semantics
What is an rvalue reference?
```C++
int a = 5;  // a is an lvalue
int& b = a;  // b is an lvalue reference
int&& c = 5;  // c is an rvalue reference
```
How can lvalue/rvalue references be used?
```C++
void printInt(int& i) { cout << "lvalue reference: " << i << endl; }
void printInt(int&& i) { cout << "lvalue reference: " << i << endl; }
```
We have overloaded `printInt` with an lvalue version and an rvalue version:
```C++
int main() {
	int a = 5;
    printInt(a);  // lvalue version
    printInt(6);  // rvalue version
}
```
_Note: if you overload with a non-reference version, the compiler will get confused:_

	void printInt(int i) { ... }

Why do we use rvalue reference? Consider the following instance:
```C++
class boVector {
	int size;
    double *arr_;
public:
	boVector(const boVector& rhs) {  // Copy Constructor
    	size = rhs.size;
        arr_ = new double[size];
        for (int i=0; i<size; i++) { arr_[i] = rhs.arr_[i]; }
    }
    ~boVector() { delete[] arr_; }
};

void foo(boVector v);

boVector createBoVector();  // creates a boVector

int main() {
	boVector reusable = createBoVector();
    foo(reusable);  // invoke costly copy constructor (it's ok, b/c we need it to stay reusable)
    
    foo(createBoVector());  // copy constructor is costly (not ok, b/c createBoVector is a temporary that is destroyed anyways)
}
 ```   
As you can see, the second call `foo(createBoVector())` is costly, because `createBoVector()` is an rvalue - it is a temporary value that gets destroyed soon. What is the point of invoking the copy constructor on it when it's going to live for so short? Compare this to the first call `foo(reusable)`. In the first call, we are willing to pay the cost of the copy constructor because we want to keep the integrity/immutability of the "reusable" `boVector`.

Anyways, how can we use `foo(createBoVector())` but at the same time avoid this costly copying? Let's create a _Move Constructor_:
```C++
class boVector {
...
public:
	...
    boVector(boVector&& rhs) {  // Move constructor
    	size = rhs.size;
        arr_ = rhs.arr_;
        rhs.arr_ = nullptr;  // clear rhs's array!
    }
};
```
You can think of the move constructor as a _shallow copy_, whereas the copy constructor performs a _deep copy_.

---

In summary, with rvalue references, instead of doing this:

	void foo_by_value(boVector v);
    void foo_by_ref(boVector& v);

You can just do this:

	void foo(boVector v);
    void foo_by_lvalref(boVector& v);
    
And the corresponding constructor (copy/move) or no constructor (in the case of lvalue reference) will be used:
```C++
int main() {
	boVector reusable = createBoVector();
    foo_by_ref(reusable);		// calls no constructor (super efficient)
    foo(reusable);				// calls copy constructor (expensive)
    foo(std::move(reusable));	// calls move constructor (efficient)
    // reusable no longer "reusable" now
    foo(createBoVector());		// also calls move constructor (efficient)
}
```
---

_Note 1: The most useful place for rvalue reference is overloading copy constructor and copy assignment operator to achieve move semantics instead of copy semantics_
```C++
X& X::operator=(X const & rhs);		// regular copy assignment operator
X& X::operator=(X&& rhs);			// new move assignment operator
```  
_Note 2: Move semantics is already implemented for all STL containers. Passing-by-value still works for STL containers._


## Rvalue Reference - Perfect Forwarding

The second major usage of rvalue reference is known as "perfect forwarding." Here is an example:
```C++
void foo (boVector arg);
// boVector has both move and copy constructors

template< typename T >
void relay(T arg) {
	foo(arg);
}

int main() {
	boVector reusable = createBoVector();
    relay(reusable);
    ...
    relay(createBoVector());
}
```    
Here, `relay` is generic function that is specialized to `boVector` during compile-time. But `boVector` is different from `boVector&&`. We can't modify `relay` since it is bound to only 1 type during compile-time. Can we get around this? Yes!
```C++
template< typename T>
void relay(T&& arg) {
	foo(std::forward<T>(arg));
}
```
For perfect forwarding to work, there are 2 requirements:

1. No costly and unnecessary copy construction of `boVector` should be made.
2. rvalue is forwarded as rvalue, and lvalue is forwarded as lvalue.

### How is perfect forwarding achieved?

#### Reference Collapsing Rules
```
T& &	==>	T&
T& &&	==>	T&
T&& &	==>	T&
T&& &&	==>	T&&
```
The compiler can deduce types as above. So even though we have `T&& arg` as the parameter in `relay`, the compiler will interpret an lvalue `T&& & ==> T&`.

#### Remove Reference
How do we remove references?
```C++
namespace std {
	template< class T>
	struct remove_reference;  // this removes references on type T (no perfect forwarding achieved)
}

// T is int&
std::remove_reference<int&>::type i;  // int i;

// T is int
std::remove_reference<int>::type i;  // int i;
```
#### Explanation

Through its reference collapsing rules, C++ can forward perfectly an lvalue or an rvalue to a variable. This means that if you want to create a reference that is an lvalue reference when assigned to an lvalue and is an rvalue reference when assigned to an rvalue, you use `T&&` to "take care of both cases."
```
// T&& variable is initialized with rvalue => rvalue reference
relay(9); => T = int&& => T&& = int&& && = int&&
// T&& variable is initialized with lvalue => lvalue reference
relay(x); => T = int& => T&& = int& && = int&
```
But what gives? You said `T&&` is an "rvalue reference", but here we see `T&&` not used as that. Rather, it is used as a "Universal Reference"!

A universal reference comes about from these 2 conditions:

1. `T` is a template type
2. Type deduction (reference collapsing) happens to `T`

In any other case, `T&&` is an rvalue reference.

---

In summary, if you want to achieve perfect forwarding, you change:
```C++
template <typename T>
void relay(T arg) {
foo(arg);    
}
```
to
```C++
template <typename T>
void relay(T&& arg) {
	foo(std::forward<T>(arg));
}

```
