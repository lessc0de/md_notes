# C++ Standard Template Library

## Iterators
There are 5 categories of iterators:

### 1. Random Access Iterator: `vector`, `deque`, `array`
```C++
vector<int> itr;
itr = itr + 5;
itr = itr - 4;
++itr;
--itr;
```
### 2. Bidirectional Iterator: `list`, `set`/`multiset`, `map`/`multimap`
```C++
list<int> itr;
++itr;
--itr;
```
### 3. Forward Iterator: `forward_list`
```C++
forward_list<int> itr;
++itr;  // --itr not allowed
```
### 4. "Input" Iterator
Think of "Input" as in read-only. You can read and process values while iterating forward.
```C++
int x = *itr;  // read into x
```
### 5. "Output" Iterator
Think of "Output" as in writer. You can output values while iterating forward.
```C++
*itr = 100;  // write 100 into *itr
```
Both Input/Output iterators can move ONLY forward. Hence, they can be thought of as a "forward iterator".

---

**Unordered containers provide "at least" a forward iterator. They could also provide a bi-directional iterator. It depends on the specific STL implementation.**

---

### `const_iterator`
Every container implements `const_iterator`, which provides read-only access to container elements:
```C++
set<int>::iterator it;
set<int>::const_iterator citr;

for_each(myset.cbegin(); myset.cend(); MyFunction);  // Only in C++11
```
### Iterator Functions
```C++
advance(itr, 4);  // move itr forward 4 spots (can be used for non-random access iterators)
distance(itr1, itr2);  // measure distance between itr1 and itr2
```
### Iterator Adaptor
Iterator adaptors are special iterators.

1. Insert iterator `<class Container>`
2. Stream iterator `<typename T>`
3. Reverse iterator `<class Iterator>`
4. Move iterator (C++11) `<class Iterator>`

### 1. Insert Iterator
These are `insert_iterator`, `back_insert_iterator`, and `front_insert_iterator`
```C++
vector<int> vec1 = {4,5};
vector<int> vec2 = {12,14,16,18};
auto it = find(vec2.begin(), vec2.end(); 16);
insert_iterator<vector<int>> i_itr(vec2,it);  // point to vec2 @ position it
copy(vec1.begin(), vec1.end(),  // source
	 i_itr);					// dest
```	
### 2. Stream Iterator
These are `istream_iterator` and `ostream_iterator`.
```C++
vector<string> vec4;
copy(istream_iterator<string>(cin), istream_iterator<string>(),
	 back_inserter(vec4));

copy(vec4.begin(), vec4.end(), ostream_iterator<string>(cout, " ");

// make it terse
copy(istream_iterator<string>(cin), istream_iterator<string>(),
	 ostream_iterator<string>(cout, " ");
```
### 3. Reverse Iterator
```C++
reverse_iterator<vector<int>::iterator> ritr;
for (ritr=vec.rbegin(); ritr != vec.rend(); ++ritr)
	cout << *ritr << endl;
```
### 4. Move Iterator (C++11)
When you dereference a move iterator, the value is forwarded as an rvalue instead of an lvalue. The effect is that you can utilize the move constructor (shallow copy) instead of the copy constructor (deep copy).
```C++
int main() {
	vector<string> v = {"this", "is", "an", "example"};
    typedef vector<string>::iterator iter_t;
    
    // accumulate/concat with string(), but use move instead of copy constructor
    string concat = std::accumulate(
    					move_iterator<iter_t>(v.begin()),
                        move_iterator<iter_t>(v.end()),
                        string());
    
    cout << "concat: " << concat << endl;
}
```

## Algorithms for Iterators
Algoritms in the STL are mostly loops.

### Note 1
Algorithms always process ranges in half-open way: `[begin, end)`
```C++
sort(vec.begin(), vec.end());
reverse(vec.begin(), vec.begin()+4);  // will sort vec[0] .. vec[3], but not vec[4]
    ```
### Note 2
Sometimes, the algorithm needs 2 ranges of data. As convention, the order the parameters is Source => Destination, but we only put the beginning of the range for the destination:
```C++
vector<int> vec2(3);
copy(itr, vec.end(),  // Source
	 vec2.begin());   // Destination
```
`vec2` must have space for at least 3 elements; otherwise, the result is undefined behavior. Here, safety is sacrificed for efficiency and flexibility.

To overcome this safety issue, use insert iterators:
```C++
vector<int> vec3;
copy(vec.begin()+7, vec.end(), back_inserter(vec3));

// above isn't efficient
vec3.insert(vec3.end(), vec.begin()+7, vec.end());
```
### Note 3
Algorithms may work with functions
```C++
bool isOdd(int i) {
	return i%2;
}

int main() {
	vector<int> vec = {2, 4, 5, 9, 2};
    vector<int>::iterator itr = find_if(vec.begin(), vec.end(), isOdd);
    	// itr -> 5
    if (itr != vec.end())
    	cout << *itr << endl;
}
```
### Note 4
Algorithms can work with native C arrays
```C++
int arr[4] = {6, 3, 7, 4};
sort(arr, arr+4);  // pointers are iterators
```
Anything that behaves like an iterator is an iterator.


## Functors
Functors are implemented as classes that implement `operator()`.
```C++
class X {
public:
	void operator()(string str) {
		cout << "Calling functor X with parameter " << str << endl;
	}
};

int main() {
	X foo;
    foo("Hi");
}
```
Functors are like closures. They can store state and access variables outside the local method/function scope (using clases). Since Functors use classes, they can thus have their own type.

Functors are used in `for_each`:
```C++
class AddValue {
int val;
public:
	AddValue(int j) : val(j) { }
	void operator() (int i) {
    	cout << i+val << endl;
    }
};

int main() {
	vector<int> vec = {2, 3, 4, 5};
    int x = 2;
    for_each(vec.begin(), vec.end(), addVal(x));  // {4, 5, 6, 7}
}
```
### Built-in Functors
STL provides built-in functors:

* `less`, `greater`, `greater_equal`, `less_equal`, `not_equal_to`
* `logical_and`, `logical_not`, `logical_or`
* `multiplies`, `minus`, `plus`, `divide`, `modulus`, `negate`

These functors are useful mostly for chaining together functor-operators:
```C++
transform(myset.begin(), myset.end(),  // src
		  back_inserter(d),  // dest
          bind(logical_or<bool>,
          	   bind(greater<int>(), placeholders::_1, 20),
               bind(less<int>(), placeholders::_1, 5))
         );
```
You can see here that `bind` and `placeholders` are used to "bind" "placeholders" to where `transform` is supposed to put in the element as it traverses the iteratorable.

Functors look disgusting though. Is there a better solution? YES! Use lambdas.

### Lambdas
```C++
transform(myset.begin(), myset.end(),
		  back_inserter(d),
          [](int x) {return (x>20) || (x < 5);}
         );
```
Lambdas are of the form `[] () {}`. They look considerably shorter than functions or functors.


### Why do we need functors?
Some functions require as their parameter to be of a specific function type. Functors fulfill this role:
```C++
set<int> myset = {3, 1, 25, 7, 12};
set<int, less<int>> myset = {3, 1, 25, 7, 12};  // same as above

bool lsb_less(int x, int y) {
	return (x%10)<(y%10);
}

int main() {
	set<int, lsb_less> myset = {3, 1, 25, 7, 12};
}
```
The above code won't compile because `set` requires the 2nd template parameter to be of type "function" (aka Functor). You can fix this with:
```C++
class Lsb_less {
public:
	bool operator() (int x, int y) {
    	return (x%10) < (y%10);
    }
};
```
