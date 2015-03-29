# New and Delete

## Demystifying Operator new/delete
What happens when the following code is executed?

	// allocate a dog on the heap
    dog* pd = new dog();

The following will happen:

1. operator new is called to allocate memory
2. dog's constructor is called to create dog
3. if step 2 throws an exception, call operator delete to free memory allocated in step 1

What happens if in step 1 an exception is thrown? Then step 3 will not occur, since step 3 only handles exceptions from step 2, not step 1.

Anyways, afterwards,

	delete pd;
    
The following will happen:

1. dog's destructor is called
2. operator delete is called to free memory

---

Here is `operator new` in more detail, and how it's implemented:
```C++
void* operator new(std::size_t size) throw(std::bad_alloc) {
	while (true) {
    	void* pMem = malloc(size);	// Allocate memory
        if (pMem)
        	return pMem;			// Return memory if successful
        
        std::new_handler Handler = std::set_new_handler(0);	// Get new handler
        std::set_new_handler(Handler);
        
        if (Handler)
        	(*Handler)();			// Invoke new handler
        else
        	throw bad_alloc();		// If new handler is null, throw exception
    }
}
```
The "new handler" is a function that is invoked when `operator new` failed to allocate memory. In other words, `new_handler` handles the failure event of memory allocation.

When memory allocation fails, the Handler is obtained (if not already obtained). If it is obtained successfully, the handler is invoked, which means the handler will try to free up memory space. If the handler could not be obtained (b/c there is no free space), an exception is thrown.

Here is a customized `operator new`:
```C++
class dog {
public:
	static void* operator new(std::size_t size) throw(std::bad_alloc) {
    	if (size == sizeof(dog))
    		customNewForDog(size);
        else
        	::operator new(size);
    }
};

class yellowdog : public dog {
	int age;
};

int main() {
	yellowdog *py = new yellowdog();
}
```
Recall that if you have polymorphism, you MUST have a virtual destructor for the base class. This is true even if you define a custom `operator delete` for the derived and base classes:
```C++
class dpg {
   public:
	static void operator delete(void *pMemory) throw() {
    	customDeleteForDog();
        free(pMemory);
    }
    virtual ~dog() = default;
};

class yellowdog : public dog {
   public:
	static void operator delete(void *pMemory) throw() {
    	customDeleteForYellowDog();
        free(pMemory);
    }
};
```
### Reasons for customizing new/delete

1. Usage error detection
  * memory leak detection or garbage collection
  * array index overrun/underrun
2. Improve efficiency
  * Cluster related objects to reduce page fault
  * Fix size allocation (good for applications with many small objects)
  * Align similar size objects to smae places to reduce fragmentation
3. Perform additional tasks
  * Fill the deallocated memory with 0's - security.
  * Collect usage statistics

---

_Before considering writing your own new/delete_

1. tweak your compiler towards your needs
2. use memory management library, e.g. Pool library from Boost


## Define your own New-Handler
The new-handler is a function that is invoked when `operator new` fails to allocate memory. Its purpose is to help memory allocation to succeed. `set_new_handler()` installs a new-handler and returns the current new-handler.
```C++
void* operator new(std::size_t size) throw(std::bad_alloc) {
	while (true) {
    	void* pMem = malloc(size);	// Allocate memory
        if (pMem)
        	return pMem;			// Return memory if successful
        
        std::new_handler Handler = std::set_new_handler(0);	// Get new-handler
        std::set_new_handler(Handler);
        
        if (Handler)
        	(*Handler)();			// Invoke new handler
        else
        	throw bad_alloc();		// If new handler is null, throw exception
    }
}
```
The new-handler must do one of the following things:

1. Make more memory available
2. Install a different new-handler
3. Uninstall the new-handler (pass a null pointer)
4. Throw an exception bad_alloc or its descendent
5. Terminate the program

Here is an example:
```C++
int main() {
	int *pGiant = new int[100000000000000L];  // bad_alloc b/c not enough memory!
    delete[] pGiant;
}
```
Here is another example:
```C++
void NoMoreMem() {
	std::cerr << "Unable to allocate memory, Victor" << endl;
}

int main() {
	std::set_new_handler(NoMoreMem);
    int *pGiant = new int[1000000000000000L];
    delete[] pGiant;
}
```
Here is a class-specific example:
```C++
class dog {
	int hair[1000000000000000L];
	std::new_handler origHandler;
public:
	static void NoMemForDog() {
    	std::cerr << "No more memory for dog, Victor" << endl;
        std::set_new_handler(origHandler);  // restore original new-handler
        throw std::bad_alloc();
    }
  	static void* operator new(std::size_t size) throw(std::bad_alloc) {
    	origHandler = std::set_new_handler(NoMemForDog);  // get original new-handler & set new-handler to custom
        void* pV = ::operator new(size);
        return pV;
    }
};
```
---

How does `shared_ptr` work with `operator new`?
```C++
int main() {
	std::shared_ptr<dog*> pd(new dog());
}
```
Recall that `shared_ptr` is responsible for destruction (when scope exits) (i.e. RAII).
