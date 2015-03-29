# Modern C++

## Random Number Distribution

### Bad Quality C-style RNG
```C++
int main() {
	unsigned seed = std::chrono::steady_clock::now().time_since_epoch().count();
    std::default_random_engine e(seed);
    cout << e() << endl;  // range: [e.min(), e.max()]
    
    // Range: [0, 5]
    cout << e() % 6 << endl;
    // 1. modulo-biased
    // 2. can only provide uniform distribution
}
```
### C++ Distributions
We use a random engine for a source of randomness, while we use a distribution to distribute the randomness to a range. _Note when we say the engine is a source of randomness, we mean pseudo-randomness. If you know the seed, the engine becomes highly deterministic. To reduce this determinism, make sure the seed is "random," e.g. setting the seed to the current time, or using a true physical random device as the seed._
```C++
int main() {
	unsigned seed = std::chrono::steady_clock::now().time_since_epoch().count();
    std::default_random_engine e(seed);
    
	std::uniform_int_distribution<int> distr(0, 5);  // Range: [0, 5]
    cout << distr(e) << endl;
    
    std::uniform_real_distribution<double> distrR(0, 5);  // Range: [0, 5)
    cout << distrR(e) << endl;
    
    std::poisson_distribution<int> distrP(1.0);  // mean
    cout << distrP(e) << endl;
    
    // print a normal distribution to stdout
    std::normal_distribution<double> distrN(10.0, 3.0);  // mean and standard deviation
    vector<int> v(20);
    for (int i=0; i<800; i++) {
    	int num = distrN(e);  // convert double to int
        if (num >= 0 && num < 20)
        	v[num]++;  // e.g. v[10] records number of times 10 appeared
    }
    for (int i=0; i<20; i++)
    	cout << i << ": " << std::string(v[i], '*') << endl;
}
```

## Tuple
Tuple is another class in the C++ STL. Recall pairs:
```C++
int main() {
	std::pair<int, string> p = make_pair(23, "hello");
    cout << p.first << " " << p.second << endl;
}
```
A tuple can be thought of as an extended pair. Instead of storing just 2 values, it can store an arbitrary number of values.
```C++
#include <tuple>
using namespace std;

int main() {
	tuple<int, string, char> t(32, "Penny wise", 'a');
    
    // C++11
    cout << get<0>(t) <<endl;
    cout << get<1>(t) <<endl;
    cout << get<2>(t) <<endl;
    get<1> = "Pound foolish";
    
    // the following also work in C++14
    cout << get<int>(t) <<endl;
    cout << get<string>(t) <<endl;
    cout << get<char>(t) <<endl;
    get<string> = "Pound foolish";
}
```
Tuples can be created without initialization:
```C++
tuple<int, string, char> t2;  // default constructor
t2 = tuple<int, string, char>(12, "Curiosity kills the cat", 'd');
t2 = make_tuple(12, "Curiosity kills the cat", 'd');  // convenience
```
### Comparison
```C++
if (t > t2) {  // lexicographical comparison
	cout << "t is larger than t2" << endl;
    t = t2;  // member by member copying
}
```
### Storing References in Tuples
Tuples can store references. No STL containers can store references - they can only copy or move.
```C++
string st = "In for a penny";

tuple<string&> t3(st);
//tuple<string&> t3 = make_tuple(ref(st));  // this works like above

get<0>(t3) = "In for a pound";  // st now contains "In for a pound"

auto t2 = make_tuple(12, "Curiosity kills the cat", 'd');
int x;
string y;
char z;

make_tuple(ref(x), ref(y), ref(z)) = t2;
std::tie(x, y, z) = t2;  // does the same as above
std::tie(x, std::ignore, z) = t2;  // std::ignore ignores the field

auto t4 = std::tuple_cat(t2, t3);  // t4 is tuple<int, string, char, string>
```
### Type Traits
```C++
cout << std::tuple_size<decltype(t4)>::value << endl;  // 4
std::tuple_element<1, decltype(t4)>::type d;  // d is a string
```
### When to Use Tuple?
A tuple can be used to store data with different data types. Structures are made for this.. yet why do we need a tuple?
```C++
int main() {
	struct Person { string name; int age; } p;
    tuple<string, int> t;
    
    cout << p.name << " " << p.age << endl;
    cout << get<0>(t) <<  " " << get<1>(t) << endl;
}
```
The structure version is clear about what it does. Tuple appears more.. generic. It doesn't communicate concretely what members are in the tuple. In other words, structures communicate semantics better than tuples.

So why tuples? Consider the following:
```
getNameAge() {

}
```
Let's pretend we haven't defined a structure that has a name and age. So let's use tuple instead:
```C++
tuple<string, int> getNameAge() {
	// ...
    return make_tuple("Bob", 34);
}
  
int main() {
	string name;
    int age;
    tie(name, age) = getNameAge();
}
```
As you can see, the tuple is convenient as a one-shot structure to transfer a group of data. It can save the hassle of defining a dedicated structure for the same purpose.

Tuples also have built-in lexicographical comparison:
```C++
tuple<int, int, int> time1, time2;
if (time1 > time2)
	cout << "time1 is a later time\n";
```
Tuples are also great for maps to create a multi-indexed map:
```C++
map<tuple<int, char, float>, string> m;
m[make_tuple(2, 'a', 2.3)] = "Faith will move mountains";

unordered_map<tuple<int, char, float>, string> m2;
...
```
Tuples are great as a trick for rotating variable data:
```C++
int a, b, c;
tie(b, c, a) = make_tuple(a, b, c);  // yay for lvalue referencing
```
---

Remember, tuples are great for one-time use. If you see your tuple being used in many parts of code, it is better to define a structure that codifies the semantic meaning of the data. Structures are more maintainable and readable.

