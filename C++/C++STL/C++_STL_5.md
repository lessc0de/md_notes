# C++ Standard Template Library

## Stream Introduction
Stream is a C++ Input/Output Library. Everyone knows this example:
```C++
	cout << "Hello" << endl;
```
What is `cout`? It's a global object of `ostream`, which lives in `std` namespace. What is `ostream`? It is really:
```C++
	typedef basic_ostream<char> ostream
```
What is `<<`? It is a member function of `ostream`:
```C++
	ostream& ostream::operator<< (string v);
```
Notice that `<<` returns `ostream&`. This makes it able to string together things like this:
```C++
	cout << "Hello" << "what" << endl;
```
We will learn more about `endl` later.

### What is a stream?
Stream is technically a serial IO interface. It handles data one-by-one. These interfaces are with respect to external devices (file, stdin/stdout, network, etc.)
```C++
string s("Hello");
s[3] = 't';  // random access
```
Above, we have random access to a string. However, streams do not have random access. You cannot do this:
```C++
	cout[3] = 't';  // error
```
Again, streams work on data one-by-one.

### `ofstream` (file output stream)
```C++
{
ofstream of("MyLog.txt");  // Create a file write-stream (o for "output", or "writing")
of << "Experience is the mother of wisdom" << endl;

of << 234 << endl;  // writes "234\n" to file
of << 2.3 << endl;  // writes "2.3\n" to file
of << bitset<8>(14);  // writes 00001110 to file
of << complex<int>(2,3);  // writes (2,3)
}  // RAII works with streams!
```

---

Every IO operation formats the data and/or communicates with external devices for data. No matter what the device is, you use the same stream API. This is called the Software Engineer Principle: low coupling.


## File Stream and Error Handling
Recall the `ofstream` file output-stream:
```C++
{
ofstream of("MyLog.txt");  // Create a file write-stream (o for "output", or "writing")
of << "Experience is the mother of wisdom" << endl;

of << 234 << endl;  // writes "234\n" to file
of << 2.3 << endl;  // writes "2.3\n" to file
of << bitset<8>(14);  // writes 00001110 to file
of << complex<int>(2,3);  // writes (2,3)
}  // RAII works with streams!
```
Today, we'll talk more about the file stream modes. Consider append mode:
```C++
{
//ofstream of("MyLog.txt");  // Open the file for write, clear the content of the file
ofstream of("MyLog.txt", ofstream::app);  // open the file for write in append mode
of << "Honesty is the best policy." << endl;
}
 ```   
Consider insert mode:
```C++
{
ofstream of("MyLog.txt", ofstream::in | ofstream::out);  // read-write mode
ofseekp(10, ios::beg);  // requires ofstream::in; ios::beg moves output ptr
// to beginning of file (requires ofstream::out)

of << "12345";  // overwrites 5 chars
of.seekp(-5, ios::end);  // move output ptr 5 chars before end.
of << "Nothing ventured, nothing gained." << endl;  // write starting 5 chars before end
of.seekp(-5, ios::cur);  // move output ptr 5 chars before current ptr
}
```
Just as there is an `ofstream`, there is an `ifstream` for file input-stream:
```C++
{
ifstream inf("MyLog.txt");
int i;
inf >> i;  // read one word
}
```
### File Error Handling

Notice above that `inf` will parse into integer. If there is a parsing error, an error will occur. The error status is indicated by "error status". There are 4 bits in the error status: `goodbit`, `badbit`, `failbit`, and `eofbit`. For example, `0010` means only the `failbit` is set.
```C++
inf.good();  // everything is OK (goodbit set to 1)
inf.bad();  // non-recoverable error (badbit set to 1)
inf.fail();  // failed but recoverable stream operation (failbit and badbit set to 1)
inf.eof();  // end of file (eofbit set to 1)

inf.clear();  // clear all error status
inf.clear(ios::badbit);  // set badbit flag to 1, everything else 0
inf.clear(inf.rdstate() & ~ios::failbit);  // clear only fail bit

if (inf)  // equivalent to if (!inf.fail())
	cout << "read successfully\n";
    
if (inf >> i)  // remember, operator>> returns a reference to inf (istream&)
	cout << "Read successfuly" << i << endl;
```    
You can also handle errors through exceptions:
```C++
inf.exceptions(ios::badbit | ios::failbit);  // setting the exception mask
// when badbit and failbit set to 1, exception of ios::failure will be thrown
// when eofbit set to 1, no exception will be thrown
inf.exceptions(ios::goodbit);  // no exception will be generated
```
## Formatted and Unformatted IO
One main purpose of the stream class is to format/parse data.

### Integer formatting
```C++
cout << 34 << endl;  // prints 34
cout.setf(ios::oct, ios::basefield);  // set format to octal number & show base
cout << 34 << endl;  // prints 42
cout.setf(ios::showbase);
cout << 34 << endl;  // prints 042
cout.setf(ios::hex, ios::basefield);
cout << 34 << endl;  // prints 0x22
cout.unsetf(ios::showbase);  // hide base information
cout << 34 << endl;  // prints 22

cout.setf(ios::dec, ios::basefield);

cout.width(10);
cout << 26 << endl;  // prints ________26, where _ is space character
cout.setf(ios::left, ios::adjustfield);
cout << 26 << endl;  // prints 26

int i;
cin.setf(ios::hex, ios::basefield);
cin >> i;  // Enter: 12, i: 18

cout.flags(ios::oct | ios::showbase);
ios::fmtflags f = cout.flags();  // cout format flags, which are equal to above
``` 
### Floating point formatting
```C++
cout.setf(ios::scientific, ios::floatfield);
cout << 340.1 << endl;  // prints 3.401000e+002
cout.setf(ios::fixed, ios::floatfield);
cout << 340.1 << endl;  // 340.100000
cout.precision(3);
cout << 340.1 << endl;  // 340.100
```
### Unformatted or Raw Text
```C++
char buf[80];

// input
{
ifstream inf("MyLog.txt");
inf.get(buf, 80);  // read up to 80 chars and save into buf
inf.getline(buf, 80);  // read up to 80 chars or until '\n'
inf.read(buf, 20);  // read 20 chars
inf.ignore(3);  // ignore the next 3 chars in stream
inf.peek();  // peek at the char at the top of the stream
inf.unget();  // return prev char back into the stream
inf.putback('z');  // put back 'z' instead of prev char into stream
inf.get();  // get only 1 char
inf.gcount();  // return the number of chars being read by last unformatted read: 1
}

// output
{
ofstream of("MyLog.txt");
of.put('c');
of.write(buf, 6);  // write first 6 chars to buf
of.flush();  // force flush output
}
```  

## Stream Manipulators
Ever wondered how `endl` worked? Is it an object? An built-in data type? A function? It looks like an object, but no. It is a function:
```C++
ostream& endl(ostream& sm) {
	sm.put('\n');
    sm.flush();
    return sm;
}
```
`endl` puts a newline, flushes the stream, then returns the stream object as reference.

Notice that `<<` looks similar:
```C++
ostream& ostream::operator<< (ostream& (*func)(ostream&)) {
	return (*func)(*this);
}
```
`ostream& (*func)(ostream&)` means a (pointer to a) function which takes an `ostream&` as parameter and returns `ostream&`. This type signature is exactly that of `endl`. We can thus see that `<<` was made with `endl` in mind.

Inside the function, `(*func)(*this)` basically invokes `func` on `*this`, which is `cout`, the `ostream&` we're looking at.
```C++
	cout << "Hi" << endl;
```
Note that if we don't call `endl`,
```C++
	cout << "Hi";
```
then we just use another version of the overloaded `operator<<`:
```C++
	ostream& ostream::operator<< (ArithmeticType val);
```
---

`endl` is called a **stream manipulator**. The standard library has many manipulators:
```C++
cout << ends;  // '\0'
cout << flush;  // flush stream
cin >> ws;  // read and discard white spaces

cout << setw(8) << left << setfill('_') << 99 << endl;  // 99______\n
// setw(8): set width to 8
// left: left align
// setfill('_'): set fill to '_'
// 99: output 99
// endl: put newline and flush

cout << hex << showbase << 14;  // 0xe
```

## Stream Buffer
IO operations have two steps: formatting the data, and communicating with external devices for data. Note that the stream module deals with formatting data. Then what is responsible with communicating with external devices? **stream buffer**.
```C++
cout << 34;
streambuf* pbuf = cout.rdbuf();  // pointer to internal buffer

ostream myCout(pbuf);  // create another ostream with pbuf as internal buffer
myCout << 34;  // 34 to stdout

// format!
myCout.setf(ios::showpos);  // show positive side of number
myCout.width(20);  // set width to 20 (default aligned to right)
myCout << 12 << endl;  // prints                  +12
cout << 12 << endl;  // prints 12
```
Here is another example:
```C++
ofstream of("MyLog.txt");  // open for writing
streambuf* origBuf = cout.rdbuf();
cout.rdbuf(of.rdbuf());  // set cout's rdbuf to of's rdbuf
cout << "Hello" << endl;  // write "Hello" to MyLog.txt, not stdout

// this is called Redirecting
// let's restore cout's original internal buffer
cout.rdbuf(origBuf);

cout << "Goodbye" << endl;  // "Goodbye" to stdout
```
### Stream Buffer Iterator
Another way to access the internal buffer is with iterators. These are essentially "stream iterators".
```C++
istreambuf_iterator<char> i(cin);
ostreambuf_iterator<char> o(cout);

// copy i to o until 'x' is seen
while(*i != 'x') {
	*o = *i;
    ++o; ++i;
}
// do the same as above but more terse
copy(i, istreambuf_iterator<char>(), o);
```
Compare this to the similar stream iterator:
```C++
istream_iterator<string>(cin);
ostream_iterator<string>(cout, " ");
```
What is the difference? Recall that stream buffers are for communicating with external devices. Streams, on the other hand, are responsible for formatting the data. Thus, the difference is that `istreambuf_iterator`/`ostreambuf_iterator` does absolutely **NO** formatting, while `istream_iterator`/`ostream_iterator` do (of course, with type templating as seen above with `string`; try changing `<string>` to `<int>`, and it'll still work).

