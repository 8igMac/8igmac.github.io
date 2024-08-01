---
title: 'What is "Resource Acquisition Is Initialization (RAII)" and how does it make C++ safe?'
date: 2024-07-27
---

> The basic idea is to represent a resource by a local object, so that the local object's destructor will release the resource. That way, the programmer cannot forget to release the resource.
>
> Bjarne Stroustrup's C++ Style and Technique FAQ

<figure>
  <img src="featured.jpg" alt="Leaking pipe." style="margin: 0 auto;">
  <figcaption style="text-align: center;">
    Photo by <a href="https://unsplash.com/@luis_tosta?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Luis Tosta</a> on <a href="https://unsplash.com/photos/tilt-shift-lens-photography-of-black-steel-faucet-SVeCm5KF_ho?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>
  </figcaption>
</figure>

Some languages like Java or Go have garbage collector that automatically
free unused resource at the cost of additional computational power.
C++ on the other hand gives you the freedom to control your 
own resource manually (via `new` and `delete`). 
But with great ability comes great responsibility. 
If one forget to release the resource after use, 
it might lead to resource leak (i.e., the available system resource
becoming less and less due to some process forget to return their
unused resource. Think of it as someone borrowing a book from the 
library, after finish reading it, he forgot to return the book.).

```cpp
void f1(int i)   // Bad: possible leak if exception is thrown.
{
    int* p = new int[12];
    // ...
    if (i < 17) throw Bad{"in f()", i};
    // ...
}

void f2(int i)   // Clumsy and error-prone: explicit release
{
    int* p = new int[12];
    // ...
    if (i < 17) {
        delete[] p;
        throw Bad{"in f()", i};
    }
    // ...
}
```
<figcaption style="text-align: center;">
Code snippet from <a href="https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#e6-use-raii-to-prevent-leaks">CppCoreGuidelines - Use RAII to prevent leaks</a>
</figcaption>

To make mistake is human, a lot of resource leak result from developers
forgetting to release the resource after use. What if we can have a 
mechanism that automatically release resource for us without having
to have a garbage collector that sacrifices runtime efficiency? This is when RAII come into play.

## What is RAII?
RAII stands for "resource acquisition is initialization" is a programming
technique that bind resource management to an object's life time to 
achieve automatic resource management. 
The concept consist of two part:

1. In the object's constructor we acquire resource from the system, if
the acquisition failed, we will throw an exception. 
2. In the object's destructor, we release the resource. So when the 
object go out of the scope, the 
destructor will be called and the resource shall be 
automatically released **even when exception is thrown**.

An example of implmenting RAII:
```cpp
#include <iostream>
#include <fstream>

class FileRAII {
private:
    std::fstream file;

public:
    // Constructor opens the file
    FileRAII(const std::string& filename) {
        file.open(filename, std::ios::in | std::ios::out | std::ios::app);
        // If resource acquisition failed, throw exception.
        if (!file.is_open()) {
            throw std::runtime_error("Failed to open file");
        }
        std::cout << "File opened\n";
    }

    // Destructor closes the file
    ~FileRAII() {
        if (file.is_open()) {
            file.close();
            std::cout << "File closed\n";
        }
    }

    // Function to write data to the file
    void write(const std::string& data) {
        if (file.is_open()) {
            file << data << std::endl;
        }
    }
};

int main() {
    try {
        FileRAII file("example.txt");
        file.write("Hello, RAII!");
    } catch (const std::exception& e) {
        std::cerr << "Exception: " << e.what() << std::endl;
    }
    // File will be closed automatically when file object goes out of scope
    return 0;
}
```

To be honest, I don't really like the name of RAII, because the 
name only tells half of the story. And the brilliant part of the 
story is the one being left out: The core idea of RAII is to 
**use a local object to represent the resource, so that the 
resource is automatically release when the local object's 
destructor is being invoked. This can prevent programmer from
forgetting to release the resource manually.** 
In my opinion, a better name would be "Resource deallocation is destruction" (RDID).

## You are probably using RAII but you don't know
You might think that such a useful concept should be applied
to everywhere right? You are right, there are a lot of useful 
utility provided by the STL which already implemented this technique,
such as smart pointers like `std::unique_ptr`. 
Smart pointer automatically free memory when its destructor is
being called. This greatly prevent the risk of memory leak and
should be the preferred solution over raw pointer.

## Conclusion
Key takeaway in this blog:
- The core idea of RAII is that resource allocation/release is tied to the lifetime of objects. And because destructors are automatically called when objects go out of scope, resources are released even if an exception is thrown. This provides a form of automatic cleanup and 
lower the risk of resource leak.

Now you understand RAII, try using it to manage your own resources!!

## Reference
- [RAII in Stroustrup's C++ FAQ](https://www.stroustrup.com/bs_faq2.html#finally)
- [cppreference - RAII](https://en.cppreference.com/w/cpp/language/raii)
- [CppCoreGuidelines - Use RAII to prevent leaks](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#e6-use-raii-to-prevent-leaks)