## 0x00 起因
起因是在Android上用了一段时间的AndPods觉得不太好用之后，换到了另一个Play商店推荐的App。动画、连接和电量提示都用的很满意，就是每次连接的弹窗和APP里面都有广告，就想着改成Pro版把广告去掉。

---

## 0x01 解包分析
首先把apk拿出来解包，开始看代码。

![混淆过的classes](https://github.com/MaYiFei1995/MaYiFei1995.github.io/blob/master/img/2020-12-16-1.png)

代码没有加固，用了混淆，搜索关键字之后发现，只做了一个签名校验，pro版本通过一个常量来判断并通过SP明文存储。
root过的话直接通过adb修改SP的内容就好，由于手机没有root，只好通过改代码的方式来实现破解。

![pro版判定](https://github.com/MaYiFei1995/MaYiFei1995.github.io/blob/master/img/2020-12-16-2.png)

![签名校验](https://github.com/MaYiFei1995/MaYiFei1995.github.io/blob/master/img/2020-12-16-3.png)

---

## 0x02  修改Pro版
通过搜索代码确定，pro版的判定只通过MenuItem::isPro，直接把值写死，setter方法return就好。
- 修改MenuItem.smali的constructor，在return-void前为isPro初始化
```diff
# direct methods
.method public constructor <init>()V
    .locals 1

    invoke-direct {p0}, Ljava/lang/Object;-><init>()V

    new-instance v0, Ljava/util/ArrayList;

    invoke-direct {v0}, Ljava/util/ArrayList;-><init>()V

    iput-object v0, p0, Lcom/pryshedko/materialpods/model/MenuItem;->possibleVariants:Ljava/util/ArrayList;

    sget-object v0, Lcom/pryshedko/materialpods/model/MenuItem$PROPERTY_TYPE;->TEXT:Lcom/pryshedko/materialpods/model/MenuItem$PROPERTY_TYPE;

    iput-object v0, p0, Lcom/pryshedko/materialpods/model/MenuItem;->type:Lcom/pryshedko/materialpods/model/MenuItem$PROPERTY_TYPE;

    const/4 v0, 0x1

    iput-boolean v0, p0, Lcom/pryshedko/materialpods/model/MenuItem;->isVisible:Z

    const/high16 v0, 0x43110000    # 145.0f

    iput v0, p0, Lcom/pryshedko/materialpods/model/MenuItem;->maxValue:F

    sget-object v0, Lb/a/a/a/a/b/a$b;->d:Lb/a/a/a/a/b/a$b;

    iput-object v0, p0, Lcom/pryshedko/materialpods/model/MenuItem;->unit:Lb/a/a/a/a/b/a$b;

    const/high16 v0, 0x40800000    # 4.0f

    iput v0, p0, Lcom/pryshedko/materialpods/model/MenuItem;->defaultFloatValue:F

    
+   const/4 v0, 0x1
+   iput-boolean v0, p0, Lcom/pryshedko/materialpods/model/MenuItem;->isPro:Z
    
    return-void
.end method
```
- 修改MenuItem.smali的setPro方法，直接删除iput-boolean
```diff
.method public final setPro(Z)V
    .locals 0

-   iput-boolean p1, p0, Lcom/pryshedko/materialpods/model/MenuItem;->isPro:Z

    return-void
.end method
```

---

## 0x03 绕过签名校验
通过搜索代码确定，签名校验的接口为`b.g.b.b.b.g.b()`，直接修改方法的返回值
```diff
.method public static b(Landroid/content/pm/PackageInfo;Z)Z
    .locals 3

-    const/4 v0, 0x0

-   if-eqz p0, :cond_1

-   iget-object v1, p0, Landroid/content/pm/PackageInfo;->signatures:[Landroid/content/pm/Signature;

-   if-eqz v1, :cond_1

-   const/4 v1, 0x1

-   if-eqz p1, :cond_0

-   sget-object p1, Lb/g/b/b/b/t;->a:[Lb/g/b/b/b/q;

-   invoke-static {p0, p1}, Lb/g/b/b/b/g;->a(Landroid/content/pm/PackageInfo;[Lb/g/b/b/b/q;)Lb/g/b/b/b/q;

-   move-result-object p0

-   goto :goto_0

-   :cond_0
-   new-array p1, v1, [Lb/g/b/b/b/q;

-   sget-object v2, Lb/g/b/b/b/t;->a:[Lb/g/b/b/b/q;

-   aget-object v2, v2, v0

-   aput-object v2, p1, v0

-   invoke-static {p0, p1}, Lb/g/b/b/b/g;->a(Landroid/content/pm/PackageInfo;[Lb/g/b/b/b/q;)Lb/g/b/b/b/q;

-   move-result-object p0

-   :goto_0
-   if-eqz p0, :cond_1

-   return v1

-   :cond_1
-   return v0

+ const/4 v0, 0x1

+ return v0

.end method
```

---

## 0x04 重打包&签名
由于我是用apktool2.4.0和旧的framework-res进行解包的，AndroidManifest中的`foregroundServiceType="location"`没有被正确解析，导致重打包时报了foregroundServiceType的错误，升级到apktool2.5.0并安装最新的framework-res.apk即可解决。

---

## 0x05 总结
这个App没有做过多的防护，重打包的过程也只是因为工具版本太低出现了一点小问题，修改起来没有什么难度，分析签名校验和Pro鉴权也算顺利，没遇到什么坑。