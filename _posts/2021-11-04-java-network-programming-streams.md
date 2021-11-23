A large part of what network programs do is simple input and output: moving bytes from one system to another. To a large extent, reading data a server sends you is not that different from reading a file, and sending text to a client is not that different from writing a file.

I/O in Java is built on *streams*. Input streams read data; output streams write data. All output streams have the same basic methods to write data and all input streams use the same basic methods to read data.

Streams are synchronous; that is, when a thread asks a stream to read or write a piece of data, it waits for the data to be read or write before it does anything else.

Java also offers nonblocking I/O using channels and buffers. Nonblocking I/O is a little more complicated, but can be much faster in some high-volume applications, such as web servers.

# Output Streams
Java's basic output class is **java.io.OutputStream**. Subclasses of **OutputStream** use these methods to write data onto particular media.

**OutputStream**'s fundamental method is **write(int b)**. This method is declared **abstract** because subclasses need to change it to handle their particular medium.

```java
	public static void generateCharacters(OutputStream out) throws IOException {
		int firstPrintableCharacter = 33;
		int numberOfPrintableCharacters = 94;
		int numberOfCharactersPerLine = 72;

		int start = firstPrintableCharacter;
		while (true) {
			for (int i = start; i < start + numberOfCharactersPerLine; i++) {
				out.write((i - firstPrintableCharacter) % numberOfPrintableCharacters + firstPrintableCharacter);
			}
			out.write('\r');
			out.write('\n');

			start = ((start + 1) - firstPrintableCharacter) % numberOfPrintableCharacters + firstPrintableCharacter;
		}
	}
```

Writing a single byte at a time is often inefficient. Consequently, most TCP/IP implementations buffer data to some extent. That is, they accumulate bytes in memory and send them to their eventual destination only when a certain number have accumulated or a certain amount of time has passed.

```java
	public static void generateCharactersV2(OutputStream out) throws IOException {
		int firstPrintableCharacter = 33;
		int numberOfPrintableCharacters = 94;
		int numberOfCharactersPerLine = 72;

		int start = firstPrintableCharacter;
		byte[] line = new byte[numberOfCharactersPerLine + 2];

		while (true) {
			for (int i = start; i < start + numberOfCharactersPerLine; i++) {
				line[i - start] =
					(byte) ((i - firstPrintableCharacter) % numberOfPrintableCharacters + firstPrintableCharacter);
			}
			line[72] = '\r';
			line[73] = '\n';

			out.write(line);

			start = ((start + 1) - firstPrintableCharacter) % numberOfPrintableCharacters + firstPrintableCharacter;
		}
	}
```

## flush and close
Streams can be buffered in software, directly in the Java code as well as in the network hardware. Consequently, if you are done writing data, it is important to flush the output stream. 

It is important to flush your streams whether you think you need to or not. Depending on how you got hold of a reference to the stream, you may or may not know whether it is buffered. Failing to flush when you need to can lead to unpredictable, unrepeatable program hangs that are extremely hard to diagnose if you don't have a good idea of what the problem is in the first place. You should flush all streams immediately before you close them. Otherwise, data left in the buffer when the stream is closed may get lost.

Finally, when you are done with a stream, close it by invoking its **close()** method. This releases any resources associated with the stream, such as file handles or ports. If the stream derives from a network connection, then closing the stream terminates the connection. Failure to close a stream in a long-running program can leak file handles, network ports, and other resources.

Dispose pattern example:
```java
	public void disposePatternExample() {
		OutputStream out = null;
		try {
			out = new FileOutputStream("/tmp/dispose_pattern_example.txt");
			out.write(new byte[] {'a', 'b', 'c'});
		} catch (IOException e) {
			System.err.println(e.getMessage());
		} finally {
			if (out != null) {
				try {
					out.close();
				} catch (IOException e) {
					// ignore
				}
			}
		}
	}
```

# Input Streams
Java's basic input class is **java.io.InputStream**. Concrete subclasses of **InputStream** use these methods to read data from particular media.

The basic method of **InputStream** is the noargs **read()** method. This method reads a single byte of data from the input stream's source and returns it as an **int** from 0 to 255. End of stream is signified by returning -1. The **read()** method waits and blocks execution of any code that follows it until a byte of data is available and ready to be read. Input and output can be slow, so if your program is doing anything else of importance, try to put I/O in its own thread. The **read()** method is declared **abstract** because subclasses need to change it to handle their particular medium.

```java
	public static byte[] readData(InputStream in) throws IOException {
		byte[] input = new byte[10];

		for (int i = 0; i < input.length; i++) {
			int b = in.read();
			if (b == -1) {
				break;
			}
			input[i] = (byte) b;
		}

		return input;
	}
```

Reading a byte at a time is as inefficient as writing data one byte a time. Consequently, there are two overloaded **read()** methods that fill a specified array with multiple bytes of data read from the stream **read(byte[] input)** and **read(byte[] input, int offset, int length)**. The first method attempts to fill the specified array **input**. The second attempts to fill the specified subarray of **input**, starting at **offset** and continuing for **length** bytes.

```java
	public static byte[] readData(InputStream in, int bytesToRead) throws IOException {
		byte[] input = new byte[bytesToRead];

		int bytesRead = 0;
		while (bytesRead < bytesToRead) {
			int result = in.read(input, bytesRead, bytesToRead - bytesRead);
			if (result == -1) {
				break;
			}

			bytesRead += result;
		}

		return input;
	}
```

# Filter Streams
Java provides a number of filter classes you can attach to row streams to translate the raw bytes to and from common data formats.

The filters come in two versions: the filter streams, and the readers and writers. The filter streams still work primarily with raw data as bytes. The readers and writers handle the special case of text in a variety of encodings such as UTF-8.

In most cases, the filter stream adds public methods with additional purposes. Sometimes these are intended to be used in addition to the usual **read()** and **write()** methods, like the **unread()** method of **PushbackInputStream**. At other times, they almost completely replace the original interface. For example, it's relatively rare to use the **write()** method of **PrintStream** instead of one of its **print()** and **println()** methods.

Filters are connected to streams by their constructors. Connection is permanent. Filters cannot be disconnected from a stream.

## BufferedOutputStream and BufferedInputStream
The **BufferedOutputStream** class stores written data in a buffer until the buffer is full or the stream is flushed. Then it writes the data onto the underlying output stream all at once.

The **BufferedInputStream** class also has  a protected byte array that serves as a buffer. When one of the stream's **read()** method is called, it first tries to get the requested data from the buffer. Only when the buffer runs out of data does the stream read from the underlying source. At this point, it reads as much data as it can from the source into the buffer, whether it needs all the data immediately or not. Data that isn't used immediately will be available for later invocations of **read()**.

## PrintStream
**System.out** is a **PrintStream**.

**PrintStream** is evil and network programmers should avoid using it. The first problem is that the output from **println()** is platform dependent. The second problem is that **PrintStream** assumes the default encoding of the platform on which it's running. The third problem is that **PrintStream** eats all exceptions.

## DataInputStream and DataOutputStream
The **DataInputStream** and **DataOutputStream** classes provide methods for reading and writing Java's primitive data types and strings in a binary format. What a data output stream writes, a data input stream can read.

# Readers and Writers
Many programmers have a bad habit of writing code as if all text were ASCII or at least in the native encoding of the platform.

Java provides an almost complete mirror of the input and output stream class hierarchy designed for working with characters instead of bytes. In this mirror image hierarchy, two abstract superclasses define the basic API for reading and writing characters. The **java.io.Reader** class specifies the API by which characters are read. The **java.io.Writer** class specifies the API by which characters are written.

The most important concrete subclasses of **Reader** and **Writer** are the **InputStreamReader** and the **OutputStreamWriter** classes. An **InputStreamReader** contains an underlying input stream from which it reads raw bytes. It translates these bytes into Unicode characters according to a specified encoding. An **OutputStreamWriter** receives Unicode characters from a running program. It then translates those characters into bytes using a specified encoding and writes the bytes onto an underlying output stream.

## Writer and OutputStreamWriter
```java
w.write("network");
```
How many and which bytes are written depends on the encoding of **w**.

Writers may be buffered, either directly by being chained to a **BufferedWriter** or indirectly because their underlying output stream is buffered. To force a write to be committed to the output medium, invoke the **flush()** method.

The **close()** method flushes the writer, then closes the underlying output stream and releases any resources associated with it.

**OutputStreamWriter** is the most important concrete subclass of **Writer**. An **OutputStreamWriter** receives characters from a Java program. It converts these into bytes according to a specified encoding and writes them onto an underlying output stream. Its constructor specifies the output stream to write and the encoding to use. 

Default character sets can cause unexpected problems at unexpected times. You're generally almost always better off explicitly specifying the character set rather than letting Java pick one for you.

## Reader and InputStreamReader
The **read()** method returns a single Unicode character as an **int** with a value from 0 to 65535 or -1 on end of stream. (Technically it returns a UTF-16 code point, though almost always that's the same as a Unicode character.) 

**InputStreamReader** is the most important concrete subclass of **Reader**. An **InputStreamReader** reads bytes from an underlying input stream. It converts these into characters according to a specified encoding and returns them. The constructor specifies the input stream to read from and the encoding to use.

## filter readers and writers
The **InputStreamReader** and **OutputStreamWriter** classes act as decorators on top of input and output streams that change the interface from a byte-oriented interface to a character-oriented interface. Once this is done, additional character-oriented filters can be layered on top of the reader or writer using the **java.io.FilterReader** and **java.io.FilterWriter** classes. As with filter streams, there are a variety of subclasses that perform specific filtering.

**BufferedReader** and **BufferedWriter** classes are the character-based equivalents of the byte-oriented **BufferedInputStream** and **BufferedOutputStream** classes. Where **BufferedInputStream** and **BufferedOutputStream** use an internal array of bytes as a buffer, **BufferedReader** and **BufferedWriter** use an internal array of chars.

The **PrintWriter** class is a replacement for Java 1.0's **PrintStream** class that properly handles multibyte character sets and international text. Unfortunately, **PrintWriter** is still not good enough for network programming, and it still has the problems of platform dependency and minimal error reporting that plague **PrintStream**.