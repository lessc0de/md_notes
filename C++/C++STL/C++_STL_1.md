# C++ Standard Template Library

## Overview
We need a **Standard Library** to encourage code reuse. The standard library should expose algorithms and containers for us to use.

Suppose we have `N` algorithms, and `M` containers. If we want every `M` containers to be able to use all `N` algorithms, we'll need to implement `N * M` implementations. This is way too much, so C++ instead uses **Standard Template Library (STL)** to avoid re-implementation. The STL instead provides `N + M` implementations.

To achieve this, algorithms will work only with the Iterators interface, and the Containers must implement this same interface.

In short, STL provides implementation and interface for Algorithm, Iterator, and Container.

Let's look at some examples:
```C++
using namespace std;

vector<int> vec;  // CONTAINER
vec.push_back(4);
vec.push_back(1);
vec.push_back(8);

vector<int>::iterator itr1 = vec.begin();  // ITERATOR
vector<int>::iterator itr2 = vec.end();

for (vector<int>::iterator itr = itr1; itr != itr2; ++itr)
	cout << *itr << " ";

sort(itr1, itr2);  // ALGORITHM
```
### Containers
There are 3 types of containers

1. Sequence Container
  * vector, deque, list, forward list, and array
2. Associative Container (binary tree)
  * set, multiset
  * map, multimap
3. Unordered Containers (hash table)
  * unordered set/multiset
  * unordered map/multimap

### STL Headers
Here are the STL headers:
```C++
#include <vector>
#include <deque>
#include <list>
#include <set>				// set and multiset
#include <map>				// map and multimap
#include <unordered_set>	// unordered set/multiset
#include <unordered_map>	// unordered map/multimap
#include <iterator>
#include <algorithm>
#include <numeric>			// numeric algorithms
#include <functional>		// functors
```

## Sequence Containers
### Common Sequence Container API
```C++
// common member functions of all containers
// vec: {4, 1, 8}
if (vec.empty()) { cout << "Not possible.\n"; }

cout << vec.size();  // 3

for (auto itr = vec.begin(); itr != vec.end(); ++itr)
	acout << *itr << " ";

vector<int> vec2(vec);  // Copy constructor, vec2: {4, 1, 8}

vec.clear()  // remove all items in vec; vec.size() == 0

vec2.swap(vec);  // vec2 becomes empty, and vec has 3 items
```
This common API has no penalty of abstraction due to inheritance/polymorphism. All abstraction has been optimized away to be very efficient.

### Vector
A vector is a dynamic array that grows in one direction.  
Here are vector specific operations:
```C++
cout << vec[2];		// no range check
cout << vec.at(2);	// throw range_error exception if out of range

// "random" access
for (int i=0; i < vec.size(); i++)
	cout << vec[i] <<  " ";

// iterator/sequential access (faster than random access)
for (auto itr = vec.begin(); itr != vec.end(); ++itr)
	cout << *itr << " ";
for (auto item: vec)  // C++11
	cout << item << " ";

// vector is just a dynamically-allocated array
// you can treat it as a C array
int *p = &vec[0];
p[2] = 100;
```
---

Properties of vector:

1. fast insert/remove _at the end_: `O(1)`
2. slow insert/remove _anywhere else_: `O(N)`
3. slow search: `O(N)`


### Deque
Deque and vector have very similar interfaces. However, deque can grow not only at the end (as in vector) but also in the beginning:
```C++
deque<int> deq = {4, 6, 7};
deq.push_front(2);  // deq: {2, 4, 6, 7}
deq.push_back(3);  // deq: {2, 4, 6, 7, 3}

// Deque has similar interface with vector
cout << deq[1];  // 4
```
Deque properties:

1. fast insert/remove _at beginning **and** end_: `O(1)`
2. slow insert/remove _in the middle_: `O(N)`
3. slow search: `O(N)`


### List
A list is really a doubly-linked list. The key attribute of a list is that it has fast insert/remove ANYWHERE!
```C++
list<int> myList = {5, 2, 9};
myList.push_back(6);  // myList: {5, 2, 9, 6}
myList.push_front(4);  // myList: {4, 5, 2, 9, 6}

list<int>::iterator itr = find(myList.begin(), myList.end(); 2);  // itr -> 2
myList.insert(itr, 8);  // insert 8 before itr
					    // myList: {4, 5, 8, 2, 9, 6}
                        // itr -> 2
itr++;  // itr -> 9
myList.erase(itr);  // itr not valid
```
List properties:

1. fast insert/remove _anywhere_: `O(1)` (but you MUST have a pointer to that location, which may take `O(N)` time)
2. slow search: `O(N)`
3. no random access: no `[]` operator


### Lists vs Arrays
Caches take advantage of "spatial locality." When you load an array like so,
```C++
for (int i=0; i < N; i++)
	cout << arr[i] << endl;
```
the program will first look at memory for `arr[0]` (slow operation), but then it will store `arr[0]` and it's neighboring memory location into cache, like `arr[1]` and `arr[2]`. Thus, lookup for `arr[1]`, `arr[2]`, etc. will be very fast since they are already in cache.

Lists store items non-contiguously, so when the cache stores contiguous memory addresses, those addresses probably do not contain many of list's items. Thus, "cache-miss" will occur many times. Hence, lists are "slower" than arrays for sequential access.

lists also require more memory. List as a structure requires 2 or more members (1 or 2 pointers and an item field), while arrays require one member (an item field).

Lists are only really great for "splicing":

	// splice myList1[a,b] into myList2 @ position itr
	myList1.splice(itr, myList2, itr_a, itr_b);  // O(1)

### Forward List
A forward list only has a forward pointer. In other words, it is a singly-linked list

### Array

How can we utilize the common sequence container API for arrays?
```C++
int a[3] = {3, 4, 5};
a.begin();
a.end();
a.size();
a.swap();
```
C++ provides a solution called "Array Container":

	array<int, 3> a = {3, 4, 5};
    
The only limitations are that the size cannot be changed. Also `array<int, 3>` is a different type from `array<int, 4>`. This limits the genericity of types for parameters for functions accepting STL arrays.


## Associative Containers
Associative Containers are really binary trees. Also, these trees are defaultly assorted by `<` operator. Opposite of sequence containers, associative containers do not expose `push_back()` or `push_front()`.

### Set
```C++
set<int> myset;
myset.insert(3);  // {3}
myset.insert(1);  // {1, 3}
myset.insert(7);  // {1, 3, 7}

set<int>::iterator it;
it = myset.find(7);

pair<set<int>::iterator, bool> ret;
ret = myset.insert(3);
if (ret.second == false)
	it = ret.first;  // it points to 3

myset.insert(it, 9);  // {1, 3, 7, 9}
					  // it provides a "hint": O(log(n)) => O(1)
                      // it should be the element preceding the inserted element

myset.erase(it);  // {1, 7, 9}

myset.erase(7);  // {1, 9}
```
### Multiset

	mutliset<int> myset;
    
Sets/Multisets are READ-ONLY. This is because sets/multisets maintain order by themselves. If you modify the element, it breaks the order that the set has maintained. The set will no longer be guaranteed to be sorted.

	it = myset.begin();
    *it = 10;  // *it is read-only

Set properties:

1. fast search: `O(log(n))`
2. slow traversal (like list, slow vs vector/deque)
3. no random access: no `[]` operator

### Maps and "Associative-ness"
Why are sets called "associative"? Sets are technically maps that map keys to the same key. Here are maps:
```C++
map<char, int> mymap;
mymap.insert(pair<char,int>('a', 100));
mymap.insert(make_pair('a', 200));

map<char,int>::iterator it = mymap.begin();
mymap.insert(it, pair<char, int>('b', 300));  // "it" is a hint

it = mymap.find('z');  // O(log(n))

// showing contents
for (it=mymap.begin(); it != mymap.end(); ++it)
	cout << (*it).first << " => " << (*it).second << endl;
```
Map properties: _same as set's_.


## Unordered Container (C++11)
Internally, an unordered container is implemented as a hash table, with buckets and entries. Each key in the bucket is calculated by a hash function. The biggest advantage of this structure is that it takes constant time to search! The disadvantage is that if the hash function isn't good enough, it'll take `O(N)` time to search, i.e. most entries are only 1 or a few buckets. This is known as hash collisions.
```C++
/* unordered_set */
unordered_set<string> myset = {"red", "green", "blue"};
unordered_set<string>::const_iterator itr = myset.find("green");  // O(1)
if (itr != myset.end())  // important check!
	cout << *itr << endl;

vector<string> vec = {"purple", "pink"};
myset.insert(vec.begin(), vec.end());
```
### Hashtable specific APIs
```C++
cout << "load factor = " << myset.load_factor() << endl;
string x = "red;
cout << x << "is in bucket #" << myset.bucket(x) << endl;
cout << "Total bucket #" << myset.bucket_count() << endl;
```
### Associative Array
This is basically a map:
```C++
unordered_map<char, string> day = {{'S', "Saturday"}, {'M', "Monday"}};

cout << day['S'] << endl;  // no range check
cout << day.at('S') << endl;  // range check

vector<int> vec = {1, 2, 3};
vec[5] = 5;  // compile error

day['w'] = "Wednesday";  // inserts {'W', "Wednesday"}
day.insert(make_pair('F', "Friday"));  // inserts {'F', "Friday"}

day.insert(make_pair('M', "MONDAY"));  // fail to modify, you can't insert a key
									   // that already exists
day['M'] = "MONDAY";  // succeed to modify
```
### Note:
```C++
void foo(const unordered_map<char, string>& m) {
	cout << m['S'] << endl;
}
```
This will not compile because the `[]` operator is treated as mutative. Thus, we need to use another API:
```C++
void foo(const unordered_map<char, string>& m) {
	auto itr = m.find('S');
    if (itr != m.end())  // remember to check!
    	cout << *itr << endl;
}
```
Associative Array properties:

1. Search time: `unordered_map, O(1)`; `map, O(log(n))`
2. Unordered_map may be degrade to `O(n)`; `map` is gauranteed `O(log(n))`
3. You cannot use (unordered) multimap because they don't provide `[]` operator.

## Container Adaptors
Stack, Queue, and Priority Queue are known as _container adaptors_. Container adaptors are not full container classes, but classes that provide a specific interface relying on an object of one of the other container classes (such as deque or list) to handle elements. The underlying container is encapsulated in such a way that its elements are accessed by the members of the container adaptor independtly of the underlying container class used.

1. stack: LIFO, `push()`, `pop()`, `top()`
2. queue: FIFO, `push()`, `pop()`, `front()`, `back()`
3. priority queue: first item always has highest priority
  * `push()`, `pop()`, `top()`


## Categorizing Containers
There are a lot of containers. Here is an easy way of thinking about them:

1. Array based containers: vector, deque
2. Node based containers: list + associative containers + unordered containers

Array based containers invalidate pointers (native pointers, iterators, and references):
```C++
vector<int> vec = {1,2,3,4};
int *p = &vec[2];
vec.insert(vec.begin(), 0);
cout << *p << endl;  // no longer gauranteed to be 2
```
