---
status: publish
published: false
title: LD_PRELOAD Hooking
author: horus
categories:
  - linux
tags:
  - linux
  - reversing
  - hooking
comments: true
image: /assets/img/images/ld.png
---
## Introduction

Hello!

My professor gave us an interesting task. We needed to make a program that takes a binary with a password and allows us to run that program using a new password.

The challenge was that **patching the binary was not allowed**.

At first, I thought about binary patching. The idea was simple: find the password inside the binary, replace it with a new one, and generate a new binary.

But since patching was not allowed, I started thinking about another way.

What if I don't change anything inside the binary, but instead change its behavior while it is running?

This idea reminded me a little bit of game cheats and runtime hooking. Instead of modifying the program on disk, we can hook a function while the program is running and change what that function does.

So I started searching about function hooking and found the `LD_PRELOAD` technique on Linux.

`LD_PRELOAD` allows us to load our own shared library before the normal libraries used by a program. This means that if the program calls a function like `strcmp()`, we can provide our own version of that function.

The idea looks like this:

![](/assets/img/images/hook.png)

So instead of patching the binary or changing the password inside it, I decided to hook the function responsible for checking the password and change its behavior at runtime.

For my test, I decided to hook `strcmp()`.

The following code gets the real `strcmp()` function using `dlsym()`. Then it reads a new password from an environment variable called `NEW_PASSWORD`.

If the password entered by the user matches `NEW_PASSWORD`, my hooked function returns `0`, which means the two strings are equal.

```c
#include <dlfcn.h>
#include <stdlib.h>
#include <string.h>

typedef int (*strcmp_t)(const char *, const char *);

int strcmp(const char *a, const char *b)
{
    static strcmp_t real_strcmp = NULL;

    if (real_strcmp == NULL) {
        real_strcmp = (strcmp_t)dlsym(RTLD_NEXT, "strcmp");
    }

    const char *new_password = getenv("NEW_PASSWORD");

    if (new_password != NULL &&
        real_strcmp(a, new_password) == 0) {
        return 0;
    }

    return real_strcmp(a, b);
}
```

To use this as an `LD_PRELOAD` library, I compiled it as a shared object:

```bash
gcc -shared -fPIC hook.c -o hook.so -ldl
```

Now I needed a simple program to test it on, so I wrote this password checker:

```c
#include <stdio.h>
#include <string.h>

int main(void)
{
    char input[100];

    printf("Password: ");
    scanf("%99s", input);

    if (strcmp(input, "SECRET_WE_DONT_KNOW") == 0)
        puts("Access granted");
    else
        puts("Access denied");

    return 0;
}
```

Normally, the program only accepts the password stored inside the binary.

But when we run it with our `LD_PRELOAD` library, we can change the behavior of `strcmp()` at runtime.

I also made a small launcher to make the process easier for the user.

The launcher takes two arguments:

```text
<target-program> <new-password>
```

Then it:

1. Sets `NEW_PASSWORD`.
    
2. Sets `LD_PRELOAD` to our `hook.so`.
    
3. Executes the target program.
    

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

int main(int argc, char *argv[])
{
    if (argc != 3) {
        fprintf(stderr,
                "Usage: %s <target-program> <new-password>\n",
                argv[0]);
        return 1;
    }

    const char *target = argv[1];
    const char *new_password = argv[2];

    char preload_path[4096];

    if (getcwd(preload_path, sizeof(preload_path)) == NULL) {
        perror("getcwd");
        return 1;
    }

    size_t len = strlen(preload_path);

    if (len + strlen("/hook.so") + 1 >= sizeof(preload_path)) {
        fprintf(stderr, "Path too long\n");
        return 1;
    }

    strcat(preload_path, "/hook.so");

    /* Pass the password to the hook */
    if (setenv("NEW_PASSWORD", new_password, 1) != 0) {
        perror("setenv NEW_PASSWORD");
        return 1;
    }

    /* Load our shared library */
    if (setenv("LD_PRELOAD", preload_path, 1) != 0) {
        perror("setenv LD_PRELOAD");
        return 1;
    }

    /* Run the target */
    execl(target, target, NULL);

    perror("execl");
    return 1;
}
```

After compiling the launcher, I can run it like this:

```bash
./launcher ./target MyNewPassword
```

The launcher automatically sets everything up and runs the target with our hooked `strcmp()` function.

The important thing here is that **the original binary is never modified**.

The password inside the binary is still the same. We are only changing the behavior of the program while it is running.

So the final flow looks something like this:
![[Pasted image 20260831055935.png]]

This was a nice way to learn about `LD_PRELOAD`, shared libraries, dynamic linking, and function hooking without modifying the original binary.
![[Pasted image 20260831060329.png]]


### Resources :
- https://www.netspi.com/blog/technical-blog/network-pentesting/function-hooking-part-i-hooking-shared-library-function-calls-in-linux/
- https://h0mbre.github.io/Learn-C-By-Creating-A-Rootkit/