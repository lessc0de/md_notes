### Override (for virtual function)
In order to avoid inadvertently create a new function in derived classes.
```C++
// C++ 03
class dog {
    virtual void A(int);
    virtual void B() const;
};

class yellowdog : public dog {
    virtual void A(float);  // created a new function
    virtual void B();       // created a new function
};
```
This is bad b/c we expected polymorphism, but there is none. Let's fix this:
```C++
// C++ 11
class yellowdog : public dog {
    virtual void A(float) override;     // Err: no virtual function to override
    virtual void B() override;          // Err: no virtual function to override
    void C() override;                  // Err: C isn't virtual
};
```
### keyword `final`
`final` means no class can be derived from this anymore.
```C++
class dog final {
    ...
};
``` 
or...
```C++
class dog {
    virtual void bark() final;  // no class can override bark()
};
```
### Compiler Generated Default Constructor
We've already seen this...
```C++
class dog {
    dog(int age);
    dog() = default;
};
```
### `delete`
We've already seen this...
```C++
class dog {
    dog(int age);
    dog() = default;
    dog& operator=(const& rhs) = delete;
};
```
### `constexpr`
The keyword `constexpr`
```C++
int arr[6];         // OK
int A() { return 3; }
int arr[A() + 3];   // compile error b/c compiler doesn't know A() is always constant

// C++ 11
constexpr int A() { return 3; } // force computation to happen at compile time

int arr[A()+3];     // create array of size 6

// write faster program with constexpr
constexpr int cubed(int x) { return x * x * x; }

int y = cubed(1789);    // computed only at compile time
```
### New String Literals
C++ 03:
* `char* a = "string";`
 
C++ 11:
* `char* a = u8"string";    // UTF-8`
* `char16_t* b = u"string"; // UTF-16`
* `char32_t* c = U"string"; // UTF-32`
* `char* d = R"string";     // raw string`

### lambda function
_This has changed in C++ 14._  
`[]` signifies a lambda function. `[&]` signifies a closure.
```C++
cout << [](int x, int y) { return x + y; }(3, 4) << endl;
auto f = [](int x, int y) { return x+y; };
cout << f(3, 4) << endl;

template<typename func>
void filter(func f, vector<int>arr) {
    for (auto i: arr) {
        if (f(i))
            cout << i << " ";
    }
}

int main() {
    vector<int> v = {1,2,3,4,5,6};
    
    filter([](int x) {return x>3;}, v);
    filter([](int x) {return x>2 && x<5;}, v);
    
    int y = 4;
    filter([&](int x) {return x>y;}, v);  // access y, which is out of function's scope
}
```    
