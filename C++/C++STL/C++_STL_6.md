# C++ Standard Template Library

## String Stream
The concept of String Streams is confusing to the beginner. It is a stream without IO operations. What the hell? It only does read/write of string. In other words, it can read/write from a string. Why? The main reason is that we can use the File API with strings, or all the formatting afforded to files that we can now use with strings! This is a matter of code re-use.
```C++
stringstream ss;
ss << 89 << "  Hex: " << hex << 89 << "  Oct: " << oct << 89;
cout << ss.str() << endl;  // prints 89  Hex: 59  Oct: 131
```
Remember to call `str()` to get the string out of the stringstream.
```C++
int a, b, c;
string s1;
ss >> hex >> a;  // formatted input works token by token.
// tokens are separated by spaces, tabs, or newlines
// a: 137

ss >> s1;  // s1: "Hex: "
ss >> dec >> b;  // b: 59

ss.ignore(6);  // ignore the next 6 chars in the stream

ss >> oct >> c;  // c: 89
```
---

Lastly, String Streams have two subclasses: `ostringstream` and `istringstream`. Use `ostringstream` if you're only going to use formatted output. `istringstream` is for formatted input. If you need input/ouput, just use `stringstream`.


## Enable streaming for your own class
```C++
struct Dog {
	int age;
    string name;
};

ostream& operator<< (ostream& sm, const Dog& d) {
    sm << "My name is" << d.name << " and my age is " << d.age;
    return sm;
}

istream& operator>> (istream& sm, Dog& d) {
    sm >> d.age;
    sm >> d.name;
    return sm;
}

int main() {
    Dog d{2, "Bob"};  // Universal Initialization from C++11
    cout << d << endl;  // My name is Bob and my age is 2
    
    cin >> d;  // Enter: 5, Enter: Gunner
    cout << d;  // My name is Gunner and my age is 5
}
```
Notice that we defined `operator>>` and `operator<<` as non-member functions, which is the preferred way for custom class streaming. Doing so emulates the way we use streaming for `cout` and etc. through the Koenig.

