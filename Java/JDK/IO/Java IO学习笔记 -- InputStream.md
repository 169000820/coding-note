# Java IO学习笔记 -- InputStream

- **InputStream** 是java提供的最基本的输入流, 位于 java.io 包下, 最基本的用法如下:

```java
public static void main(String[] args) {
    File file = new File("test.txt");
    // java7 以后提供的 try(resource) 语法, 编译器会自动关闭流资源, 否则需用 is.close()关闭
    // 流资源, 否则会消耗内存
    try(InputStream is = new FileInputStream(file)){
        int len = 0;
        // 当 is.read() 的值为 -1 时, 代表文件已经读完
        StringBuilder sb = new StringBuilder();
        while((len = is.read()) != -1) {
            sb.append((char)len);
        }
        System.out.println("文件内容为" + sb)
    }
}
```

* 此外, **InputStream** 还提供了两个<code>read()</code>的重置方法, 分别是 <code>read(byte b[])</code>和<code>read(byte b[],int off,int len)</code>, 下面是 <code>read(byte b[])</code>的用法。

```java
public static void main(String[] args){
    byte[] data = new byte[]{72, 101, 108, 108, 111, 33};
    // ByteArrayInputStream实际上是把一个byte[]数组在内存中变成一个InputStream，虽然实际应    //用不多，但测试的时候，可以用它来构造一个InputStream。
    try(InputStream is = new ByteArrayInputStream(data)) {
        byte[] b = new byte[10];
       	int len;
        whlile((len = is.read(b)) != -1) {
            // 此时 len 为一次读入的字节
            System.out.println("len" + len)
        }
    }
}
```

* 综合使用

```java
public static void main(string[] args){
    byte[] data = new byte[]{72, 101, 108, 108, 111, 33};
    try(InputStream is = new ByteArrayInputStream(data)) {
        String text = readString(is);
        System.out.println("text = " + text)
    } catch (IOException e){
        // 打印异常信息
        e.printStackTrace();
    }
}

/**
* @param input 输入流
*公用代码抽出来独立成为一个方法，参数为InputStream类型
*/
public static string readString(InputStream input) throws IOException {
    StringBuilder sb = new StringBuilder();
    int len;
    while((len = input.read()) != -1) {
        sb.append((char)len);
    }
    return sb.toString();
}
```



