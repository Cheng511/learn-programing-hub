# Python break與continue 🔀

[上一章：for迴圈基礎](009_for迴圈基礎.md) | [下一章：pass與else子句](011_pass與else子句.md)

在迴圈中，有時我們需要提前結束迴圈或跳過某些特定的迭代。Python 提供了 `break` 和 `continue` 這兩個關鍵字來實現這些功能。

## break 關鍵字 ⛔

`break` 用於完全結束迴圈，不再執行後續的迭代。

### 基本用法
```python
# 在 for 迴圈中使用 break
for i in range(1, 6):
    if i == 3:
        break
    print(i)
# 輸出：1, 2

# 在 while 迴圈中使用 break
count = 1
while True:
    if count > 3:
        break
    print(count)
    count += 1
# 輸出：1, 2, 3
```

### 實用範例
```python
# 1. 尋找第一個能被7整除的數
for num in range(1, 100):
    if num % 7 == 0:
        print(f"找到了：{num}")
        break

# 2. 簡單的選單系統
while True:
    print("\n1. 開始遊戲")
    print("2. 設定")
    print("3. 離開")
    choice = input("請選擇：")
    
    if choice == "3":
        print("謝謝使用！")
        break
```

## continue 關鍵字 ⏭️

`continue` 用於跳過當前迭代，直接進入下一次迭代。

### 基本用法
```python
# 在 for 迴圈中使用 continue
for i in range(1, 6):
    if i == 3:
        continue
    print(i)
# 輸出：1, 2, 4, 5

# 在 while 迴圈中使用 continue
count = 0
while count < 5:
    count += 1
    if count == 3:
        continue
    print(count)
# 輸出：1, 2, 4, 5
```

### 實用範例
```python
# 1. 印出非偶數
for num in range(1, 11):
    if num % 2 == 0:
        continue
    print(num)  # 只印出奇數

# 2. 過濾特定條件
scores = [85, -1, 92, -1, 78, 65, -1]
for score in scores:
    if score == -1:  # 跳過無效分數
        continue
    print(f"有效分數：{score}")
```

## break 和 continue 的進階用法 🎯

### 1. 在巢狀迴圈中使用
```python
# break 只會跳出最內層的迴圈
for i in range(3):
    for j in range(3):
        if j == 2:
            break
        print(f"i={i}, j={j}")

# 如果要跳出外層迴圈，可以使用標記
found = False
for i in range(3):
    for j in range(3):
        if i * j == 4:
            found = True
            break
    if found:
        break
```

### 2. 搭配 else 子句
```python
# 當迴圈正常結束（沒有被break中斷）時執行else
for i in range(5):
    if i == 10:  # 永遠不會成立
        break
else:
    print("迴圈正常完成！")

# 被break中斷時不執行else
for i in range(5):
    if i == 3:
        break
else:
    print("這行不會執行")
```

## 實際應用範例 💡

### 1. 資料驗證
```python
while True:
    password = input("請輸入密碼（至少6位）：")
    if len(password) < 6:
        print("密碼太短，請重新輸入")
        continue
    
    if password.isdigit():
        print("密碼不能只包含數字")
        continue
    
    print("密碼設定成功！")
    break
```

### 2. 搜尋系統
```python
data = ["蘋果", "香蕉", "橘子", "葡萄", "西瓜"]
search = "橘子"

for index, item in enumerate(data):
    if item == search:
        print(f"找到 {search} 在位置 {index}")
        break
else:
    print(f"找不到 {search}")
```

## 練習時間 💪

1. 實作一個簡單的除數計算器：
```python
number = 12
print(f"{number} 的除數有：")
for i in range(1, number + 1):
    if number % i != 0:
        continue
    print(i, end=" ")
```

2. 實作一個質數判斷器：
```python
num = 17
is_prime = True

for i in range(2, int(num ** 0.5) + 1):
    if num % i == 0:
        is_prime = False
        break
        
print(f"{num} {'是' if is_prime else '不是'}質數")
```

## 小提醒 💡

- break 用於完全結束迴圈
- continue 用於跳過當前迭代
- 在巢狀迴圈中要注意 break 只影響最內層迴圈
- 適時使用 else 子句來處理迴圈的正常結束
- 不要過度使用 break 和 continue，可能影響程式碼的可讀性

[上一章：for迴圈基礎](009_for迴圈基礎.md) | [下一章：pass與else子句](011_pass與else子句.md)

---
下一章，我們將學習Python的pass與else子句！ 🚀 