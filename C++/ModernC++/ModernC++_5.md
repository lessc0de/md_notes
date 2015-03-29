# Modern C++

## Regular Expressions
A regular expression is a specific pattern that provides concise and flexible means to "match" strings of text, such as particular characters, words, or patterns of characters.
```C++
#include <regex>
using namespace std;

int main() {
	string str;
    while (true) {
    	cin >> str;
        regex e("abc");
        
        bool match = regex_match(str, e);
        
        cout << (match? "Matched" : "Not matched") << endl << endl;
    }
}
```
The main API we'll use are the following two:
```C++
string str = "some string to match or search";
regex e("somepattern");

// Matching
bool match = regex_match(str, e);

// Searching
bool match = regex_search(str, e);
```
C++11 provides 6 types of regular expression grammars:

1. ECMAScript (default)
2. basic
3. extended
4. awk
5. grep
6. egrep

Change the grammar type as follows:

	regex e("^abc.+", regex_constants::grep);
    
### Submatching
```C++
string str = "www.blah.com";

regex e(
	"([[:w:]]+)@([[:w:]]+)\.com"  // ()s store submatches
);  // [[:w:]] is \w

smatch m;  // typedef std::match_results<string> smatch;
bool found = regex_search(str, m, e);  // save results into m

// print all matches
cout << "m.size()" << m.size() << endl;
for (int n=0; n<m.size(); ++n) {
	cout << "m[" << n << "]: str()=" << m[n].str() << endl;
    cout << "m[" << n << "]: str()=" << m.str(n) << endl;  // works too
    cout << "m[" << n << "]: str()=" << *(m.begin()+n) << endl;  // works too
}

cout << "m.prefix().str(): " << m.prefix.str() << endl;
cout << "m.suffix().str(): " << m.suffix.str() << endl;
```
If we input the following, we'll get this:

	boq@gmail.com
    m.size() 3
    m[0]: str()=boq@gmail.com
    m[1]: str()=boq
    m[2]: str()=gmail
    m.prefix().str(): 
    m.suffix().str(): 

How about this?

	<email>boq@yahoo.com</email>
    m.size() 3
    m[0]: str()=boq@yahoo.com
    m[1]: str()=boq
    m[2]: str()=yahoo
    m.prefix().str(): <email>
    m.suffix().str(): </email>

### Iterators
Now we'll look at:

1. Regex Iterator
2. Regex Token Iterator
3. `regex_replace`

#### Regex Iterator
```C++
int main() {
	string str = "boq@gmail.com;  boqian@hotmail.com;;  bo_qian2000@163.com";
    
    regex e(
    	"([[:w:]]+)@([[:w:]]+)\.com"
    );
    
    smatch m;
    bool found = regex_search(str, m, e);
    for (int n=0; n<m.size(); n++)
    	cout << "m[" << n << "]: str()=" << *(m.begin()+n) << endl;
}
```
Notice the above example will only match once and then stop. What if we need more than just one (whole) match? The solution is the following:
```C++
int main() {
	string str = "boq@gmail.com;  boqian@hotmail.com;;  bo_qian2000@163.com";
    
    regex e(
    	"([[:w:]]+)@([[:w:]]+)\.com"
    );
    
    sregex_iterator pos(str.cbegin(), str.cend(), e);
    sregex_iterator end;  // default constructor defines past-the-end iterator
    for(; pos != end; pos++) {
    	cout << "Matched:  " << pos->str(0) << endl;
        cout << "user name:  " << pos->str(1) << endl;
        cout << "domain:  " << pos->str(2) << endl;
        cout << endl;
    }
}
```
#### Regex Token Iterator
Regex Iterator points the match result. Regex Token Iterator points to submatches.
```C++
int main() {
	string str = "boq@gmail.com;  boqian@hotmail.com;;  bo_qian2000@163.com";
    
    regex e(
    	"([[:w:]]+)@([[:w:]]+)\.com"
    );
    
    sregex_token_iterator pos(str.cbegin(), str.cend(), e);
    sregex_token_iterator end;  // default constructor defines past-the-end iterator
    for(; pos != end; pos++) {
    	cout << "Matched:  " << pos->str() << endl;
        cout << endl;
    }
}
```
You can control which submatch/"token" to extract by passing another integer parameter marking which submatch you're looking for:
```C++
sregex_token_iterator pos(str.cbegin(), str.cend(), e, 0);  // default: match whole
sregex_token_iterator pos(str.cbegin(), str.cend(), e, 1);  // extract 1st submatch
sregex_token_iterator pos(str.cbegin(), str.cend(), e, 2);  // extract 2nd submatch
sregex_token_iterator pos(str.cbegin(), str.cend(), e, -1);  // extract unmatched tokens
```
#### Regex Replace
```C++
int main() {
	string str = "boq@gmail.com;  boqian@hotmail.com;;  bo_qian2000@163.com";
    
    regex e("([[:w:]]+)@([[:w:]]+)\.com");
    
    cout << regex_replace(str, e, "$1 is on $2");
    
}
```
`regex_replace` replaces all matched strings with `"$1 is on $2"`, where `$1` signifies the first submatch, `$2` the second submatch, etc.

`regex_replace` can take another paramater as a flag:
```C++
regex_replace(str, e, "$1 is on $2", regex_constants::format_no_copy);
// unmatched tokens are not copied to the new string; aka erased

regex_replace(str, e, "$1 is on $2",
	regex_constants::format_no_copy|regex_constants::format_first_only);
// format_first_only extracts only one match and stops there.
```
`regex` can also take flags:
```C++
regex e("...", regex_constants::grep|regex_constants::icase);
// use grep grammar and ignore case
```

## Clocks and Timers
`<chrono>` is a library for time and date. It is a precision-neutral library. It separates time points from clocks.

### Clock and Period/Frequency
What is a clock? There are 3 types of clocks.

1. System clock: `std::chrono::system_clock`
  * current time on the system - is not steady
2. Steady clock: `std::chrono::steady_clock`
  * goes at a uniform rate
3. High-Resolution clock: `std::chrono::high_resolution_clock`
  * provides smallest possible tick period

In C++, a clock's period (or frequency) is represented by a ratio. What is a ratio?
```C++
int main() {
	std::ratio<1, 10> r;  // 1/10
    cout << r.num << '/' << r.den << endl;
    
    cout << chrono::system_clock::period::num << '/' 
    	<< chrono::system_clock::period::den << endl;
    
	return 0;
}
```

### Time Duration
So far we have talked about clocks. Now, let's talk about time duration. How do you represent time duration? `std::chrono::duration<>` represents time duration. It requires a number and a unit.
```C++
duration<int, ratio<1,1>>  // represents number of seconds (1/1) stored as int
duration<double, ratio<60,1>  // number of minutes (60/1) stored as double
```
See here, we have arbitrary duration and precision! Here are pre-defined durations.

	system_clock::duration  -- duration<T, system_clock::period>
    chrono::nanoseconds
    chrono::microseconds
    chrono::milliseconds
    chrono::seconds
    chrono::minutes
    chrono::hours

Let's experiment with durations.
```C++
int main() {
	chrono::microseconds mi(2700);  // 2700 microseconds
    chrono::nanoseconds na = mi;  // 2700000 nanoseconds
    
    //chrono::milliseconds mill = mi;  // doesn't compile if high -> low precision
    chrono::milliseconds mil = chrono::duration_cast<chrono::milliseconds>(mi);
    // 2 milliseconds
    
    mi = mill + mi;  // 2000 + 2700 = 4700
    
    mi.count();  // 4700
    na.count();  // 2'700'000
}
```
### Time point
`std::chrono::time_point<>` represents a point in time. To represent a point in time, we need a reference point. In the Western World, the reference point is the year of Jesus' birthday. In the programming world, the reference point is `00:00 January 1, 1970 (Coordinated Universal Time - UTC)`. This reference is also called the "epoch of a clock".

For example,

	time_point<system_clock, milliseconds>
    
This is the point in time, according to system clock, in milliseconds with respect to the epoch.

You can see `time_point<>` requires 2 template parameters:

1. clock
2. time duration precision

Every clock defines its own time point:

* `system_clock::time_point` - `time_point<system_clock, system_clock::duration>`
* `steady_clock::time_point` - `time_point<steady_clock, steady_clock::duration>`

How can time points be used?
```C++
int main() {
	chrono::system_clock::time_point tp = chrono::system_clock::now();  // "current" time
    cout << tp.time_since_epoch().count() << endl;
    tp = tp + chrono::seconds(2);
    cout << tp.time_since_epoch().count() << endl;
    
    chrono::steady_clock::time_point start = chrono::steady_clock::now();
    cout << "I am bored" << endl;
    chrono::steady_clock::time_point end = chrono::steady_clock::now();
    chrono::steady_clock::duration d = end - start;
    if (d == chrono::steady_clock::duration::zero())
    	cout << "no time elapsed.\n";
    else
    	cout << "measurable time elapsed.\n";
    cout << chrono::duration_cast<chrono::microseconds>(d.count()) << endl;
}
```
Steady clocks are better suited for measuring time duration/span.


## Random Number Engine
**Changed in C++14**
Random Number Generation is made up of two parts: (1) engine, and (2) distribution.

### Random Engine
A random engine is a stateful generator that generates random values within predefined min and max. It is not truly random -- pseudorandom.
```C++
int main() {

	std::default_random_engine eng;
    cout << "Min: " << eng.min() << endl;
    cout << "Max: " << eng.max() << endl;
    
    cout << eng() << endl;
    cout << eng() << endl;
    
    std::stringstream state;
    state << eng;  // save the current state

    cout << eng() << endl;
    cout << eng() << endl;
    
    state >> eng;  // restore original state

    cout << eng() << endl;
    cout << eng() << endl;

	return 0;
}
```
The above shows you how to use the engine. Let's look at another use case:
```C++
void printRandom(std::default_random_engine e) {
	for (int i=0; i<10; i++)
    	cout << e() << " ";
    cout << endl;
}

int main() {
	std::default_random_engine e;
    std::default_random_engine e2;
    
    printRandom(e);
    printRandom(e2);
    
	return 0;
}
```
Both engines `e` and `e2` will print the same values because they are in the same state. How do we randomize the initial state/seed?
```C++
unsigned seed = std::chrono::steady_clock::now().time_since_epoch().count();
std::default_random_engine e3(seed);
printRandom(e3);  // random-ish, but still vulnerable to time-based attacks
   
e.seed();  // reset engine to initial state
e.seed(109);  // seed with 109
e2.seed(109);  // e2 and e are now in the same state
```
#### Shuffling
```C++
vector<int> d = {1,2,3,4,5,6,7,8,9};
std::shuffle(d.begin(), d.end(), std::default_random_engine());  // probably should seed
return 0;
```
