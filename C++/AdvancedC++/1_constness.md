# Constness
## const data vs const pointer
`const` is a compile time constraint that an object cannot be modified
```C++
int main() {
	const int i = 9; 
    i = 6; // FAIL
    
    const int *p1 = &i;  // data is const, pointer is NOT
    p1++;  // legal
    
    int* const p2;  // pointer const, but pointee is NOT
    
    const int* const p3;  // pointer and pointee are both const
    
}
```
### The rule of thumb:

* if `const` is on the left of `*`, data is const
  * `const int *p1 = &i`
    * `p1`'s value can change
    * `p1` can point to another mem addr
    * **`p1` is a read-only pointer.**
* if `const` is on the right of `*`, pointer is const
  * `int* const p2`
    * **`p2` is a read-write pointer constrained to one mem addr**

### Why use `const`?

1. Guard (in compile time) against inadvertent writes to variables that should be read-only
2. Self documenting read-only access to variables
3. Enable compiler optimizations for `const`'d variables
4. `const` can be used in Read-Only Memory (ROM)

---

What if you really want to modify a `const`'d variable? Use `const_cast<int&>`.

	const int i = 9;
    i = 6;  // FAIL
    const_cast<int&>(i) = 6;  // "cast away" the const of i

I want to make a variable `const` in run-time: use `static_cast<const int&>`.

    int j;
    static_cast<const int&>(j) = 7;


## const parameters, return values, and functions
### const parameters
Put the `const` keyword next to the parameter.
```C++
class Dog {
	int age;
    string name;
public:
	Dog() { age = 3; name = "dummy"; }
    void setAge(int &a) { age = a; a++; }
};

int main() {
	Dog d;
    
    int i = 9;
    d.setAge(i);
    cout << i << endl;
	return 0;
}
```
Notice that `a++` in `setAge` modifies `i` in the `main` scope. What if we don't want a function to modify it's parameter?

	void setAge(const int& a) { age = a; a++; }
    
The above will NOT compile.

`const` is useful for references/pointers. For example:

	void setAge(const int a) { age = a; }

In C++, since parameters are passed by values, `a` will be a copy of the argument, so there is no point in `const`ing a copied variable.

### const return value
Put the `const` keyword next to the output type of the function.

	const string& getName() { return name; }

`getName()` returns a `string` that is `const`.
	
    Dog d;
	const string &n = d.getName();
    cout << n << endl;

A const return value means the output of a function cannot be mutated.

### const function
Put the `const` keyword after the function declaration.

	void printDogName() const { const << name << endl; }
    
A const function will NOT modify its member variables.

---

If a method is overloaded with a const and non-const version, e.g.

	void printDogName() const { cout << name << " const" << endl; }
    void printDogName() { cout << name << " non-const" << endl; }
    
then the const version is used when the object calling it is itself const:

	Dog d;
    d.printDogName();  // "dummy non-const"
    
    const Dog d2;
    d2.printDogName();  // "dummy const"

## logic constness vs bitwise constness
What does it really mean for a function to be "const"? Consider the following
```C++
class BigArray {
	vector<int> v;
    int accessCounter;
public:
	int getItem(int index) const {
    	accessCounter++;
        return v[index];
    }
}

int main() {
	BigArray b;
}
```

The above will not compile b/c `accessCounter` is mutated, even though we said `getItem` will not mutate member variables. To get around this, just put `mutable` in front of the member variable we want to mutate even in a const function:

	mutable int accessCounter;

### Gotcha!
Consider this example:
```C++
class BigArray {
	int* v2;
public:
	void setV2Item(int index, int x) const {
    	*(v2+index) = x;
    }
}

int main() {
	BigArray b;
}
```    
The above compiles, even though we're mutating the array pointed by `v2`! Why does this happen?  
`const` really refers to "bitwise const" rather than the "logical" view of const we have in our head. `setV2Item` doesn't affect `v2` itself. It affects the pointee of `v2 + index`. Thus, `const`ness is maintained.

The solution is to just get rid of the `const` from the function declaration.
