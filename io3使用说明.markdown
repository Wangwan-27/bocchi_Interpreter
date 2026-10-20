# io3 扩展方法文档
ver1.1 2026.10.19

> 文件来源：`set void = void;.txt`
> 语言：Bocchi 4.1
> 本文件定义了一套通用工具函数（数学、列表、字符串、文件、判空），供其他 Bocchi 脚本 `use "io3"` 后调用。

---
## 1.1新增:
```bocchi
sort(list, reverse = false)
//对列表 `list` 进行排序。默认按升序排序，`reverse = true` 时按降序排序。

reverse2(list)
//对列表 `list` 进行反转。
```
## 目录

- [全局常量](#全局常量)
- [数学类](#数学类)
- [列表类](#列表类)
- [字符串类](#字符串类)
- [文件类](#文件类)
- [工具类](#工具类)
- [分类速查表](#分类速查表)
- [依赖关系](#依赖关系)
- [使用示例](#使用示例)

---

## 全局常量

| 名称 | 值 | 用途 |
|---|---|---|
| `void` | `"void"` | 表示"空值"的字符串哨兵 |
| `nil` | `"<nil>"` | 另一个空值标记（本文件定义但未使用） |

---

## 数学类

### `min(...nums)`

求所有参数中的最小值。

- **参数**：任意数量数字（可变参数）
- **返回**：最小值；参数为空时返回 `void`
- **示例**：`min(3, 1, 4, 1, 5) → 1`

```bocchi
fn min(...nums) {
    if (nums.len == 0) {
        return void;
    }
    set m = nums[0];
    for v in nums {
        if (v < m) {
            m = v;
        }
    }
    return m;
}
```

---

### `max(...nums)`

求所有参数中的最大值。

- **参数**：任意数量数字（可变参数）
- **返回**：最大值；参数为空时返回 `void`
- **示例**：`max(3, 1, 4, 1, 5) → 5`

```bocchi
fn max(...nums) {
    if (nums.len == 0) {
        return void;
    }
    set m = nums[0];
    for v in nums {
        if (v > m) {
            m = v;
        }
    }
    return m;
}
```

---

### `sum(...nums)`

求和。

- **参数**：任意数量数字（可变参数）
- **返回**：数字，初始为 `0`
- **示例**：`sum(1, 2, 3) → 6`

```bocchi
fn sum(...nums) {
    set total = 0;
    for v in nums {
        total = total + v;
    }
    return total;
}
```

---

### `avg(...nums)`

求平均值。

- **参数**：任意数量数字（可变参数）
- **返回**：平均数；无参数时返回 `void`
- **示例**：`avg(1, 3, 4324, 2342) → 1667.5`

```bocchi
fn avg(...nums) {
    set total = 0;
    set count = 0;
    for v in nums {
        total = total + v;
        count = count + 1;
    }
    if (count == 0) {
        return void;
    }
    return total / count;
}
```

---

### `abs(x)`

求绝对值。

- **参数**：`x` 数字
- **返回**：非负数
- **示例**：`abs(-10) → 10`

```bocchi
fn abs(x) {
    if (x < 0) {
        return 0 - x;
    }
    return x;
}
```

---

### `formax(a, b)`

截取数字字符串前 `b` 位并转为浮点数。

- **参数**：
  - `a`：数字字符串，如 `"3.1415926"`
  - `b`：截取位数
- **返回**：`float`
- **说明**：
  - `a` 按字符拆成数组 `aa`
  - 若 `aa.len < b`，直接 `a.float`
  - 否则用 `for..in i b+2` 把前 `b+2` 个字符写入 `test.txt`，再 `file.read` 转 `float`
  - **注意**：实现依赖全局中转文件 `test.txt`，且实际多取 2 位（依赖 `float` 解析截断）
- **示例**：`formax("3.1415926", 3) → 3.14`

```bocchi
fn formax(a,b) {
    set aa = a.split("");
    if (aa.len < b) {
        set x = a.float;
    } else {
        set i = 0;
        for..in i b+2 {
            file.append("test.txt", aa[i]);
            i = i + 1;
        }
        set lin = file.read("test.txt");
        set x = lin.float;
    }
    return x;
}
```

---

## 列表类

### `addlist(a, b)(旧版，不建议使用)`合并两个列表。
当前拥有更安全的memory操作，建议使用`memory.append`。

- **参数**：`a`、`b` 两个列表
- **返回**：合并后的新列表
- **说明**：**实现绕远路**——通过 `test.txt` 做序列化中转：
  1. `file.write` 写入 `a.string + "\n" + b.string`
  2. `file.read` 读回
  3. 按行 `split("\n")`
  4. 循环中去掉 `[` `]` 和多余空格，再 `split(" ")`
  5. 最后 `file.delete` 删除中转文件
- **注意**：
  - 循环条件 `i < x.len + 1` 与外部变量 `i` 耦合
  - 循环内 `set a = ...` 反复重新赋值，最终返回最后一次处理结果
  - 含空格或特殊字符的元素容易解析错误
- **示例**：`addlist([1, 2], [3, 4]) → [1, 2, 3, 4]`

```bocchi
set addlist = lambda(a,b) {
    file.write("test.txt", a.string +"\n"+ b.string);
    lin = file.read("test.txt");
    x = lin.split("\n");
    file.delete("test.txt");

    loop (i<x.len+1) {
        i = i + 1;
        set a = x.string.replace("]","");
        set a = a.string.replace("[","");
        set a = a.string.replace("   ","");
        set a = a.string.split(" ");
    }

    return a;
};
```

---

### `del(list1, index)`

删除列表指定下标的元素。

- **参数**：
  - `list1`：列表
  - `index`：要删除的下标
- **返回**：新列表
- **依赖**：`addlist`
- **示例**：`del(["a","b","c","d","e"], 2) → ["a","b","d","e"]`

```bocchi
fn del(list1,index) {
    set l1 = list1[0:index];
    index = index + 1;
    set l2 = list1[index:];
    set l1 = addlist(l1,l2);
    return l1;
}
```

---

### `setlist(listc, index, index2)`

把列表中下标 `index` 的元素替换为 `index2`。

- **参数**：
  - `listc`：列表
  - `index`：要替换的下标
  - `index2`：新值
- **返回**：新列表
- **依赖**：`addlist`
- **示例**：`setlist(["a","b","c"], 1, "X") → ["a","X","c"]`

```bocchi
fn setlist(listc,index,index2) {
    set l1 = listc[0:index];
    index = index + 1;
    set l2 = listc[index:];
    set l1 = addlist(l1,index2);
    set l1 = addlist(l1,l2);
    return l1;
}
```

---

### `isinlist(list1, value)`

判断列表是否包含某个值。

- **参数**：`list1` 列表，`value` 待查值
- **返回**：`bool`
- **示例**：`isinlist([1,2,3,4,5], 3) → true`

```bocchi
fn isinlist(list1, value) {
    for v in list1 {
        if (v == value) {
            return true;
        }
    }
    return false;
}
```

---

### `indexof(list1, value)`

查找元素下标。

- **参数**：`list1` 列表，`value` 待查值
- **返回**：首次出现的下标；找不到返回 `-1`
- **示例**：`indexof([1,2,3,4,5], 3) → 2`

```bocchi
fn indexof(list1, value) {
    set i = 0;
    for v in list1 {
        if (v == value) {
            return i;
        }
        i = i + 1;
    }
    return -1;
}
```

---

### `countof(list1, value)`

统计某值在列表中出现的次数。

- **参数**：`list1` 列表，`value` 待查值
- **返回**：数字
- **示例**：`countof([1,2,3,4,5,3], 3) → 2`

```bocchi
fn countof(list1, value) {
    set c = 0;
    for v in list1 {
        if (v == value) {
            c = c + 1;
        }
    }
    return c;
}
```

---

### `join(list1, sep)`

用分隔符把列表拼成字符串。

- **参数**：`list1` 列表，`sep` 分隔符字符串
- **返回**：字符串
- **说明**：首元素前不加分隔符
- **示例**：`join([1,2,3,4,5], ".") → "1.2.3.4.5"`

```bocchi
fn join(list1, sep) {
    set result = "";
    set first = true;
    for v in list1 {
        if (first) {
            result = result + v.string;
            first = false;
        } else {
            result = result + sep + v.string;
        }
    }
    return result;
}
```

---

### `choice(list1)`

随机取列表中的一个元素。

- **参数**：`list1` 列表
- **返回**：任意元素；空列表返回 `void`
- **示例**：`choice([1,2,3,4,5]) → 随机一个`

```bocchi
fn choice(list1) {
    if (list1.len == 0) {
        return void;
    }
    return list1[rand(0, list1.len - 1)];
}
```

---

## 字符串类

### `trim(s)`

去除字符串中的所有空格。

- **参数**：`s` 字符串
- **返回**：去掉空格的字符串
- **说明**：本语言无原生 `trim`，这里用 `replace(" ", "")` 简化实现，**不处理** `\t`、`\n` 和首尾空白
- **示例**：`trim("  123  ") → "123"`

```bocchi
fn trim(s) {
    set x = s.replace(" ", "").string;
    return x;
}
```

---

### `repeatstring(s, n)`

重复字符串 `n` 次。

- **参数**：`s` 字符串，`n` 次数
- **返回**：字符串
- **示例**：`repeatstring("hello", 3) → "hellohellohello"`

```bocchi
fn repeatstring(s, n) {
    set result = "";
    for..in i n {
        result = result + s;
    }
    return result;
}
```

---

### `padleft(s, pad, total)`

左填充字符串到指定长度。

- **参数**：
  - `s` 原字符串
  - `pad` 填充字符
  - `total` 目标总长度
- **返回**：字符串
- **说明**：用 `split("").len` 判断当前长度
- **示例**：`padleft("5", "0", 3) → "005"`

```bocchi
fn padleft(s, pad, total) {
    set result = s;
    loop (result.split("").len < total) {
        result = pad + result;
    }
    return result;
}
```

---

### `padright(s, pad, total)`

右填充字符串到指定长度。

- **参数**：
  - `s` 原字符串
  - `pad` 填充字符
  - `total` 目标总长度
- **返回**：字符串
- **示例**：`padright("5", "0", 3) → "500"`

```bocchi
fn padright(s, pad, total) {
    set result = s;
    loop (result.split("").len < total) {
        result = result + pad;
    }
    return result;
}
```

---

## 文件类

### `fileadd(file1, txt)`

向文件追加文本。

- **参数**：
  - `file1`：文件路径
  - `txt`：要追加的内容
- **返回**：`""`（空字符串）
- **说明**：
  - 先 `file.read` 读入全局 `file2`
  - 拼接 `txt` 后 `file.write` 回写
  - **会覆盖全局变量 `file2`**
- **示例**：`fileadd("use.txt", "1")` — 把 `"1"` 追加到 `use.txt` 末尾

```bocchi
set fileadd = lambda(file1,txt) {
    file2 = file.read(file1);
    file2 = file2 + txt;
    file.write(file1,file2);
    return "";
};
```

---

## 工具类

### `isVoid(x)`

判断变量是否为空值。

- **参数**：`x` 任意
- **返回**：`bool`
- **说明**：与全局常量 `void`（字符串 `"void"`）比较
- **示例**：`isVoid(void) → true`，`isVoid(1) → false`

```bocchi
fn isVoid(x) {
    return x == void;
}
```

---

## 分类速查表

| 类别 | 方法 |
|---|---|
| **数值** | `min` `max` `sum` `avg` `abs` `formax` |
| **列表** | `addlist` `del` `setlist` `isinlist` `indexof` `countof` `join` `choice` |
| **字符串** | `trim` `repeatstring` `padleft` `padright` |
| **文件** | `fileadd` |
| **工具** | `isVoid` |

---

## 依赖关系

```
addlist ──► del
         └─► setlist
formax ──► test.txt（全局中转文件，未清理）
fileadd ─► file2（全局变量，被覆盖）
```

### 潜在问题

- **`addlist` 是核心但实现脆弱**：依赖 `test.txt` 序列化中转，循环条件 `i < x.len + 1` 与外部变量 `i` 耦合，循环内 `set a = ...` 反复重定义，对含空格或特殊字符的列表元素容易出错。
- **`del` / `setlist` 的正确性完全取决于 `addlist`**，若 `addlist` 解析失败，二者也会失败。
- **`formax` 同样依赖 `test.txt`**，且循环 `for..in i b+2` 会写入 `b+2` 个字符，与"前 b 位"的语义有偏差（多取 2 位），实际依赖 `float` 解析截断。
- **`trim` 是简化版**：只删空格，不处理 `\t`、`\n` 和首尾空白。
- **`void` / `nil` 是普通字符串**，`isVoid` 用 `==` 比较字符串。

---

## 使用示例

### 常用调用

```bocchi
// 数学
puts min(3, 1, 4, 1, 5);       // 1
puts max(3, 1, 4, 1, 5);       // 5
puts sum(1, 2, 3);              // 6
puts avg(1, 3, 4324, 2342);    // 1667.5
puts abs(-10);                  // 10
puts formax("3.1415926", 3);   // 3.14

// 列表
puts isinlist([1,2,3,4,5], 3);           // true
puts indexof([1,2,3,4,5], 3);            // 2
puts countof([1,2,3,4,5,3], 3);          // 2
puts join([1,2,3,4,5], ".");             // "1.2.3.4.5"
puts choice([1,2,3,4,5]);                // 随机
puts del(["a","b","c","d","e"], 2);      // ["a","b","d","e"]
puts setlist(["a","b","c"], 1, "X");     // ["a","X","c"]

// 字符串
puts trim("  123  ");                    // "123"
puts repeatstring("hello", 3);           // "hellohellohello"
puts padleft("5", "0", 3);               // "005"
puts padright("5", "0", 3);              // "500"

// 文件
puts fileadd("use.txt", "1");

// 工具
puts isVoid(void);                       // true
puts isVoid(1);                          // false
```