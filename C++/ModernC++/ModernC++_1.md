# Modern C++
## Learn C++ 11
### `initializer_list` Constructor
All relevant STL containers accept initializer lists. What does this mean?
```C++
// C++ 03
int arr[4] = {3, 2, 4, 5};

// C++ 11
vector<int> v = {3, 2, 4, 5};   // this wasn't allowed in C++ 03
                                // this calls the initializer_list constructor
```
Let's define a custom `initializer_list` constructor:
```C++
#include <initializer_list>

class boVector {
    vector<int> m_vec;
public:
    boVector(const initializer_list<int>& v) {
        for (initializer_lst<int>::iterator itr = v.begin(); itr != v.end(); ++itr)
            m_vec.push_back(*itr);
    }
};

int main() {
    boVector v1 = {0, 2, 3, 4};
    boVector v2 {0, 2, 3, 4}; // effectively the same
}
```
### Aggregate Initialization
C++ 03 enabled "aggregate class or struct initialization".
```C++
// C++ 03
class dog {
public:
    int age;
    string name;
};

int main() {
    dog d1 = {5, "Henry"};  // aggregate initialization
}
```
C++ 11 extends the scope of curly brace initialization:
```C++
class dog {
public:
    dog(int age, string name) { ... };
};

int main() {
    dog d1 = {5, "Henry"};
}
```
---

Note: the compiler will search the following constructors in order (this is called "uniform initialization search order"):

1. `initializer_list` constructor
2. regular constructor that takes the appropriate parameters
3. aggregate initializer

.
```C++
class dog {
public:
    int age;                                // 3rd choice
    
    dog(int a): age(a) {}                   // 2nd choice
    
    dog(const initializer_list<int>&vec) {  // 1st choice
        age = *(vec.begin());
    }
};
```
### `auto` type
_This has changed in C++ 14._
```C++
std::vector<int> vec = {2, 3, 4, 5};

// C++ 03
for (std::vector<int>::iterator it = vec.begin(); it != vec.end(); ++it)
    m_vec.push_back(*it);

// C++ 11
for (auto it=vec.begin(); it!=vec.end(); ++it)
    m_vec.push_back(*it);
    
auto a = 6;     // a is int
auto b = 9.6;   // b is double
auto c = a;     // c is int
```
### foreach
```C++
// C++ 03
for (vector<int>::iterator it = v.begin(); it != v.end(); ++it)
    cout << (*it);

// C++ 11
for (int i: v)  // works on any class that has begin() and end()
    cout << i;  // read-only access (i is implicitly const)

for (int& i: v)
    i++;        // i is now the derefence of the iterator of v
```
### `nullptr`
`nullptr` replaces `NULL`
```C++
void foo(int i) {}
void foo(char* pc) {}

int main() {
    foo(NULL);  // ambiguity
    
    foo(nullptr);  // call foo(char*)
}
```
### `enum` class
```C++
// C++ 03
enum apple { green_a, red_a };
enum orange { big_o, small_o };
apple a = green_a;
orange o = big_o;

if (a == o)
    cout << "green apple and big orange are the same\n";
else
    cout << "green apple and big orange are not the same\n";

// C++ 11
enum class apple { green, red };
enum class orange { big, small };
apple a = apple::green;
orange o == orange::big;

if (a == o)
    cout << "green apple and big orange are the same\n";
else
    cout << "green apple and big orange are not the same\n";

// Compile fails because we haven't defined `operator==(apple, orange)`
```
### `static_assert`
```C++
// run-time assert
assert( myPointer != NULL );

// compile-time assert (C++ 11)
static_assert( sizeof(int) == 4 );
```
### Delegating Constructor
```C++
class dog {
public:
    dog() { ... }
    dog(int a) { dog(); doOtherThings(a); }  // this will create 2 dogs :(
};

// C++ 03
class dog {
    init() { ... };
public:
    dog() { init(); }
    dog(int a) { init(); doOtherThings(a); }
};

// C++ 11
class dog {
public:
    dog() { ... };
    dog(int a) : dog() { doOtherThings(a); }
    // limitation: dog() has to be called first
};
```
