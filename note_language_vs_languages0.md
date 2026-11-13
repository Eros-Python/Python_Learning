# 笔记：`language` vs `languages[0]` 的区别

> 来自练习 6.4.2 改进 favorite_languages 程序时的讨论

## 两种写法对比

假设 `languages = ['python', 'ruby']`：

| 写法 | 含义 | 拿到的值 |
|---|---|---|
| `for language in languages` 里的 `language` | 遍历列表，循环变量依次取每个元素 | `'python'` → `'ruby'` |
| `languages[0]` | 用下标直接取列表的第 1 个元素 | `'python'` |

**两者拿到的都是字符串**，都能调 `.title()` 等字符串方法。区别只在于"怎么拿到"的。

---

## 为什么 else 分支里必须用 `languages[0]`，不能用 `language`？

### ❌ 错误写法

```python
for name, languages in favorite_languages.items():
    if len(languages) > 1:
        for language in languages:       # ← 只有这里才给 language 赋值
            print(...)
    else:
        print(f"... {language.title()}") # ← 这里的 language 可能是残留值！
```

### ⚠️ Python 的"坑"：for 循环变量不会自动销毁

Python 的 `for` 循环变量作用域是**整个函数/模块**，不是只在循环内部。循环结束后变量会保留**最后一次的值**。

看上面代码的执行流程：

| 步骤 | 处理谁 | 走哪个分支 | `language` 变量值 |
|---|---|---|---|
| 1 | Jen | if 分支（进 for 循环） | 循环跑完残留 `'ruby'` |
| 2 | Sarah | **else 分支（不进 for 循环）** | 还是上一步残留的 `'ruby'` 💥 |
| 3 | Edward | if 分支（进 for 循环） | 循环跑完残留 `'go'` |
| 4 | Phil | if 分支（进 for 循环） | 循环跑完残留 `'haskell'` |

Sarah 走了 else 分支，**根本没进 for 循环**，所以 `language` 根本没被重新赋值，用的是 Jen 循环遗留的 `'ruby'`，导致输出错误：

```
Sarah's favorite language is Ruby   ← 错！应该是 C
```

### ✅ 正确写法

```python
for name, languages in favorite_languages.items():
    if len(languages) > 1:
        for language in languages:
            print(f"\t{language.title()}")
    else:
        print(f"... {languages[0].title()}")  # ← 直接取列表第一个元素
```

`languages[0]` 从列表本身取值，**不依赖循环变量**，无论走哪个分支都能拿到正确的值。

---

## 什么时候用哪种？

| 场景 | 用哪种 | 原因 |
|---|---|---|
| 需要逐个处理每个元素 | `for language in languages` | 循环变量自动依次取每个元素 |
| 只需要某一个元素 | `languages[i]` | 直接按下标取，不依赖循环状态 |
| 在 if/else 等分支里用值 | `languages[i]` | 避免循环变量残留的坑 |
