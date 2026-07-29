# 封装 (Encapsulation)

封装就是把数据和操作数据的方法放在一起，并且隐藏内部实现细节，只对外暴露必要接口。

比如把成员变量设置成private，只通过public的方法来访问和修改状态。其目的是保护类的不变量 (invariant)。

---

# 封装的核心是 invariant

一个类最重要的不只是“有哪些成员变量”，而是它维护什么 invariant：
- `std::vector` &rarr; pointer / size / capacity 之间必须保持一致
- `std::unique_ptr` &rarr; 同一时刻只有一个拥有者
- `MutexGuard` &rarr; 对象活着时锁必须处于持有状态
- `Fraction` &rarr; 分母不能为 0，并且可能要保持约分后的规范形式

这意味着：

- 构造函数的职责是建立 invariant
- 成员函数的职责是维护 invariant
- 析构函数的职责是安全地结束生命周期
- API 设计的职责是防止用户轻易破坏 invariant

---

# 封装的本质是保护 invariant

封装并不是把所有的东西都放在private，而是只暴露那些不会破坏类 invariant 的操作。

- 用户只能通过受控方式修改状态
- 类可以集中检查规则
- invariant 不容易被外部代码破坏

所以**private只是手段，保护invariant才是目的**。