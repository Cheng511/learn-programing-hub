# Python while迴圈基礎 🔄

[上一章：if-elif-else多重條件](007_if-elif-else多重條件.md) | [下一章：for迴圈基礎](009_for迴圈基礎.md)

當我們需要重複執行某些程式碼，直到特定條件不再成立時，就可以使用 while 迴圈。

## while 迴圈的基本結構 📝

```python
# 基本語法
while 條件:
    # 要重複執行的程式碼

# 實際例子
count = 1
while count <= 5:
    print(f"第 {count} 次執行")
    count += 1
```

## while 迴圈的運作方式 🎯

1. 檢查條件是否為 True
2. 如果是 True，執行程式碼區塊
3. 執行完後回到第1步
4. 如果是 False，跳出迴圈

## 實用範例 💡

### 1. 猜數字遊戲
```python
import random

target = random.randint(1, 100)
guess = 0
attempts = 0

while guess != target:
    guess = int(input("猜一個1-100的數字："))
    attempts += 1
    
    if guess > target:
        print("太大了！")
    elif guess < target:
        print("太小了！")
    else:
        print(f"恭喜猜對了！共猜了 {attempts} 次")
```

### 2. 密碼驗證
```python
correct_password = "python123"
max_attempts = 3
attempts = 0

while attempts < max_attempts:
    password = input("請輸入密碼：")
    attempts += 1
    
    if password == correct_password:
        print("登入成功！")
        break
    else:
        remaining = max_attempts - attempts
        print(f"密碼錯誤，還有 {remaining} 次機會")

if attempts >= max_attempts:
    print("登入失敗次數過多，帳號已鎖定")
```

### 3. 計算總和
```python
# 計算 1 到 10 的總和
total = 0
number = 1

while number <= 10:
    total += number
    number += 1

print(f"1到10的總和是：{total}")
```

## 無限迴圈與中斷 ⚠️

### 1. 無限迴圈
```python
# 無限迴圈
while True:
    response = input("要繼續嗎？(y/n)：")
    if response.lower() == 'n':
        break
    print("繼續執行...")
```

### 2. 使用 break 中斷迴圈
```python
# 找到第一個能被7整除的數
number = 1
while number <= 100:
    if number % 7 == 0:
        print(f"找到了：{number}")
        break
    number += 1
```

### 3. 使用 continue 跳過當前迭代
```python
# 印出 1-10 中的奇數
number = 0
while number < 10:
    number += 1
    if number % 2 == 0:  # 如果是偶數
        continue         # 跳過這次迭代
    print(number)
```

## 常見的迴圈模式 📋

### 1. 計數迴圈
```python
counter = 0
while counter < 5:
    print(f"計數：{counter}")
    counter += 1
```

### 2. 累加迴圈
```python
numbers = [1, 2, 3, 4, 5]
index = 0
sum = 0

while index < len(numbers):
    sum += numbers[index]
    index += 1

print(f"總和：{sum}")
```

### 3. 輸入驗證
```python
while True:
    age = input("請輸入年齡（1-120）：")
    if age.isdigit() and 1 <= int(age) <= 120:
        print(f"你的年齡是：{age}")
        break
    print("請輸入有效的年齡！")
```

## 練習時間 💪

1. 實作一個簡單的 ATM：
```python
balance = 1000
while True:
    print("\n1. 查詢餘額")
    print("2. 提款")
    print("3. 存款")
    print("4. 離開")
    
    choice = input("請選擇操作：")
    
    if choice == "1":
        print(f"餘額：{balance}")
    elif choice == "2":
        amount = int(input("請輸入提款金額："))
        if amount <= balance:
            balance -= amount
            print("提款成功")
        else:
            print("餘額不足")
    elif choice == "3":
        amount = int(input("請輸入存款金額："))
        balance += amount
        print("存款成功")
    elif choice == "4":
        print("謝謝使用")
        break
```

## 小提醒 💡

- 確保迴圈有終止條件
- 避免無限迴圈（除非有特殊需求）
- 適時使用 break 和 continue
- 注意迴圈變數的更新
- 考慮使用 for 迴圈處理已知次數的重複

[上一章：if-elif-else多重條件](007_if-elif-else多重條件.md) | [下一章：for迴圈基礎](009_for迴圈基礎.md)

---
下一章，我們將學習Python的for迴圈基礎！ 🚀 