# C++ Standard Template Library

## Non-modifying Algorithms
Non-modifying algorithms include `count`, `min`, `max`, `compare`, linear search, and attribute.

Before we start, we're going to define the following:
```C++
vector<int> vec = {9, 60, 90, 8, 45, 87, 90, 69, 69, 55, 7};
vector<int> vec2 = {9, 60, 70, 8, 45, 87};
vector<int>::iterator itr, itr2;
pair<vector<int>::iterator, vector<int>::iterator> pair_of_itr;
```
### Counting
```C++
int n = count(vec.begin()+2, vec.end(), 69);  // 2
int m = count_if(vec.begin(), vec.end(), [](int x) {return x<10;});  // 3
```
### Min and Max
```C++
itr = max_element(vec.begin()+2, vec.end());  // 90
itr = max_element(vec.begin(), vec.end(),
				  [](int x, int y){ return (x%10)<(y%10);});  // 9 

pair_of_itr = minmax_element(vec.begin(), vec.end(),  // {60, 69}
							 [](int x, int y) {return (x%10) < (y%10);});
```
### Linear Search (use Binary Search if sorted)
```C++
itr = find(vec.begin(), vec.end(), 55);
itr = find_if(vec.begin(), vec.end(), [](int x){ return x>80; });
itr = find_if_not(vec.begin(), vec.end(), [](int x){return x>80; });

// Search n=2 consecutive items of 69
itr = search_n(vec.begin(), vec.end(), 2, 69);

// search subrange
vector<int> sub = {45, 87, 90};
itr = search(vec.begin(), vec.end(), sub.begin(), sub.end());
itr = find_end(vec.begin(), vec.end(), sub.begin(), sub.end());

// search any of
vector<int> items = {87, 69};
itr = find_first_of(vec.begin(), vec.end(), items.begin(), items.end());
itr = find_first_of(vec.begin(), vec.end(), items.begin(), items.end(),
					[](int x, int y){return x==y*4;});

// search adjacent
itr = adjacent_find(vec.begin(), vec.end());  // find 2 adjacent items that are same
itr = adjacent_find(vec.begin(), vec.end(), [](int x, int y){return x==y*4;});
```
### Comparing Ranges
```C++
if (equal(vec.begin(), vec.end(), vec2.begin()))
	cout << "vec and vec2 are same.\n";

if (is_permutation(vec.begin(), vec.end(), vec2.begin())
	cout << "vec and vec2 have same items, but in different order.\n";

pair_of_itr = mismatch(vec.begin(), vec.end(), vec2.begin());
// find first difference
// pair_of_itr.first is an iterator of vec
// pair_of_itr.second is an iterator of vec2
   
// lexicographical comparison: one-by-one comparison with "less than"
lexicographical_compare(vec.begin(), vec.end(), vec2.begin(), vec2.end());
// {1, 2, 3, 5} < {1, 2, 4, 5}
// {1, 2}       < {1, 2, 3}
```
### Check Attributes
```C++
is_sorted(vec.begin(), vec.end());

itr = is_sorted_until(vec.begin(), vec.end());
// itr points to first place to where elements are no longer sorted

// partition x>80 elements to the right, otherwise to the left
is_partitioned(vec.begin(), vec.end(), [](int x){return x>80;});

is_heap(vec.begin(), vec.end());
itr = is_heap_until(vec.begin(), vec.end());
```
### Misc
```C++
all_of(vec.begin(), vec.end(), [](int x) {return x>80;});
any_of(vec.begin(), vec.end(), [](int x) {return x>80;});
none_of(vec.begin(), vec.end(), [](int x) {return x>80;});
```
## Modifying Algorithms
Modifying algorithms include `copy`, `move`, `transform`, `swap`, `fill`, `replace`, and `remove`.

As before, we have pre-defined the following:
```C++
vector<int> vec = {9, 60, 70, 8, 45, 87, 90};
vector<int> vec2 = {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0};
vector<int>::iterator itr, itr2;
pair<vector<int>::iterator, vector<int>::iterator> pair_of_itr;
```
### Copy
```C++
copy(vec.begin(), vec.end(), vec2.begin());
copy_if(vec.begin(), vec.end(), [](int x) {return x>80;});
// vec2: {87, 90, 0, 0, 0, 0, 0, 0, 0, 0, 0}

copy_n(vec.begin(), 4, vec2.begin());
// vec2: {9, 60, 70, 8, 0, 0, 0, 0, 0, 0, 0}

copy_backward(vec.begin(), vec.end(), vec2.end());
// vec2: {0, 0, 0, 0, 9, 60, 70, 8, 45, 87, 90}
``` 
### Move
```C++
vector<string> vec = {"apple", "orange", "pear", "grape"};
vector<string> vec2 = {"", "", "", "", "", ""};

move(vec.begin(), vec.end(), vec2.begin());
// vec: {"", "", "", ""}  // undefined
// vec2: {"apple", "orange", "pear", "grape", "", ""};
//
// if move semantics are defined for the element type, elements are moved over,
// otherwise they are copied over with copy constructor, just like copy()

move_backward(vec.begin(), vec.end(), vec2.end());
// vec2: {"", "", "apple", "orange", "pear", "grape"}
```
### transform
```C++
vector<int> vec = {9, 60, 70, 8, 45, 87, 90};
vector<int> vec2 = {9, 60, 70, 8, 45, 87, 90};
vector<int> vec3 = {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0};

transform(vec.begin(), vec.end(),
		  vec3.begin(),
          [](int x) {return x-1;});

transform(vec.begin(), vec.end(),
		  vec2.begin(),
          vec3.begin(),
          [](int x, int y) {return x+y;});
// add items from vec and vec2 and save in vec3
// vec3[0] = vec[0] + vec2[0]
// vec3[1] = vec[1] + vec2[1]
// ...
```
### swap
Swap is two way copying (literally swapping two ranges of data between 2 containers

	swap_ranges(vec.begin(), vec.end(), vec2.begin());
    
### fill
```C++
vector<int> vec = {0, 0, 0, 0, 0};

fill(vec.begin(), vec.end(), 9); // vec: {9, 9, 9, 9, 9}
fill_n(vec.begin(), 3, 9);  // vec: {9, 9, 9, 0, 0}

generate(vec.begin(), vec.end(), rand);  // use rand generator to fill
generate_n(vec.begin(), 3, rand);
```
### replace
```C++
replace(vec.begin(), vec.end(), 6, 9); // replace 6s w/ 9s
replace_if(vec.begin(),m vec.end(),
		   [](int x) {return x>80;},
           9);
replace_copy(vec.begin(), vec.end(),
		     vec2.begin(),
             6, 9);
// copy into vec2, but replace 6s w/ 9s
```
### remove
```C++
remove(vec.begin(), vec.end(), 3);  // remove all 3s
remove_if(vec.begin(), vec.end(), [](int x){return x>80;});
remove_copy(vec.begin(), vec.end(), vec2.begin(), 6);

unique(vec.begin(), vec.end());
unique(vec.begin(), vec.end(), less<int>());  // remove elems whose prev is less than itself
unique_copy(vec.begin(), vec.end(), vec2.begin());
```
### Order-changing Algorithms
Order-changing algorithms include `reverse`, `rotate`, `permute`, and `shuffle`. They change the order of elements in the container, but not necessarily the elements themselves.

### reverse
```C++
vector<int> vec = {9, 60, 70, 8, 45, 87, 90};
vector<int> vec2 = {0, 0, 0, 0, 0, 0, 0};

reverse(vec.begin()+1, vec.end()-1);
// vec: {9, 87, 45, 8, 70, 60, 90}

reverse_copy(vec.begin()+1, vec.end()-1, vec2.begin());
// vec2: {87, 45, 8, 70, 60, 0, 0}
```
### rotate
```C++
rotate(vec.begin(), vec.begin()+3, vec.end());
// vec: {8, 45, 87, 90, 9, 60, 70};

rotate_copy(vec.begin(), vec.begin()+3, vec.end(), vec2.begin());
```
### permute
```C++
next_permutation(vec.begin(), vec.end());
prev_permutation(vec.begin(), vec.end());
// {1, 2, 3, 5} < {1, 2, 4, 4}
// {1, 2}       < {1, 2, 3}
//
// sorted in ascending order <-> lexicographically smallest
// sorted in descending order <-> lexicographically greatest
```
### shuffle
```C++
random_shuffle(vec.begin(), vec.end());
random_shuffle(vec.begin(), vec.end(), rand);  // using rand generator

// C++11
shuffle(vec.begin(), vec.end(), default_random_engine());  // better than rand
```
## Sorting
Sorting algorithms requires random access iterators: `vector`, `deque`, container array, and native array.
```C++
vector<int> vec = {9, 1, 10, 2, 45, 3, 90, 4, 9, 5, 8};

sort(vec.begin(), vec.end());  // sort with operator <

bool lsb_less(int x, int y) {
	return (x%10) < (y%10);
}

sort(vec.begin(), vec.end(), lsb_less);  // sort with lsb_less();
```
### Partial Sorting (Partial Quicksort and QuickSelect)
```C++
partial_sort(vec.begin(), vec.begin()+5, vec.end(), greater<int>());
partial_sort(vec.begin(), vec.begin()+4, vec.end());  // default: less<int>()

nth_element(vec.begin(), vec.begin()+5, vec.end(), greater<int>());

partition(vec.begin(), vec.end(), lessThan10);

stable_partition(vec.begin(), vec.end(), lessThan10);  // stable version (insertion/merge)
``` 
### Heap Algorithms
```C++
vector<int> vec = {9, 1, 10, 2, 45, 3, 90, 4, 9, 5, 8};

make_heap(vec.begin(), vec.end());
// vec: {90, 45, 10, 9, 8, 3, 9, 4, 2, 5, 1}  // 0th indexed

pop_heap(vec.begin(), vec.end());
// vec: {45, 9, 10, 4, 8, 3, 9, 1, 2, 5, 90}  // 90 still remains
vec.pop_back();  // remove last item (largest one)
// vec: {45, 9, 10, 4, 8, 3, 9, 1, 2, 5}

vec.push_back(100);
push_heap(vec.begin(), vec.end());  // heapify

vector<int> vec = {9, 1, 10, 2, 45, 3, 90, 4, 9, 5, 8};
make_heap(vec.begin(), vec.end());

sort_heap(vec.begin(), vec.end());  // requires container to be heap already
```  

## Sorted Data Algorithms
There are many algorithms that work only with sorted data. These include binary search, merge, etc.

### Binary Search
```C++
bool found = binary_search(vec.begin(), vec.end(), 9);

vector<int> s = {9, 45, 66};
bool found = includes(vec.begin(), vec.end(), s.begin(), s.end());  // search 2 containers
// both containers must be sorted already

itr = lower_bound(vec.begin(), vec.end(), 9);
// find 1st pos where 9 could be inserted and still keep sorted order

itr = upper_bound(vec.begin(), vec.end(), 9);

pair_of_itr = equal_range(vec.begin(), vec.end(), 9);
// returns both 1st and last position
```
### Merge
```C++
vector<int> vec = {8, 9, 9, 10};
vector<int> vec2 = {7, 9, 10};

merge(vec.begin(), vec.end(),
	  vec2.begin(), vec2.end(),
      vec_out.begin());

inplace_merge(vec.begin(), vec.begin()+4, vec.end());
```
### Set operations
```C++
set_union(vec.begin(), vec.end(),
		  vec2.begin(), vec2.end(),
          vec_out.begin());

set_intersection(...);
set_difference(...);
set_symmetric_difference(...);
```
    
## Numeric Algorithms
Numeric algorithms include `accumulate`, inner product, partial sum, and adjacent difference.

### Accumulate
```C++
int x = accumulate(vec.begin(), vec.end(), 10);  // default is add<T>()
int x = accumulate(vec.begin(), vec.end(), 10, multiplies<int>());
```
### Inner Product
```C++
int x = inner_product(vec.begin(), vec.begin()+3,  // range #1
					  vec.end()-3,  // range #2
                      10);  // init value
// 10 + vec[0]*vec[4] + vec[2]*vec[5] + vec[3]*vec[6]

int x = inner_product(vec.begin(), vec.begin()+3,
					  vec.end()-3,
                      10,
                      multiplies<int>(),
                      plus<int>());
// 10 * (vec[0]+vec[4]) * (vec[2]+vec[5]) * (vec[3]+vec[6])
```
### Partial Sum
```C++
partial_sum(vec.begin(), vec.end(), vec2.begin());
// vec2[0] = vec[0]
// vec2[1] = vec[0] + vec[1];
// vec2[2] = vec[0] + vec[1] + vec[2];
// vec2[3] = ...

partial_sum(vec.begin(), vec.end(), vec2.begin(), multiplies<int>());
```  
### Adjacent Differences
```C++
adjacent_difference(vec.begin(), vec.end(), vec2.begin());
// vec2[0] = vec[0]
// vec2[1] = vec[1] - vec[0]
// vec2[2] = vec[2] + vec[1] + vec[0];
// vec2[3] = ...

adjacent_difference(vec.begin(), vec.end(), vec2.begin(), plus<int>());
``` 
