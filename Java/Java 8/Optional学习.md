# Optional 学习笔记

Java 里提供了一个 Java.Util.Option<T> 类来改善常见的 NullPointerException 异常问题 。

使用：

1. 创建 Optional ，创建 Optional 一共有三种方式：

   ```java
public class Person {
   	private String name;
	private Integer age;
   	private GenderEnum gender;
   			
   	public enum GenderEnum {
   		Men(1), Women(0);
   		private int code;
   		Gender(code){ // 枚举类的构造方法默认为 private
   			this.code = code;
   		}
   	}
   }
   ```
   
   
   
   * 使用静态工程方法 <code>Option.empty（）</code>创建，代码如下：
   
     ```java
     Optional<Person> optPerson = Option.empty();
     
     /**
     	Optional.empty() 源码如下：
     		// private static final Optional<?> EMPTY = new Optional<>();
     		pulic static <T> Optional<T> empty() {
     			Optional<T> t  = (Optional<T>) EMPTY;
     			return t;
     		}
     */
     ```
   
     
   
   * 使用静态方法 <code>Optional.of()</code>创建，代码如下：
   
     ```java
     Person p = new Person();
     // 如果 p 为空，会抛出 NullPointerException
     Optional<Person> optPerson = Optinoal.of(p);
     /**
     	Optional.of() 源代码如下:
         	public static <T> Option<T> of(T value) {
         		return new Optional<>(value);
         	}
     */
     ```
   
   * 使用静态方法 <code>Optional.ofNullable()</code>创建一个可以为空的对象，代码如下：
   
     ```java
     Person p = new Person();
     // 如果 p 为空，使用 get() 获取的是否会抛出 NoSuchElementException
     Optional<Person> optPerson = Optional.ofNullable(p);
     
     /**
     	Optional.ofNullable() 源代码如下：
     		public static <T> Optional<T> ofNullable(T value) {
     			return value == null ? empty() : of(value);
     		}
     */
     ```

2. Optional 内部方法介绍：

   - `isPresent():boolean` 如果值存在返回 true，否则返回 false：

     ```java
     public boolean isPresent() {
         retrun this.value != null;
     }
     ```

   * `isPresent(Consumer<? extent T> consumer):boolean`

   * `get()` 如果有值则返回，否则会抛出 `NoSuchElementException`, 示例：

     ```java
     try {
     	Optional.empty().get();
     } catch (NoSuchElementException e) {
         e.printStackTrace();
     }
     ```

   * `filter(Predicate<? super T> predicate)::void`

   * `map(Function<? super T,? extends U>):Optional<U>`

   * `flatMap(Function<? super T, Optional<U> maper):Optional<U>`

   * `orElse(T other):T` 如果有值则返回，没有则返回 other 提示信息， 示例：

     ```java
     Optional.empty()
         	.orElse("value is not found");
     /**
     	Optional.empty() 源码如下：
     		public T orElse(T other) {
     			return value != null ? value : other;
     		}
     */
     ```

   - `orElseGet(Supplier<? extend T> other):` 如果有值就返回，可接受 supplier 来生成默认值：

     ```java
     Optional.empty()
         	.orElseGet(() -> System.out.println("there is no value"));
     /**
     	Optional.OrElseGet 源码如下：
     		public T orElseGet(Supplier<? extend T> other) {
     			return value != null ? value : other.get();
     		}
     */
     ```

   * `orElseThrow(Supplier<? extends X> exceptionSupplier):T` 如果有值则将其返回，否则抛出`Supplier`接口创建的异常。实例:
   
     ```java
     try {
         Optional.empty().orElseThrow(NoSuchElementException::new);
     } catch (Exception e) {
         e.printStackTrace();
     }
     /**
     
     orElseThrow源码如下:
     	public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
             if (value != null) {
                 return value;
             } else {
                 throw exceptionSupplier.get();
             }
         }
     */
     ```
   
     



