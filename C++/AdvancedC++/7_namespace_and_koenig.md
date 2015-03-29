# Namespace and Koenig

## Anonymous Namespace
```C++
namespace {
	void h() { std::cout << "h()\n"; }
}
```
is equivalent to

	static void h() { std::cout << "h()\n"; }

Both are great if you want to avoid name pollution, in case you use the same name (e.g. `h`) in many source files that are compiled together. This is similar to the following coded in Javascript:

`file1.js`
```JavaScript
(function () {
	var h = function() { console.log("hello"); };
	h();
 })();
```
`file2.js`
```	Javascript
(function () {
	var h = function() { console.log("good bye"); };
	h();
 })();
```

## Koenig Lookup - Argument Dependent Lookup
Example 1:
```C++
namespace A {
	struct X {};
    void g( X ) { cout << "calling A::g() \n"; }
}

int main() {
	A::X x1;
    g(x1);  // will this not compile?
}
```
This code will compile! And it'll work! How? It turns out that when the compiler sees `g(x1)`, it will not only search for `g` in the current scope, but also in the scope of where its parameter `x1` was defined in (`A`). This is called Koenig Lookup, or Argument Dependent Lookup (ADL).

With Koenig Lookup, we have increased the compiler search scope.

Example 2:
```C++
class C {
public:
	struct Y {};
    static void h(Y) { cout << "calling C::h() \n"; }
};

int main() {
	C::Y y;
    h(y);  // will this compile?
}
```
Due to Koenig Lookup, will the compiler lookup `h` from inside class `C`? No! Koenig lookup works only with namespaces, NOT classes!

Example 3:
```C++
namespace A {
	struct X {};
    void g( X ) { cout << "calling A::g() \n"; }
}

namespace C {
	void g(A::X) { cout << "calling C::g() \n"; }
    void j() {
    	A::X x;
        g(x);
    }
}

int main() {
	C::j();  // what will happen?
}
```   
This code will NOT compile. `C::j()` will call `g(x)`, but which `g` should we use? Due to Koenig lookup, `C::g` and `A::g` both match! What happens if we change `C` to a class?
```C++
namespace A {
	struct X {};
    void g( X ) { cout << "calling A::g() \n"; }
}

class C {
	void g(A::X) { cout << "calling C::g() \n"; }
    void j() {
    	A::X x;
        g(x);
    }
};

int main() {
	C::j();  // what will happen?
}
```  
This compiles. The compiler will lookup the scope of the class first, then its parent class, and then lookup global/A namespace if not found in class scope.

### Name hiding: namespace example
```C++
namespace A {
	void g(int x) {}
    namespace C {
    	void g() {}
        void j() {
        	g(8);
        }
    }
}

int main() {
	A::C::j();
}
```
This code will not compile b/c `A::C::j()` calls `A::C::g()`. It doesn't use `A::g(int x)`. How to get around that? Use `using`:
```C++
namespace A {
	void g(int x) {}
    namespace C {
    	void g() {}
        void j() {
        	using A::g // <---
        	g(8);
        }
    }
}

int main() {
	A::C::j();
}
```
### Why Koenig Lookup?

#### Practical Reason

```C++
// with koenig lookup
std::cout << "Hi.\n";  // calls std::operator<<, not the default bitshift << operator

// without koenig lookup
std::operator<<(std::cout, "Hi. \n");  // ugly!
```
#### Theoretical Reason - Engineering Principle

The Engineering Principle says the following:

1. Functions that operate on class `C` and in a same namespace with `C` are part of `C`'s interface.
2. Functions that are part of `C`'s interface should be in the same namespace as `C`.

```C++
A::C c;
c.f();
h(c);
```
In the above snippet, `h` is in the same namespace of `C`. Even though `h` isn't a member of `C`, it is still defined as "part" of `C`, since fundamentally any function that works with `C` should be "part" of `C`.

This is great for one reason. If the function `h` is "overloaded" because `h` is defined not only in `A` (`A::h`), but also in, say `std` (`std::h`), when you're in `std`, if you call `h` on `C`, since `h` is in the same namespace, Koenig lookup will gaurantee `A::h` is called, and not `std::h`. Without Koenig Lookup, `std::h` will be called, throwing a compiler error.

What is a common overloaded function in this situation? `operator+`!
