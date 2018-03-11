Neat trick — a template struct have traits inhereted by default from `std::false_type`, but there is exception of using as a template argument <void>, in which case it's from `std::true_type`.

    #include <iostream>
    template<typename T>   // primary template
    struct is_void : std::false_type
    {
    };
    template<>  // explicit specialization for T = void
    struct is_void<void> : std::true_type
    {
    };
    int main()
    {
        // for any type T other than void, the
        // class is derived from false_type
        std::cout << is_void<char>::value << '\n';
        // but when T is void, the class is derived
        // from true_type
        std::cout << is_void<void>::value << '\n';
    }

# pretty print of a struct

Source: https://stackoverflow.com/questions/3311182/linux-c-easy-pretty-dump-printout-of-structs-like-in-gdb-from-source-co Depending on circumstances you better be sure your structs are surrounded with `#pragma pack(push, 1)` and `#pragma pack(pop)`.

```
#include <stdio.h> //printf
#include <stdlib.h> //calloc, system
#include <unistd.h> // getpid

extern const char *__progname;

void printout_struct(const void* invar, const char* structname){
    /* dumpstack(void) Got this routine from http://www.whitefang.com/unix/faq_toc.html
    ** Section 6.5. Modified to redirect to file to prevent clutter
    */
    /* This needs to be changed... */
    char dbx[256];
    sprintf(dbx, "echo 'p (struct %s)*%p\n' > gdbcmds", structname, invar );
    system(dbx);
    sprintf(dbx, "echo 'where\ndetach' | gdb -batch --command=gdbcmds %s %d > struct.dump", __progname, getpid() );
    system(dbx);
    sprintf(dbx, "cat struct.dump");
    system(dbx);
    return;
}
```
