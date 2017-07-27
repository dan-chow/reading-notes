## Chapter 11: Nonblocking I/O

### Buffers

- From a programming perspective, the key difference between streams and channels is that streams are byte-based whereas channels are block-based. The second key difference between streams and channels/buffers is that channels and buffers tend to support both reading and writing on the same object.

- Besides its list of data, each buffer tracks four key pieces of information. All buffers have the same methods to set and get these values, regardless of the buffer’s type: position, capacity, limit, mark.

- The basic allocate() method simply returns a new, empty buffer with a specified fixed capacity.

- The ByteBuffer class (but not the other buffer classes) has an additional allocateDirect() method that may not create a backing array for the buffer. The VM may implement a directly allocated ByteBuffer using direct memory access to the buffer on an Ethernet card, kernel memory, or something else.

- If you already have an array of data that you want to output, you’ll normally wrap a buffer around it, rather than allocating a new buffer and copying its components into the buffer one at a time. 

- Buffers are designed for sequential access. Recall that each buffer has a curent position identified by the position() method that is somewhere between zero and the number of elements in the buffer, inclusive. The buffer’s position is incremented by one when an element is read from or written to the buffer. 

- Before you can read the data you wrote in out again, you need to flip the buffer.

- Each call to get() moves the position forward one. When the position reaches the limit, hasRemaining() returns false. This is called draining the buffer.

- If you know the ByteBuffer read from a SocketChannel contains nothing but elements of one particular primitive data type, it may be worthwhile to create a view buffer. This is a new Buffer object of appropriate type (e.g., DoubleBuffer, IntBuffer, etc.), that draws its data from an underlying ByteBuffer beginning with the current position. Changes to the view buffer are reflected in the underlying buffer and vice versa. However, each buffer has its own independent limit, capacity, mark, and position.

- Compacting shifts any remaining data in the buffer to the start of the buffer, freeing up more space for elements. Any data that was in those positions will be overwritten. The buffer’s position is set to the end of the data so it’s ready for writing more data.

- The duplicated buffers share the same data, including the same backing array if the buffer is indirect. Changes to the data in one buffer are reflected in the other buffer. Thus, you should mostly use this method when you’re only going to read from the buffers.

- Slicing a buffer is a slight variant of duplicating. Slicing also creates a new buffer that shares data with the old buffer. However, the slice’s zero position is the current position of the original buffer, and its capacity only goes up to the source buffer’s limit. That is, the slice is a subsequence of the original buffer that only contains the elements from the current position to the limit. Rewinding the slice only moves it back to the position of the original buffer when the slice was created. The slice can’t see anything in the original buffer before that point.

- Like input streams, buffers can be marked and reset if you want to reread some data. Unlike input streams, this can be done to all buffers, not just some of them.

- Two buffers are considered to be equal if:
• They have the same type (e.g., a ByteBuffer is never equal to an IntBuffer but may be equal to another ByteBuffer).
• They have the same number of elements remaining in the buffer.
• The remaining elements at the same relative positions are equal to each other.

### Channels

- Channels move blocks of data into and out of buffers to and from various I/O sources such as files, sockets, datagrams, and so forth.

- The SocketChannel class reads from and writes to TCP sockets. The data must be encoded in ByteBuffer objects for reading and writing. 

- The SocketChannel class does not have any public constructors. Instead, you create a new SocketChannel object using one of the two static open() methods.

- To read from a SocketChannel, first create a ByteBuffer the channel can store data in. Then pass it to the read() method

- The channel fills the buffer with as much data as it can, then returns the number of bytes it put there. When it encounters the end of stream, the channel fills the buffer with any remaining bytes and then returns –1 on the next call to read(). If the channel is blocking, this method will read at least one byte or return –1 or throw an exception. If the channel is nonblocking, however, this method may return 0.

- Socket channels have both read and write methods. In general, they are full duplex. In order to write, simply fill a ByteBuffer, flip it, and pass it to one of the write methods, which drains it while copying the data onto the output — pretty much the reverse of the reading process.

- Just as with regular sockets, you should close a channel when you’re done with it to free up the port and any other resources it may be using.

- The ServerSocketChannel class has one purpose: to accept incoming connections. You cannot read from, write to, or connect a ServerSocketChannel. The only operation it supports is accepting a new incoming connection.

- The static factory method ServerSocketChannel.open() creates a new ServerSocketChannel object. However, the name is a little deceptive. This method does not actually open a new server socket. Instead, it just creates the object. Before you can use it, you need to call the socket() method to get the corresponding peer ServerSocket. At this point, you can configure any server options you like, such as the receive buffer size or the socket timeout, using the various setter methods in ServerSocket. Then connect this ServerSocket to a SocketAddress for the port you want to bind to.

- accept() can operate in either blocking or nonblocking mode. In blocking mode, the accept() method waits for an incoming connection. It then accepts that connection and returns a SocketChannel object connected to the remote client. The thread cannot do anything until a connection is made. This strategy might be appropriate for simple servers that can respond to each request immediately. Blocking mode is the default. A ServerSocketChannel can also operate in nonblocking mode. In this case, the accept() method returns null if there are no incoming connections. Nonblocking mode is more appropriate for servers that need to do a lot of work for each connection and thus may want to process multiple requests in parallel. Nonblocking mode is normally used in conjunction with a Selector. To make a ServerSocketChannel nonblocking, pass false to its configureBlocking() method.

- Channels is a simple utility class for wrapping channels around traditional I/O-based streams, readers, and writers, and vice versa. It’s useful when you want to use the new I/O model in one part of a program for performance, but still interoperate with legacy APIs that expect streams. It has methods that convert from streams to channels and methods that convert from channels to streams, readers, and writers.

### Readiness Selection

- In order to perform readiness selection, different channels are registered with a Selector object. Each channel is assigned a SelectionKey. The program can then ask the Selector object for the set of keys to the channels that are ready to perform the operation you want to perform without blocking.

- After the different channels have been registered with the selector, you can query the selector at any time to find out which channels are ready to be processed. Channels may be ready for some operations and not others.

- When you know the channels are ready to be processed, retrieve the ready channels using selectedKeys().

- You iterate through the returned set, processing each SelectionKey in turn. You’ll also want to remove the key from the iterator to tell the selector that you’ve handled it. Otherwise, the selector will keep telling you about it on future passes through the loop.

- SelectionKey objects serve as pointers to channels. They can also hold an object attachment, which is how you normally store the state for the connection on that channel.