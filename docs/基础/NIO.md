## 核心

- channel 通道， 比喻为铁路
- buffer 缓冲区，比喻为列车
- selector 选择器

NIO是双向的，比喻为列车的卸货和装货

## 缓冲区

### 类型

对应基本数据类型，除了boolean

- ByteBuffer
- ShortBuffer
- IntBuffer
- LongBuffer
- FloatBuffer
- DoubleBuffer
- CharBuffer

### 核心属性

参见源代码 java.nio.Buffer

- capacity 最大容量，一旦声明不可修改
- limit 界限，limit后数据不能读写，初始时等于capacity
- position 位置
- mark 标记当前postition,通过reset方法回到remark位置

mark <= position <= limit <= capacity

### 核心方法

- allocate() 获取缓冲区
- put() 存入
- get() 取出
- flip() 切换到读模式，position指向开始位置，limit指向内容最大位置
- rewind() 重新读取，position指向开始位置
- clear() 清空，核心属性恢复初始值，缓冲区的数据仍旧存在

### 直接缓冲区与非直接缓冲区

- allocate() 缓冲区建立在JVM内存中(有误？核心态和用户态上下文切换？)
- allocateDirect() 缓冲区建立在物理内存中，可以提高效率，但是有风险，不可控
- buf.isDirect() 判断是否是直接缓冲区

## 通道

### DMA

- Direct Memory Access
- DMA可以直接控制总线和内存，但依附于CPU，需要向CPU申请权限
- DMA和CPU分时使用内存的方式
  - 停止CPU访问内存 DMA需要想CPU申请，让CPU放弃对地址总线、数据总线等相关控制总线的控制权，用完后归还
  - 周期挪用 即有DMA请求时优先执行DMA，可能CPU执行指令期间放弃执行交给DMA，这样会影响CPU指令的执行时间
  - DMA与CPU交替 C1、C2控制的多路转换器，转换几乎没有时间消耗，又称“透明的DMA”，就是会导致硬件复杂些
- 传统IO流就是使用DMA

### Channel

- 拥有独立处理器(IOPU)
- 不需要向CPU申请权限

### Channel接口 

- java.nio.channels.Channel
  - File
    - FileChannel
  - TCP Transmission Control Protocol
    - SocketChannel
    - ServerSocketChannel
  - UDP User Datagram Protocol
    - DatagramChannel

### 获取通道

- 方法一：针对各通道提供 getChannel()
  - 支持通道的类
    - 本地
      - FileInputStream/FileOutputStream
      - RandomAccessFile
    - 网络
      - Socket
      - ServerSocket
      - DatagramSocket
- 方法二：JDK7 NIO.2 针对各通道提供静态方法 open()
- 方法三：JDK7 NIO.2 Files.newByteChannel()

### 通道间的数据传输

- transferFrom()
- transferTo()

### 样例

JVM内存方式：

```java
FileInputStream fis = new FileInputStream("1.jpg");
FileOutputStream fos = new FileOutputStream("2.jpg");

FileChannel inChannel = fis.getChannel();
FileChannel outChannel = fos.getChannel();

ByteBuffer buf = ByteBuffer.allocate(1024);

while (inChanel.read(buf) != -1) {
  buf.flip();
  outChannel.write(buf);
  buf.clear();
}

outChannel.close();
inChannel.close();
fos.close();
fis.close();
```

直接内存方式：

```java
FileChannel inChannel = FileChannel.open(Paths.get("1.mkv"), StandardOpenOption.READ);
FileChannel outChannel = FileChannel.open(Paths.get("2.mkv"), StandardOpenOption.READ, StandardOpenOption.WRITE, StandardOpenOption.CREATE);

MappedByteBuffer inMappedBuf = inChannel.map(MapMode.READ_ONLY, 0, inChannel.size());
MappedByteBuffer outMappedBuf = outChannel.map(MapMode.READ_WRITE, 0, inChannel.size());

byte[] dst = new byte[inMappedBuf.limit()];
inMappedBuf.get(dst);
outMappedBuf.put(dst);

inChannel.close();
outChannel.close();
```

通道间数据传输方式：

```java
FileChannel inChannel = FileChannel.open(Paths.get("1.mkv"), StandardOpenOption.READ);
FileChannel outChannel = FileChannel.open(Paths.get("2.mkv"), StandardOpenOption.READ, StandardOpenOption.WRITE, StandardOpenOption.CREATE);

// inChannel.transferTo(0, inChannel.size(), outChannel);
outChannel.transferFrom(inChannel, 0, inChannel.size());

inChannel.close();
outChannel.close();
```

## Selector

以上文件读写没有非阻塞模式，非阻塞模式用于网络操作

```java
@Test
void server() throws IOException {
    ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
    serverSocketChannel.bind(new InetSocketAddress(8989));

    serverSocketChannel.configureBlocking(false);

    Selector selector = Selector.open();

    serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

    while (selector.select() > 0) {
        Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();

        while (iterator.hasNext()) {
            SelectionKey next = iterator.next();
            if (next.isAcceptable()) {
                SocketChannel socketChannel = serverSocketChannel.accept();
                socketChannel.configureBlocking(false);
                socketChannel.register(selector, SelectionKey.OP_READ);
            } else if (next.isReadable()) {
                SocketChannel socketChannel = (SocketChannel) next.channel();
                ByteBuffer buf = ByteBuffer.allocate(1024);
                socketChannel.read(buf);
                buf.flip();
                System.out.println(new String(buf.array(), 0, buf.limit()));
                socketChannel.close();
            }
            iterator.remove();
        }
    }

    serverSocketChannel.close();
}

@Test
void client() throws IOException {
    SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1", 8989));

    socketChannel.configureBlocking(false);

    ByteBuffer buf = ByteBuffer.allocate(1024);

    buf.put("你好吗".getBytes());
    buf.flip();
    socketChannel.write(buf);
    buf.clear();

    socketChannel.close();
}
```

## DatagramChannel

```java
@Test
void server() throws IOException {
    DatagramChannel datagramChannel = DatagramChannel.open();
    datagramChannel.bind(new InetSocketAddress(8989));

    ByteBuffer buf = ByteBuffer.allocate(1024);

    while (datagramChannel.receive(buf) != null) {
        buf.flip();
        System.out.println(new String(buf.array(), 0, buf.limit()));
        buf.clear();
    }

    datagramChannel.close();
}

@Test
void client() throws IOException {
    DatagramChannel datagramChannel = DatagramChannel.open();

    datagramChannel.connect(new InetSocketAddress("127.0.0.1", 8989));

    ByteBuffer buf = ByteBuffer.allocate(1024);

    buf.put("你好吗".getBytes());
    buf.flip();
    datagramChannel.write(buf);
    buf.clear();

    datagramChannel.close();
}
```

## Pipe

```java
@Test
void test() throws IOException {
    Pipe pipe = Pipe.open();
    Pipe.SinkChannel sink = pipe.sink();

    ByteBuffer buf = ByteBuffer.allocate(1024);

    buf.put("这个世界真美好".getBytes());

    buf.flip();
    sink.write(buf);

    Pipe.SourceChannel source = pipe.source();
    buf.flip();
    source.read(buf);

    System.out.println(new String(buf.array(), 0, buf.limit()));

    source.close();
    sink.close();
}
```