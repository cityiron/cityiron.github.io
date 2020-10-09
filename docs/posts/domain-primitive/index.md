# 领域驱动中Domain_Primitive的简单使用


本文介绍一个实际项目中可以使用的概念：Domain Primitive。

<!--more-->

## 一、前言

领域驱动这个概念也听到很多次了，书上看过的已经忘记，道听途说的不敢苟同（自己也分不清真假，听闻也没一个真正”成功“的案例），这里我就说一句我的理解：深入挖掘业务内核，找到问题的根本，把复杂问题分为多个领域（领域内聚）去解决业务实际的问题，而不是通过技术角度随着岁月留下一堆跳跃的代码。

## 二、本文主角 Domain Primitive

### 2.1 Domain Primitive 的定义

{{< admonition type=note title="概念" >}}
1. 不从任何其他事物发展而来
2. 初级的形成或生长的早期阶段
{{< /admonition >}}

让我们重新来定义一下 Domain Primitive ：Domain Primitive 是一个在特定领域里，拥有精准定义的、可自我验证的、拥有行为的 Value Object 。

• DP是一个传统意义上的Value Object，拥有Immutable的特性
• DP是一个完整的概念整体，拥有精准定义
• DP使用业务域中的原生语言
• DP可以是业务域的最小组成部分、也可以构建复杂组合

> 注意：Domain Primitive的概念和命名来自于Dan Bergh Johnsson & Daniel Deogun的书 Secure by Design。

### 2.2 使用 Domain Primitive 的三原则

• 让隐性的概念**显性**化
• 让隐性的上下文**显性**化
• 封装多对象行为

### 2.3 Domain Primitive 和 DDD 里 Value Object 的区别

在 DDD 中， Value Object 这个概念其实已经存在：

• 在 Evans 的 DDD 蓝皮书中，Value Object 更多的是一个非 Entity 的值对象
• 在Vernon的IDDD红皮书中，作者更多的关注了Value Object的Immutability、Equals方法、Factory方法等

Domain Primitive 是 Value Object 的进阶版，在原始 VO 的基础上要求每个 DP 拥有概念的整体，而不仅仅是值对象。在 VO 的 Immutable 基础上增加了 Validity 和行为。当然同样的要求无副作用（side-effect free）。

▍Domain Primitive 和 Data Transfer Object (DTO) 的区别

在日常开发中经常会碰到的另一个数据结构是 DTO ，比如方法的入参和出参。DP 和 DTO 的区别如下：

|            | DTO                                      | DP                   |
| ---------- | ---------------------------------------- | -------------------- |
| 功能       | 数据传输<br />属于技术细节               | 业务领域中的概念     |
| 数据的关联 | 只是一堆数据放在一起<br />不一定有关联度 | 数据之间的高相关新   |
| 行为       | 无行为                                   | 丰富的行为和业务逻辑 |

### 2.4 什么情况下应该用 Domain Primitive

常见的 DP 的使用场景包括：

• 有格式限制的 String：比如Name，PhoneNumber，OrderNumber，ZipCode，Address等
• 有限制的Integer：比如OrderId（>0），Percentage（0-100%），Quantity（>=0）等
• 可枚举的 int ：比如 Status（一般不用Enum因为反序列化问题）
• Double 或 BigDecimal：一般用到的 Double 或 BigDecimal 都是有业务含义的，比如 Temperature、Money、Amount、ExchangeRate、Rating 等
• 复杂的数据结构：比如 Map> 等，尽量能把 Map 的所有操作包装掉，仅暴露必要行为

## 三、项目的的尝试

举例子：

DP对象

```java
@Getter
@JSONType(deserializer = NameDeserializer.class, serializer = NameSerializer.class)
public class Name implements Serializable {
    private static final long serialVersionUID = 7482387369308807214L;

    private final String name;

    public Name(final String name) {

        if (StringUtils.isBlank(name)) {
            throw new ValidationException("名称不能为空!!!");
        }

        if (!isValid(name)) {
            throw new ValidationException("仅支持1-64位大小写字母，数字，中划线和下划线组成，必须字母开头!!!");
        }
        this.name = name;
    }

    private static boolean isValid(String code) {
        String pattern = "^[a-zA-Z][a-zA-Z0-9_-]{1,64}$";
        return code.matches(pattern);
    }
}
```

领域方法

```java
    // 根据应用和Git信息查询流水线发布记录
    public PipelineDO queryByAppAndGit(final Name appName, final Git git) {
        // 进行逻辑处理
    }
```

说明

- final 修饰字段，不可变性的特点
- 构造方法就直接校验合法性，不需要factory和validutil，更内聚自己的功能
- 因为存在JSON序列化和反序列的情况，这里需要自定义序列化和反序列的方法

效果

- 前端联调企图传随便的内容进来，直接通不过参数校验，就进不到核心方法了
- 参数相对更能直观其意，并且进来的参数值取出来肯定是合法的
- 代码可以写的更少

容易出错的地方

- 如果用Map做一个缓存，Key放String，而实际对应的是Code对象，那么查询的时候要用code.genCodeString()，一开始容易写code忘记转换，导致查询不到结果
- JSON转换在上面已经提到了需要自己定义扩展，会增加一定的重复代码量（我们暂时用的是每个DP类型两个转换类，可以合并到一个方法，根据类型判断，但是语义不够清晰

还没做好的地方

- 如果通过jar包提供给第三方接口调用也使用这个DP类，那么接口调用方传参不合法的时候就会报错

## 四、参考
[阿里云领域驱动DP文章](https://developer.aliyun.com/article/716908?spm=a2c6h.12873581.0.0.41f165c3iAKf34&groupCode=taobaotech)


<!-- ## 工作中的日常吐槽

> 和本文无关

项目接口定义的参数是page和length，进行联调的过程中，前端同学钉钉发来问page是否是0，10，20？，这让我楞了一下，怎么页码无中生有起飞了，于是找前端同学沟通了解到page对他们来讲起始位置（他们之前一直是这样做的，说让后端方便点），我心里想怎么把limit给搞出来了，涨见识了。

因为前端告知比较麻烦，乐于助人的我就上线了，我第一时间考虑用构造函数接受这两个参数处理，但是我之前把分页对象的属性抽象到了QueryParams，让所有的BaseEntity都继承了，我用了实体对象就没法通过构造函数，委屈求全临时加了个属性去操作，如下：

```java
    /**
     * 起始位置，为前端特意定制
     */
    @JSONField(serialize = false)
    protected Integer startIndex = 0;

    /**
     * 必须用在 setPageSize 之后
     * 设置分页起始位置
     *
     * @param startIndex 分页起始位置
     */
    public void setStartIndex(Integer startIndex) {
        this.startIndex = startIndex;
        pageNumber();
    }

    private void pageNumber() {
        // 0的话不用处理
        if (startIndex <= 0) {
            startIndex = 0;
            return;
        }

        if (startIndex % pageSize != 0) {
            throw new RuntimeException("非法的分页起始位置");
        }

        // 计算页码
        this.pageNumber = startIndex / pageSize + 1;
    }
``` -->

