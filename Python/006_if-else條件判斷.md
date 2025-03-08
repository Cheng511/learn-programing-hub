# Python if-else條件判斷 🔀

[上一章：運算子](005_運算子.md) | [下一章：if-elif-else多重條件](007_if-elif-else多重條件.md)

在程式設計中，我們經常需要根據不同的條件來執行不同的程式碼。Python的 if-else 語句就是用來處理這種情況的工具。

## 基本的 if 語句 ✨

最簡單的條件判斷只需要 `if`：

```python
age = 18
if age >= 18:
    print("你已經成年了！")
```

記住：Python 使用縮排來區分程式碼區塊！

## if-else 語句 🔄

當條件不成立時，我們可能想執行其他程式碼：

```python
temperature = 38

if temperature >= 37.5:
    print("體溫過高，建議就醫！")
else:
    print("體溫正常，很健康！")
```

## 條件運算式 🎯

我們可以使用各種運算式來做條件判斷：

```python
# 比較運算子
age = 16
is_student = True

if age < 18:
    print("未成年")

# 邏輯運算子
if age < 18 and is_student:
    print("你是學生，可以享有優惠！")

# 成員運算子
fruits = ["蘋果", "香蕉", "橘子"]
if "蘋果" in fruits:
    print("有蘋果！")
```

## 實用範例 💡

### 1. 簡單的登入檢查
```python
username = "python"
password = "12345"

if username == "python" and password == "12345":
    print("登入成功！")
else:
    print("帳號或密碼錯誤！")
```

### 2. 年齡分類
```python
age = 25

if age < 12:
    print("兒童")
else:
    print("青少年或成人")
```

### 3. 成績評級
```python
score = 85

if score >= 90:
    print("優秀！")
else:
    print("繼續加油！")
```

## 條件判斷的技巧 🔍

1. 使用括號增加可讀性：
```python
if (age >= 18) and (score >= 60):
    print("符合條件")
```

2. 避免巢狀的條件判斷太深：
```python
# 不好的寫法
if condition1:
    if condition2:
        if condition3:
            print("太多層了！")

# 較好的寫法
if condition1 and condition2 and condition3:
    print("更清楚！")
```

3. 善用 `in` 運算子：
```python
# 不好的寫法
if fruit == "蘋果" or fruit == "香蕉" or fruit == "橘子":
    print("這是水果")

# 較好的寫法
if fruit in ["蘋果", "香蕉", "橘子"]:
    print("這是水果")
```

## 練習時間 💪

試試看解決以下問題：

1. 檢查數字是否為偶數：
```python
number = 10
if number % 2 == 0:
    print(f"{number} 是偶數")
else:
    print(f"{number} 是奇數")
```

2. 簡單的訂票系統：
```python
age = 25
price = 100

if age < 12:
    price = price * 0.5  # 兒童半價
else:
    price = price * 1.0  # 原價

print(f"票價為：{price} 元")
```

## 小提醒 💡

- 條件判斷要清楚明確
- 注意縮排的正確性
- 善用邏輯運算子組合條件
- 保持程式碼的可讀性

[上一章：運算子](005_運算子.md) | [下一章：if-elif-else多重條件](007_if-elif-else多重條件.md)

---
下一章，我們將學習Python的if-elif-else多重條件！ 🚀 