
`sys` is part of Python's standard library, so there's nothing to install — just `import sys`. It gives you access to variables and functions the interpreter itself uses: command-line arguments, the module search path, standard input/output streams, and details about the running Python installation.

**1. `sys.argv` — command-line arguments**  
Holds the arguments passed when you run a script. `argv[0]` is always the script's name.

python

```python
import sys
print("Script name:", sys.argv[0])
print("Arguments:", sys.argv[1:])
```

Running `python script.py hello world` prints:

```
Script name: script.py
Arguments: ['hello', 'world']
```

**2. `sys.exit()` — stop the program**  
Ends execution immediately. You can pass an exit code (`0` = success, non-zero = error).

python

```python
import sys

def check_age(age):
    if age < 0:
        print("Invalid age!")
        sys.exit(1)
    print("Age is:", age)

check_age(-5)
```

**3. `sys.path` — where Python looks for modules**  
A list of directories searched on `import`. You can append to it to make your own modules importable.

python

```python
import sys
sys.path.append("/my/custom/modules")
print(sys.path)
```


**4. `sys.platform` — which OS you're running on**

python

```python
import sys

if sys.platform.startswith("win"):
    print("Windows")
elif sys.platform == "darwin":
    print("macOS")
else:
    print("Linux/Unix:", sys.platform)
```

**5. `sys.version` / `sys.version_info` — Python version**

python

```python
import sys
print(sys.version)         # full version string, e.g. '3.11.4 (main, ...) [GCC ...]'
print(sys.version_info)    # e.g. sys.version_info(major=3, minor=11, micro=4, ...)

if sys.version_info >= (3, 8):
    print("Running Python 3.8+")
```

**6. `sys.stdin` / `sys.stdout` / `sys.stderr` — standard streams**

python

```python
import sys

sys.stdout.write("Hello, World!\n")        # same effect as print()
sys.stderr.write("Something went wrong\n") # goes to the error stream instead
```

**7. `sys.getsizeof()` — memory used by an object**

python

```python
import sys
print(sys.getsizeof(42))          # size of an int, in bytes
print(sys.getsizeof("hello"))     # size of a string
print(sys.getsizeof([1, 2, 3]))   # size of a list
```

**8. `sys.maxsize` — largest usable integer/index size**

python

```python
import sys
print(sys.maxsize)   # 9223372036854775807 on most 64-bit systems
```

**9. `sys.modules` — currently loaded modules**

python

```python
import sys
print("os" in sys.modules)   # False, until os is imported
import os
print("os" in sys.modules)   # True
```

**10. `sys.executable` — path to the Python interpreter**

python

```python
import sys
print(sys.executable)   # e.g. /usr/bin/python3 or C:\Python311\python.exe
```

## `sys.stdin` in Python

`sys.stdin` is the standard input stream — it lets your script read data typed by the user or piped in from another program. It behaves like a read-only file object.

**1. Reading a single line**

python

```python
import sys

line = sys.stdin.readline()
print("You entered:", line.strip())  # strip() removes the trailing newline
```

**2. Reading all input until EOF**  
Useful when input is piped in (e.g. from a file or another command) rather than typed interactively.

python

```python
import sys

data = sys.stdin.read()
print("Full input:", data)
```

**3. Reading line by line (looping)**  
`sys.stdin` is iterable — this is the most common pattern for processing piped input.

python

```python
import sys

for line in sys.stdin:
    print("Got:", line.strip())
```

Run it like:

bash

```bash
echo -e "hello\nworld" | python script.py
```

Output:

```
Got: hello
Got: world
```

**4. Reading all lines into a list**

python

```python
import sys

lines = sys.stdin.readlines()
print(lines)   # ['hello\n', 'world\n']
```

**5. `sys.stdin` vs `input()`**  
`input()` is the simpler, more common choice for interactive scripts — it also lets you show a prompt string.

python

```python
name = input("Enter your name: ")   # simpler, interactive
```

Use `sys.stdin` instead when:

- You want to process **piped/redirected input** (from files or other programs via `|` or `<`)
- You need to read **multiple lines** or an unknown amount of data at once
- You're processing input as a stream (e.g. a script that filters lines like `cat file | python script.py`)

**6. Example: a simple line-counting script**

python

```python
import sys

count = 0
for line in sys.stdin:
    count += 1

print(f"Total lines: {count}")
```

bash

```bash
cat somefile.txt | python count_lines.py
```

**7. Checking if input is being piped**

python

```python
import sys

if sys.stdin.isatty():
    print("Reading from keyboard (interactive)")
else:
    print("Reading from a pipe or redirected file")
```