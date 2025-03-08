# Python pass與else子句 🎯

[上一章：break與continue](010_break與continue.md) | [下一章：巢狀條件判斷](012_巢狀條件判斷.md)

在 Python 中，`pass` 是一個特殊的關鍵字，而 `else` 子句不只能用在條件判斷中，還能用在迴圈中。讓我們來深入了解它們的用法。

## pass 關鍵字 ⏭️

`pass` 是一個空操作，當語法需要一個語句但程式不需要執行任何動作時使用。

### 1. 基本用法
```python
# 在函式中使用
def my_function():
    pass  # 稍後再實作

# 在類別中使用
class MyClass:
    pass  # 稍後再加入屬性和方法

# 在條件判斷中使用
if condition:
    pass
else:
    print("執行其他操作")
```

### 2. 常見應用場景
```python
# 1. 預留程式架構
def process_data():
    # TODO: 實作資料處理邏輯
    pass

# 2. 最小類別定義
class EmptyClass:
    pass

# 3. 忽略特定條件
for i in range(100):
    if i % 2 == 0:
        pass  # 忽略偶數
    else:
        print(i)  # 只處理奇數
```

## else 子句的進階用法 🔄

### 1. 在 for 迴圈中使用 else
```python
# 搜尋數字
numbers = [1, 3, 5, 7, 9]
search = 4

for num in numbers:
    if num == search:
        print("找到了！")
        break
else:
    print("沒有找到")  # 當迴圈正常結束時執行

# 檢查質數
def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(n ** 0.5) + 1):
        if n % i == 0:
            return False
    else:
        return True  # 當沒有找到除數時執行
```

### 2. 在 while 迴圈中使用 else
```python
# 嘗試次數限制
attempts = 3
while attempts > 0:
    password = input("請輸入密碼：")
    if password == "secret":
        print("登入成功！")
        break
    attempts -= 1
else:
    print("登入失敗次數過多")  # 當嘗試次數用完時執行
```

## pass 與 else 的組合應用 🎯

### 1. 錯誤處理
```python
def process_file(filename):
    try:
        with open(filename) as f:
            content = f.read()
    except FileNotFoundError:
        pass  # 忽略檔案不存在的錯誤
    else:
        return content  # 檔案存在且讀取成功時執行
```

### 2. 資料驗證
```python
def validate_user_input():
    while True:
        value = input("請輸入一個正整數：")
        if not value.isdigit():
            pass  # 忽略無效輸入
        else:
            return int(value)  # 輸入有效時返回
```

## 實用範例 💡

### 1. 簡單的狀態機
```python
class StateMachine:
    def state1(self):
        pass  # 預留狀態1的處理邏輯
    
    def state2(self):
        pass  # 預留狀態2的處理邏輯
    
    def run(self):
        while True:
            if condition1:
                self.state1()
            elif condition2:
                self.state2()
            else:
                break
```

### 2. 資料過濾器
```python
def filter_data(data):
    result = []
    for item in data:
        if item is None:
            pass  # 忽略空值
        elif isinstance(item, str) and not item.strip():
            pass  # 忽略空字串
        else:
            result.append(item)
    return result

# 使用範例
data = [1, None, "", "hello", " ", 42]
filtered = filter_data(data)
print(filtered)  # [1, 'hello', 42]
```

## 練習時間 💪

1. 實作一個簡單的計算機：
```python
def calculator():
    while True:
        try:
            num1 = float(input("第一個數字："))
            operator = input("運算符號 (+,-,*,/)：")
            num2 = float(input("第二個數字："))
            
            if operator not in ['+', '-', '*', '/']:
                pass  # 忽略無效的運算符
            else:
                if operator == '+':
                    print(f"結果：{num1 + num2}")
                elif operator == '-':
                    print(f"結果：{num1 - num2}")
                elif operator == '*':
                    print(f"結果：{num1 * num2}")
                elif operator == '/':
                    if num2 == 0:
                        print("錯誤：除數不能為零")
                        pass
                    else:
                        print(f"結果：{num1 / num2}")
                break
        except ValueError:
            pass  # 忽略無效的數字輸入
```

2. 檢查檔案內容：
```python
def check_file_content(filename, keyword):
    try:
        with open(filename, 'r') as file:
            for line in file:
                if keyword in line:
                    print(f"找到關鍵字：{line.strip()}")
                    break
            else:
                print("沒有找到關鍵字")
    except FileNotFoundError:
        pass  # 忽略檔案不存在的錯誤
```

## 小提醒 💡

- pass 用於保持程式結構的完整性
- 不要濫用 pass，應該在有明確意圖時使用
- else 子句可以讓程式邏輯更清晰
- 在迴圈中的 else 只有在迴圈正常結束時才會執行
- 適當的註解可以讓 pass 的用意更明確

[上一章：break與continue](010_break與continue.md) | [下一章：巢狀條件判斷](012_巢狀條件判斷.md)

---
下一章，我們將學習Python的巢狀條件判斷！ 🚀 