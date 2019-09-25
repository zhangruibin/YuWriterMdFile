

最近几天再过一遍阿里啊Java开发规范，打算将里面感觉理解模糊的小知识点做一个记录。
然后看到了@Deprecated注解，那这个注解有啥用呢？
 

字面含义为：
| deprecated |
| 英[ˈdeprəkeɪtɪd][](javascript:;)[](javascript:;) | 美[ˈdeprəkeɪtɪd][](javascript:;)[](javascript:;) |

| v. | 对…表示极不赞成; 强烈反对; |

| [词典] | **deprecate**的过去分词和过去式; |

那在Java代码中怎么使用有什么用途呢？

可以这么理解：
如果某类或某方法加上该注解之后，表示此方法或类不再建议使用，调用时也会出现删除线，不是不能用，而是不推荐使用。因为还有更好的方法可以调用。


看下源码：
```
/*
 * Copyright (c) 2003, 2013, Oracle and/or its affiliates. All rights reserved. * ORACLE PROPRIETARY/CONFIDENTIAL. Use is subject to license terms. * * * * * * * * * * * * * * * * * * * * */   package java.lang;

import java.lang.annotation.*;
import static java.lang.annotation.ElementType.*;

/**
 * A program element annotated &#64;Deprecated is one that programmers
 * are discouraged from using, typically because it is dangerous, * or because a better alternative exists.  Compilers warn when a * deprecated program element is used or overridden in non-deprecated code. * * @author Neal Gafter
 * @since 1.5
 * @jls 9.6.3.6 @Deprecated
 */
@Documented @Retention(RetentionPolicy.RUNTIME)
@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})
public @interface Deprecated {
}
```

原文注解翻译过来如下：

注释为@deprecated的程序元素是不鼓励程序员使用的元素，通常是因为它是危险的，或者是因为存在更好的替代方案。编译器在非弃用代码中使用或重写预先指定的程序元素时发出警告。


那在代码中使用效果是怎样的呢？
如下图所示：

![](https://www.zhangruibin.com/upload/2019/09/24fo7ohc0ujs3q9mp9bdvbv0l0.png)


