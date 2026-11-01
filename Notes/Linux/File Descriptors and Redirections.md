A file descriptor in Unix/Linux OS is a reference, maintained by the kernel, that allows the system to manage Input/Output operations. It acts as a unique identifier for an open file, socket, or any other I/O resource. The file descriptor is the system's way of keeping tracking of active I/O connections, such as reading from or writing to a file.

By default the first three file descriptors in Linux are: 

1. Data stream for Input
	- `STDIN - 0`
2. Data Stream for Output
	- `STDOUT - 1`
3. Data Stream for Output that relates to an error occuring.
	- `STDERR - 2`

### Redirect STDIN Stream to a File 

We can also use the double lower-than character (`<<`) to add our standard input through a stream. We can use the so called End-Of-File (EOF) function to a Linux system file, which defines the input's end. In the next example, we will use the cat command to read out streaming input through the stream and direct it to a file called "stream.txt"

```
Crackhead2001@htb[/htb]$ cat << EOF > stream.txt
```

```
~ ❯ cat << EOF
∙ Abeer
∙ Gupta
∙ Manas
∙ EOF
Abeer
Gupta
Manas
```

