## @Getter

### 定义

```Java
@Target({ElementType.FIELD, ElementType.TYPE})
@Retention(RetentionPolicy.SOURCE)
public @interface Getter {
    AccessLevel value() default AccessLevel.PUBLIC;

    AnyAnnotation[] onMethod() default {};

    boolean lazy() default false;
}
```

### 注

* **`AccessLevel`**： 由枚举类 `AccessLevel` 提供。可选 PUBLIC 、PROTECTED 、 PRIVATE 。表示该字段或类中字段的 get 访问权限。
> **覆盖规则**： 字段上的 @Getter 注解优先级高于类上的。并且类上打该注解，相当于对类内全部字段打该注解。

* **`onMethod`**： 对于 @Getter 注解，编译后相当于一个 `getVariable()` 方法方法。既然是方法，就可以对其打上一些注解。打上的注解就可以用这个注解属性进行传递。
```Java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Log {
    String value() default "getter方法";
}

public class User {
    // 为生成的 getName() 方法添加 @Log 和 @Override 注解
    @Getter(onMethod = {@Log("获取姓名"), @Override})
    private String name;
}

// 实际编译后的代码：getVariable() 方法上面带上了指定的注解
public class User {
    private String name;

    @Log("获取姓名")
    @Override
    public String getName() {
        return name;
    }
}
```

* **`lazy`**： 控制字段的懒加载，默认关闭。开启后第一次调用 getter 才初始化字段。目前不知道有什么用。

## @Setter

### 定义

```Java
@Target({ElementType.FIELD, ElementType.TYPE})
@Retention(RetentionPolicy.SOURCE)
public @interface Setter {
    AccessLevel value() default AccessLevel.PUBLIC;

    AnyAnnotation[] onMethod() default {};

    AnyAnnotation[] onParam() default {};
}
```

### 注

value 、 onMethod 与 @Getter 一致。

* **`onParam`**： 理解和 onMethod 一样。 打上 @Setter 注解的字段，在编译时会生成一个 `setVariable([Type] value)` 方法。它和 @Getter 不一样，生成的方法是有参数的。而注解的作用域恰好包括形参（ ElementType.PARAMETER ），比如 @NonNull 。所以需要这个注解属性来为生成的 Set 方法中的参数传递注解。

```Java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface Test {
    String msg() default "参数不能为空";
}

public class User {
    @Setter(onParam = @Test)
    private String name;
}

// 实际编译后：
public class User {
    private String name;
    private Integer age;

    // set 方法的形参上附加了 @Test
    public void setName(@Test String name) {
        this.name = name;
    }
}
```

## @Accessors

该注解可被用于配置 @Getter 与 @Setter 的生成。

### 定义
```Java
@Target({ElementType.TYPE, ElementType.FIELD})
@Retention(RetentionPolicy.SOURCE)
public @interface Accessors {
    boolean fluent() default false;

    boolean chain() default false;

    boolean makeFinal() default false;

    String[] prefix() default {};
}
```

### 注

* **`boolean chain() default false`**： **强烈建议开启**，可以使得 set 方法可链式调用。

* **`boolean fluent() default false`**： 生成无前缀的极简 get/set。比如 setAge() ，变成 age() 。

* 其他不用管。

## 构造方法相关注解

### @AllArgsConstructor

#### 定义

```Java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.SOURCE)
public @interface AllArgsConstructor {
    String staticName() default "";

    AnyAnnotation[] onConstructor() default {};

    AccessLevel access() default AccessLevel.PUBLIC;
}
```

#### 注

* **作用**： 为 class 中的全部参数生成一个构造方法。

* `onConstructor` 用于在编译后的构造方法上添加注解。格式和前面的 `onMethod` 、 `onParam` 一致。

* `staticName` 用于额外生成一个静态的工厂方法。传入的字符串作为静态方法的方法名。最经典的例子就是 `List.of(...)` 。

### @NoArgsConstructor

#### 定义

```Java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.SOURCE)
public @interface NoArgsConstructor {
    String staticName() default "";

    AnyAnnotation[] onConstructor() default {};

    AccessLevel access() default AccessLevel.PUBLIC;

    boolean force() default false;
}
```

#### 注

* **作用**：生成一个无参构造方法。
> 这个点经常忘。一个 class 可以有有参构造方法，也可以有无参构造方法。这个注解就是用于生成一个无参的构造方法。因为如果写了构造方法，会覆盖原本的默认无参构造方法。在框架中，会导致一些问题。所以需要补上一个无参构造方法。

* `force` : `force = true` 时，强制为类生成无参构造方法，即使类中有**未显式初始化**的 final 字段。此时注解处理器会在无参构造中，为这些 final 字段赋上 Java 的默认值，强行满足 final 的初始化规则。

### @RequiredArgsConstructor

#### 定义

```Java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.SOURCE)
public @interface RequiredArgsConstructor {
    String staticName() default "";

    AnyAnnotation[] onConstructor() default {};

    AccessLevel access() default AccessLevel.PUBLIC;
}
```

#### 注

* **作用**：**生成仅包含类中必需字段的构造方法**，入参顺序与字段声明顺序一致。
> 必须字段：指未显式初始化的 final 字段、标注了 @NonNull 注解的字段。这两类字段 Java 语法要求对象创建时必须赋值，否则编译报错。

## @ToString

用于重写 `toString()` 方法。
### 定义

```Java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.SOURCE)
public @interface ToString {
    boolean includeFieldNames() default true;

    String[] exclude() default {};

    String[] of() default {};

    boolean callSuper() default false;

    boolean onlyExplicitlyIncluded() default false;

    @Target({ElementType.FIELD})
    @Retention(RetentionPolicy.SOURCE)
    public @interface Exclude {
    }

    @Target({ElementType.FIELD, ElementType.METHOD})
    @Retention(RetentionPolicy.SOURCE)
    public @interface Include {
        int rank() default 0;

        String name() default "";
    }
}
```

### 注

* **`boolean includeFieldNames() default true`**： 打印时是否显示字段名

* **`String[] exclude() default {}`**：指定排除的字段（黑名单）

* **`String[] of() default {}`**：指定仅包含的字段（白名单）
> 不建议与 exclude 同时使用。

* **`boolean callSuper() default false`**：拼接打印父类的字 toString 信息。**要求父类重写 `toString()` 方法**。
> 继承场景下建议开启。

* **`boolean onlyExplicitlyIncluded() default false`**：全局控制字段的默认行为，是 @ToString.Include 注解的开关属性。当设为 true 时，只有标注了` @ToString.Include` 的字段 / 方法，才会参与 toString 拼接。**更推荐这种做法！**

### 内部嵌套注解

#### @ToString.Exclude

* **作用**：仅字段级排除打印。
> 注解直接标在字段上，代码可读性更高。还有一个好处是，修改字段名称时，如果用的是 exclude 注解属性，**要同步修改里面的 String 值**。但是用内部注解不需要。

#### @ToString.Include

#####  作用范围
* 类的**非静态字段** 、 **无参、有返回值的实例方法** 。字段是显然的，下面介绍打在实例方法上。

* **作用在无参方法上**：将方法的返回值拼接到 toString() 中。

##### 属性
  
* **`String name() default ""`**：自定义打印的别名。

* **`int rank() default 0`**：自定义排序权重。权重越大，在 toString () 中出现的位置越靠前。若多个内容的rank值相同，则按代码中的声明顺序拼接。

### 举个例子

```Java
@ToString(onlyExplicitlyIncluded = true,  callSuper = true)
public class User {
    @ToString.Include(name = "用户ID", rank = 20)
    private Long id;
    
    @ToString.Include(name = "姓名", rank = 15)
    private String name;
    
    @ToString.Include(rank = 10)
    private Integer age;

    // 未标注，默认排除
    private String pwd; 
    private String address; 
    private LocalDate birthday; // 日期字段，通过方法格式化
    
    @ToString.Include(name = "生日", rank = 8)
    public String formatBirthday() {
        if (birthday == null) {
            return "未知生日";
        }
        return birthday.format(DateTimeFormatter.ofPattern("yyyy年MM月dd日"));
    }
}
```

## EqualsAndHashCode

用于重写 `equals()` 方法和 `hashCode()` 方法

### 定义

```Java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.SOURCE)
public @interface EqualsAndHashCode {
    String[] exclude() default {};

    String[] of() default {};

    boolean onlyExplicitlyIncluded() default false;

    @Target({ElementType.FIELD})
    @Retention(RetentionPolicy.SOURCE)
    public @interface Exclude {
    }

    @Target({ElementType.FIELD, ElementType.METHOD})
    @Retention(RetentionPolicy.SOURCE)
    public @interface Include {
        String replaces() default "";

        int rank() default 0;
    }
}
```

### 注

有些属性之前已经介绍过了，不过多赘述。

* **`boolean onlyExplicitlyIncluded() default false`**：全局包含开关，设为 true 时，仅标注 `@EqualsAndHashCode.Include` 的字段 / 方法才参与比较。

* **`boolean callSuper() default false`**： 子类生成的 equals() 会先调用父类的 equals() 方法（比较父类字段），父类字段相等后，再比较子类字段；hashCode() 会结合父类的 hashCode() 结果 + 子类字段的哈希值一起计算，保证父类 + 子类字段完全相同的对象，才会判为相等、哈希值相同。**要求父类必须也重写了 `equals()` 和 `hashCode()`** 。
> 继承场景下建议开启。

### 内部嵌套注解

#### @EqualsAndHashCode.Exclude

* **作用**：仅字段级排除比较。优势同 @ToString 的。

#### @EqualsAndHashCode.Include

#####  作用范围
* 类的**非静态字段** 、 **无参、有返回值的实例方法** 。字段是显然的，下面介绍打在实例方法上。

* **作用在无参方法上**：将方法的返回值用于比较。

##### 属性
  
* **`String replaces() default ""`**：一般都用于方法上。让方法的返回值，去替代replace中的字段，从而实现比较。replace中的String值就是 class 中某个非静态字段的名称。
```Java
@Data
@Accessors(chain = true)
public class User {
    private Integer age;

    @EqualsAndHashCode.Include(replaces = "age")
    public Integer getDealAge() {
        return this.age == null ? 0 : this.age; 
    }
}

public class Main {
    public static void main(String[] args) {
        User u1 = new User().setAge(null);
        User u2 = new User().setAge(-5);

        System.out.println(u1.equals(u2));
        // 结果为 true
    }
}
```
> **在比较 age 这个字段时，统一用 getDealAge() 的返回值。不再使用原本字段的值**。所以最后 u1 u2 的 age 值都用方法的返回值，即为 0 。所以相等了。

* **`int rank() default 0`**：不用管，基本上用不到。

## @Data

@Data 仅适用于**可变实体类**，不适用于**工具类**（ @UtilityClass ）与**记录类**（ @Value ）。

### 组成

组合了 5 个基础注解。 @Data 等价于类上同时标注： **@Getter + @Setter + @ToString + @EqualsAndHashCode + @RequiredArgsConstructor**

> * 仅处理**非静态实例字段**
> * 生成默认的 @ToString() & @EqualsAndHashCode : superCall = false；
> * 仅生成 @RequiredArgsConstructor 。**强烈建议补全有参与无参构造方法**。

### 模版
```Java
@Data
@NoArgsConstructor  // 补充无参构造
@AllArgsConstructor // 补充全参构造

// 继承父类时，需额外加
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
public class User extends BaseEntity {
    // 子类字段
}
```

## @Value

专为不可变类设计，是**不可变版的 @Data** 打上该注解相当于将 class 变成 **[记录类](JavaSE.md#java-常见类--记录类)**

### 组成

组合了 5 个基础注解。 @Value 等价于类上同时标注： **@Getter + @FieldDefaults(makeFinal=true, level=PRIVATE) + @AllArgsConstructor + @ToString + @EqualsAndHashCode** 。打上该注解后可以使用**紧凑构造器**。
> 你的字段权限是 private ，但是你 public 权限的 get 方法又弥补了 这一点。

### 举个例子
```Java
@Value
public class User {
    Long id;
    String name;
    Integer age;

    public User {
        if (age == null || age < 0) {
            throw new IllegalArgumentException("年龄不能为null/负数");
        }
        if (name == null) name = "未知";
    }
}

public class Main {
    public static void main(String[] args) {
        User user = new User(1L, "张三", 24);

        Long id = user.id(); // 取值：id()，而非getId()
        String name = user.name();
    }
}
```

## @FieldDefaults

### 定义
```Java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.SOURCE)
public @interface FieldDefaults {
    AccessLevel level() default AccessLevel.NONE;

    boolean makeFinal() default false;
}
```

可以很方便地通过修改 level 的值来快速为 class 中字段生成修饰符。 level 的值由枚举类 `AccessLevel` 提供。

## @UtilityClass

打上直接变工具类，全部方法变静态。



