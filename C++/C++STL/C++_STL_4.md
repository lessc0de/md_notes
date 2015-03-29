# C++ Standard Template Library

## String Constructor and Size
String is an important class b/c many real-life applications process text.
```C++
int main() {
	// There are many string constructors
    string s1("Hello");  // C string -> C++ string
    string s2("Hello", 3);  // s2: "Hel"
    string s3(s1, 2);  // s3: "llo"
    string s4(s1, 2, 2);  // s4: "ll"
    string s5(5, 'a');  // s5: "aaaaa"
    string s6({'a', 'b', 'c'});  // s6: "abc"
    
    // Size
    s1 = "Goodbye";
    s1.size(); s1.length();  // synonymous: both return 7
    s1.capacity();  // size of storage space of array currently allocated to s1
    s1.reserve(100);  // allocate 100 bytes to s1
    
    s1.reserve(5);
    // reduce allocation to 5 bytes
    // but s1: "Goodbye", s1.size() == 7
    // s1.capacity() == 7
    // BASICALLY: reserve cannot affect size or content, so it cannot affect capacity
    
    s1.shrink_to_fit();  // shrink capacity to equal current size
    
    s1.resize(9);  // s1.size() == 9, s1: "Goodbye\0\0"
    s1.resize(12, 'x');  // s1.size() == 12, s1: "Goodbye\0\0xxx"
    s1.resize(3);  // s1.size() == 3, s1: "Goo"
}
```  
Notice that `s1.reserve(5)` doesn't do anything because `5` is smaller than the current size of `s1`. Since `reserve` isn't allowed to modify size (and consequently string content), it isn't allowed to modify capacity. Hence, it does nothing.

`s1.shrink_to_fit()` affects capacity, but also doesn't touch size and consequently the string content.

`s1.resize`, however, can affect size and consequently both string content and capacity.

---

In summary:

1. Use `reserve` to increase/decrease capacity such that it is greater than or equal to size
2. Use `resize` to increase/decrease capacity AND size together. If previous size is smaller than capacity, pad with `'\0'` or another chosen character
3. Use `shrink_to_fit` to decrease capacity to size.

**Note: If the string constructor is called with any integer parameters, the first is the starting index, and the second is the "size" away from starting index. If there is only one integer parameter, it is treated as "size", and the starting index is `0`:**
```C++
string s2("Hello", 3);  // s2: "Hel"
string s4("Hello", 2, 2);  // s4: "ll"
```


## Accessing String Characters
### Single Element Access
```
s1 = "Goodbye";
s1[2];  // lvalue reference to char @ pos. 2
s1[2] = 'x';  // "Goxdbye"
s1.at(2) = 'y';  // "Goydbye"

s1.at(20);  // can throw out_of_range exception
s1[20];  // undefined behavior
```
STL container API is similar to `vector`:
```C++
s1.front();  // 'G'
s1.back();  // 'e'
s1.push_back('z');  // s1: "Goodbyez"
s1.pop_back();  // 'z'

s1.begin();  // iterator
s1.end();  // iterator

string s3(s1.begin(), s1.begin()+3);  // s3: "Goo"

// just like vector, no push_front() or pop_front()
```
### Ranged Access
We will look at assign, append, insert, and replace.
```C++
string s2 = "Dragon Land";

s1.assign(s2);  // s1 = s2;

// assign is more flexible than operator=
s1.assign(s2, 2, 4);  // s1: "agon"
s1.assign("Wisdom");  // s1: "Wisdom"
s1.assign("Wisdom", 3);  // s1: "Wis"
s1.assign(s2, 3);  // Error
s1.assign(3, 'x');  // s1: "xxx"
s1.assign({'a', 'b', 'c'});  // s1: "abc"


s1.append(" def");  // s1: "abc def"
s1.insert(2, "mountain", 4);  // s1: "abmounc def"
s1.replace(2, 5, s2, 3, 3);  // s1: "abgon def"

s1.erase(1, 4);  // s1: "a def"

s2.substr(2, 4);  // "agon"


s1 = "abc";
s1.c_str(); s1.data();  // C++ string -> C string: "abc\0"

s1.swap(s2);  // swaps the 2 strings (acts like vector's implementation)
```
Note: Like string constructors, the first integer parameter to these methods is starting index, and the second is "size" after starting index. If there is only one integer parameter, it is treated as "size":
```C++
string s1 = "abc def";
s1.insert(2, "mountain", 4);  // s1: "abmounc def"

s1 = "Dragon Land";
string s2;
s2.assign(s1, 2, 4);  // s2: "agon"

s2.assign(4, 'q');  // s2: "qqqq"
```
In the case of `replace`, input the start index, size, then start index and size again:

	s1.replace(2, 5, s2, 3, 3);  // s1: "abgon def"

---

In summary, these methods take a position and a size. If only one int param is used, it is treated as size.


## Member Function Algorithms
Let's talk about the member function algorithms: copy, find, and compare. **Be wary, these member functions don't really follow a parameter order convention, so they are difficult to remember/memorize.**

### Copy
```C++
string s1 = "abcdefg";
char buf[20];

size_t len = s1.copy(buf, 3);  // buf: "abc", len: 3
// buf is NOT null-terminated

len = s1.copy(buf, 4, 2);  // buf: "cdef", len: 4
```
Note above that the order of integer parameters is reversed: the first int is size, the 2nd is position. `copy` is the only case where this parameter reversal occurs.

### Find
```C++
s1 = "If a job is worth doing, it's worth doing well.";
size_t found = s1.find("doing");  // found: 18
found = s1.find("doing", 20);  // start searching at pos 20; found: 36
found = s1.find("doing well", 0);  // found: 36

found = s1.find("doing well", 0, 5);
// search s1 starting at 0
// for the first 5 characters in "doing well"
// found: 18

found = s1.find_first_of("doing");
// find any of the characters in "doing"
// found: 6

found = s1.find_first_of("doing", 10);
// find starting at pos 10
// found: 12

found = s1.find_first_of("doing", 10, 1);
// find starting at pos 10
// for only 1 character of "doing": 'd'
// found: 18


found = s1.find_last_of("doing");
// find the last instance in s1
// for the last char of "doing": 'g'
// found: 40

found = s1.find_first_not_of("doing");
// find 1st char that doesn't appear in "doing"
// found: 0

found = s1.find_last_not_of("doing");
// find starting from behind
// found: 46
```
### Compare
```C++
s1.compare(s2);  // return pos if (s1 > s2); neg if (s1 < s2); else 0 if (s1 == s2)
if (s1 > s2) {}  // same as s1.compareTo(s2) > 0

s1.compare(3, 2, s2);  // (start, size, s2)

string ss = s1 + s2;  // overloaded operator+ to concatenate
``` 

## Non-member Functions
```C++
cout << s1 << endl;

// default: carriage return ('\r') signals end of string
cin >> s1;
getline(cin, s1);  // same as cin >> s1
getline(cin, s1, ';');  // delimiter is ';', not '\r'

// convert number to string
to_string(8);  // returns "8"
to_string(2.3e7);  // returns "23000000"
to_string(0xa4);  // returns "164"
to_string(034);  // returns "28"

// convert string to number
s1 = "190";
int i = stoi(s1);  // i = 190

s1 = "190 monkeys";
size_t pos;
i = stoi(s1, &pos);
// pos is the index where parsing error occurs
// i: 190, pos: 3

s1 = "a monkey";
i = stoi(s1, &pos);  // exception of invalid_argument

i = stoi(s1, &pos, 16);
// parse as hexadecimal number
// i: 10, pos: 1

// stol, stod, stof, etc. also exist
// use stringstream for more complex parsing
// use lexical_cast() for simple string conversion (from Boost Library)
```

## String Algorithms
So far we have talked about things from the `<string>` header. Since strings themselves are  containers that implement iterators, we can apply algorithms from `<algorithm>`.
```C++
string s1 = "Variety is the spice of life.";

int num = count(s1.begin(), s1.end(), 'e');  // num: 4
num = count_if(s1.begin(), s1.end9), [](char c){return (c <= 'e' && c >= 'a');}); // 6

s1 = "Goodness is better than beauty.";
string::iterator itr = search_n(s1.begin(), s1.begin()+20, 2, 's'); // itr -> first 's'
s1.erase(itr, itr+5);  // s1: "Variety i spice of life."

s1.insert(itr, 3, 'x');  // insert 'x' at pos 3
s1.replace(itr, itr+3, 3, 'y');  // replace itr to itr+3 with 3 'y's

is_permutation(s1.begin(), s1.end(), s2.begin());  // test if s1 is permutation of s2

replace(s1.begin(), s1.end(), 'e', ' ');  // replace 'e' w/ ' '
```
Note that the string member function `replace` replaces substrings, while `<algorithm>`'s `replace` replaces individual characters.
```C++
string s2;
transform(s1.begin(), s1.end(),
		  s2.begin(),
          [](char c) {
          	if (c < 'n')
              return 'a';
            else
              return 'z';
          });

s1 = "abcdefg";
rotate(s1.begin(), s1.begin()+3, s1.end());  // s1: "defgabc"
```

---

The above algorithms also work for different string classes!
```C++
string s;
u16string s1;  // vector of char16_t
u32string s2;  // vector of char32_t
wstring s0;  // vector of wchar_t (wide character)
```
