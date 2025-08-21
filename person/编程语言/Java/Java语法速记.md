# Java 基础速记手册

## 变量定义与访问

```java
// 基本定义
int a = 10;
String b = "hello";
boolean c = true;

// 常量定义
final double PI = 3.14;
final int MAX_USERS = 100;

// 变量作用域
public int publicVar;    // 公共
protected int protVar;  // 受保护
private int privVar;    // 私有
static int classVar;    // 类变量
transient int tempVar;  // 不被序列化
volatile int sharedVar; // 多线程可见

// 类型推断 (Java 10+)
var list = new ArrayList<String>();
```

## 数据类型

```java
// 基本类型
byte, short, int, long, float, double, char, boolean

// 包装类
Byte, Short, Integer, Long, Float, Double, Character, Boolean

// 引用类型
String, Array, Class, Interface

// 类型转换
int i = (int) 3.14;      // 强制转换
double d = i;            // 自动提升
String s = String.valueOf(123); // 转字符串
int num = Integer.parseInt("123"); // 字符串转数字
```

## 数组操作

```java
// 数组定义
int[] arr1 = new int[5];
int[] arr2 = {1, 2, 3};
String[][] multiArr = new String[2][3];

// 数组操作
arr.length;              // 数组长度
Arrays.sort(arr);        // 排序
Arrays.copyOf(arr, 10);  // 复制
Arrays.fill(arr, 0);     // 填充
Arrays.equals(arr1, arr2); // 比较
Arrays.toString(arr);    // 转字符串

// 数组遍历
for (int i = 0; i < arr.length; i++) {}
for (int num : arr) {}
Arrays.stream(arr).forEach(System.out::println);
```

## 字符串操作

```java
// 字符串定义
String s1 = "hello";
String s2 = new String("world");

// 常用方法
s1.length();             // 长度
s1.concat(s2);           // 连接
s1.equals(s2);           // 比较内容
s1.equalsIgnoreCase(s2); // 忽略大小写比较
s1.substring(1, 3);      // 子串
s1.indexOf("ll");        // 查找
s1.replace("l", "L");    // 替换
s1.toUpperCase();        // 转大写
s1.split(",");           // 分割
String.join("-", "a", "b", "c"); // 连接

// StringBuilder/StringBuffer
StringBuilder sb = new StringBuilder();
sb.append("Hello").append(" ").append("World");
String result = sb.toString();
```

## 类与对象

```java
// 类定义
public class Person {
    // 字段
    private String name;
    private int age;
    
    // 构造方法
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    // 方法
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    // 静态方法
    public static Person create(String name) {
        return new Person(name, 0);
    }
    
    // toString方法
    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + "}";
    }
}

// 继承
public class Student extends Person {
    private String studentId;
    
    public Student(String name, int age, String studentId) {
        super(name, age);
        this.studentId = studentId;
    }
    
    // 方法重写
    @Override
    public String toString() {
        return super.toString() + ", studentId=" + studentId;
    }
}

// 接口
public interface Logger {
    void log(String message);
    default void info(String message) {
        System.out.println("[INFO] " + message);
    }
}

// 抽象类
public abstract class Animal {
    public abstract void makeSound();
    public void sleep() {
        System.out.println("Zzz");
    }
}

// 枚举
public enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}

// 记录类 (Java 16+)
public record Point(int x, int y) {}

// 对象操作
Person p = new Person("Alice", 25);
Student s = new Student("Bob", 20, "S12345");
Object obj = new Object();
obj.hashCode();
obj.equals(obj);
obj.getClass();
```

## 函数/方法

```java
// 方法定义
public int add(int a, int b) {
    return a + b;
}

// 可变参数
public int sum(int... numbers) {
    int total = 0;
    for (int num : numbers) {
        total += num;
    }
    return total;
}

// 方法重载
public void print(String s) {
    System.out.println(s);
}

public void print(int i) {
    System.out.println(i);
}

// Lambda表达式
Function<Integer, Integer> square = x -> x * x;
Consumer<String> printer = s -> System.out.println(s);
Supplier<Double> random = Math::random;
Predicate<String> isEmpty = String::isEmpty;

// 方法引用
List<String> names = Arrays.asList("Alice", "Bob");
names.forEach(System.out::println);
```

## 控制结构

```java
// if-else
if (a > b) {
    System.out.println("a is greater");
} else if (a == b) {
    System.out.println("a equals b");
} else {
    System.out.println("a is smaller");
}

// switch (Java 14+)
String result = switch (day) {
    case "M", "W", "F" -> "MWF";
    case "T", "TH", "S" -> "TTS";
    default -> {
        if (day.isEmpty()) yield "Please insert a valid day";
        else yield "Looks like a Sunday";
    }
};

// for循环
for (int i = 0; i < 10; i++) {
    System.out.println(i);
}

// 增强for循环
for (String name : names) {
    System.out.println(name);
}

// while/do-while
while (i < 10) {
    System.out.println(i++);
}

do {
    System.out.println(i++);
} while (i < 10);

// 异常处理
try {
    // 可能抛出异常的代码
    int num = Integer.parseInt("123a");
} catch (NumberFormatException e) {
    System.out.println("Invalid number format");
} catch (Exception e) {
    System.out.println("General exception");
} finally {
    System.out.println("Always executed");
}

// 抛出异常
if (age < 0) {
    throw new IllegalArgumentException("Age cannot be negative");
}

// try-with-resources
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    String line;
    while ((line = br.readLine()) != null) {
        System.out.println(line);
    }
}
```

## 常用类与方法

### 集合框架

```java
// List
List<String> list = new ArrayList<>();
list.add("a"); list.get(0); list.size();
List<String> immutable = List.of("a", "b", "c");

// Set
Set<Integer> set = new HashSet<>();
set.add(1); set.contains(1); set.remove(1);

// Map
Map<String, Integer> map = new HashMap<>();
map.put("a", 1); map.get("a"); map.containsKey("a");

// 工具类
Collections.sort(list);
Collections.reverse(list);
Collections.shuffle(list);
Collections.max(list);
Collections.min(list);
```

### 日期时间 (Java 8+)

```java
// LocalDate/LocalTime/LocalDateTime
LocalDate today = LocalDate.now();
LocalDate date = LocalDate.of(2023, 1, 1);
date.plusDays(1).minusMonths(1).getDayOfWeek();

// Instant/Duration/Period
Instant now = Instant.now();
Duration.between(start, end).toHours();
Period.between(startDate, endDate).getMonths();

// DateTimeFormatter
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
String formatted = date.format(formatter);
LocalDate parsed = LocalDate.parse("2023-01-01", formatter);
```

### 文件IO

```java
// 文件操作
Files.readAllLines(Paths.get("file.txt"));
Files.write(Paths.get("file.txt"), content.getBytes());
Files.copy(source, target);
Files.delete(path);
Files.exists(path);

// 流式读写
try (BufferedReader reader = Files.newBufferedReader(path)) {
    reader.lines().forEach(System.out::println);
}

try (BufferedWriter writer = Files.newBufferedWriter(path)) {
    writer.write("Hello World");
}
```

### 多线程

```java
// 线程创建
Thread thread = new Thread(() -> System.out.println("Running"));
thread.start();

// ExecutorService
ExecutorService executor = Executors.newFixedThreadPool(4);
executor.submit(() -> System.out.println("Task"));
executor.shutdown();

// 同步
synchronized (lockObject) {
    // 临界区代码
}

// 并发集合
ConcurrentHashMap<String, Integer> concurrentMap = new ConcurrentHashMap<>();
CopyOnWriteArrayList<String> threadSafeList = new CopyOnWriteArrayList<>();
```

### 其他常用类

```java
// Math
Math.abs(-5), Math.sqrt(16), Math.pow(2, 3)
Math.max(1, 2), Math.min(1, 2), Math.random()

// System
System.currentTimeMillis(), System.nanoTime()
System.getenv(), System.getProperty("user.dir")
System.out.println(), System.err.println()

// Objects
Objects.equals(a, b), Objects.hashCode(obj)
Objects.requireNonNull(obj), Objects.toString(obj)

// Optional
Optional<String> opt = Optional.ofNullable(null);
opt.orElse("default"), opt.orElseGet(() -> "default")
opt.ifPresent(System.out::println)
```

## 注解

```java
// 内置注解
@Override, @Deprecated, @SuppressWarnings
@FunctionalInterface, @SafeVarargs

// 元注解
@Target(ElementType.METHOD), @Retention(RetentionPolicy.RUNTIME)
@Documented, @Inherited, @Repeatable

// 自定义注解
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Loggable {
    String level() default "INFO";
}
```

## 泛型

```java
// 泛型类
public class Box<T> {
    private T content;
    public void set(T content) { this.content = content; }
    public T get() { return content; }
}

// 泛型方法
public static <T> T getFirst(List<T> list) {
    return list.get(0);
}

// 通配符
List<?> unknownList;
List<? extends Number> numbers;
List<? super Integer> integers;
```

## 反射

```java
Class<?> clazz = Class.forName("java.lang.String");
Field[] fields = clazz.getDeclaredFields();
Method[] methods = clazz.getMethods();
Constructor<?>[] constructors = clazz.getConstructors();

// 创建实例
Object instance = clazz.getConstructor().newInstance();

// 调用方法
Method method = clazz.getMethod("length");
int length = (int) method.invoke("hello");
```