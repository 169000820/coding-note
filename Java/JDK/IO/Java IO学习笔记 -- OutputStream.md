# Java IO 学习笔记 -- OutputStream

* 和 <kbd>InputStream</kbd>一样, <kbd>OutputStream</kbd>也是<kbd>Java</kbd>提供的最基本的输出流。基本示例:

  ```java
   public static void main(String[] args) throws IOException {
          File file = new File("out/readme.txt");
          if (!file.exists()) {
              System.out.println(file.getParent());
              File parent = new File(file.getParent());
              // 递归创建文件
              if (!parent.exists()) {
                  boolean mkdirs = parent.mkdirs();
                  System.out.println("mkdirs = " + mkdirs);
              }
              boolean res = file.createNewFile();
              System.out.println("res = " + res);
          }
       // 写入到文件中
        try (OutputStream os = new FileOutStream(file)) {
              os.write("Hello ".getBytes(StandardCharsets.UTF_8));
              os.write("World!".getBytes(StandardCharsets.UTF_8));
          } catch (IOException e) {
             e.printStrackTrace;
          }
          byte[] data = null;
          try (ByteArrayOutputStream os = new ByteArrayOutputStream()) {
              os.write("Hello ".getBytes(StandardCharsets.UTF_8));
              os.write("World!".getBytes(StandardCharsets.UTF_8));
              data = os.toByteArray();
          } catch (IOException e) {
             e.printStrackTrace;
          }
          System.out.println(new String(data, StandardCharsets.UTF_8));
      }
  ```

  和`InputStream`类似，`OutputStream`也提供了`close()`方法关闭输出流，以便释放系统资源。要特别注意：`OutputStream`还提供了一个`flush()`方法，它的目的是将缓冲区的内容真正输出到目的地。

  为什么要有`flush()`？因为向磁盘、网络写入数据的时候，出于效率的考虑，操作系统并不是输出一个字节就立刻写入到文件或者发送到网络，而是把输出的字节先放到内存的一个缓冲区里（本质上就是一个`byte[]`数组），等到缓冲区写满了，再一次性写入文件或者网络。对于很多IO设备来说，一次写一个字节和一次写1000个字节，花费的时间几乎是完全一样的，所以`OutputStream`有个`flush()`方法，能强制把缓冲区内容输出。

  通常情况下，我们不需要调用这个`flush()`方法，因为缓冲区写满了`OutputStream`会自动调用它，并且，在调用`close()`方法关闭`OutputStream`之前，也会自动调用`flush()`方法。

  但是，在某些情况下，我们必须手动调用`flush()`方法。举个栗子：

  小明正在开发一款在线聊天软件，当用户输入一句话后，就通过`OutputStream`的`write()`方法写入网络流。小明测试的时候发现，发送方输入后，接收方根本收不到任何信息，怎么肥四？

  原因就在于写入网络流是先写入内存缓冲区，等缓冲区满了才会一次性发送到网络。如果缓冲区大小是4K，则发送方要敲几千个字符后，操作系统才会把缓冲区的内容发送出去，这个时候，接收方会一次性收到大量消息。

  解决办法就是每输入一句话后，立刻调用`flush()`，不管当前缓冲区是否已满，强迫操作系统把缓冲区的内容立刻发送出去。

  实际上，`InputStream`也有缓冲区。例如，从`FileInputStream`读取一个字节时，操作系统往往会一次性读取若干字节到缓冲区，并维护一个指针指向未读的缓冲区。然后，每次我们调用`int read()`读取下一个字节时，可以直接返回缓冲区的下一个字节，避免每次读一个字节都导致IO操作。当缓冲区全部读完后继续调用`read()`，则会触发操作系统的下一次读取并再次填满缓冲区。