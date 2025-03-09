# Python if-elif-else多重條件 🔀

[上一章：if-else條件判斷](006_if-else條件判斷.md) | [下一章：while迴圈基礎](008_while迴圈基礎.md)

有時候我們需要判斷多個條件，並根據不同的條件執行不同的程式碼。這時候就需要用到 `if-elif-else` 結構。

## if-elif-else 基本結構 📝

```python
score = 85

if score >= 90:
    print("優秀！")
elif score >= 80:
    print("良好！")
elif score >= 70:
    print("及格！")
else:
    print("需要加油！")
```

## 多重條件的運作方式 🎯

1. 條件是**依序**判斷的
2. 一旦某個條件成立，就會執行對應的程式碼
3. 其他的條件就不會再判斷了
4. `else` 是可選的，用於處理所有條件都不成立的情況

## 實用範例 💡

### 1. BMI 計算器
```python
weight = 65  # 公斤
height = 1.75  # 公尺
bmi = weight / (height ** 2)

if bmi < 18.5:
    print("體重過輕")
elif bmi < 24:
    print("體重正常")
elif bmi < 27:
    print("體重過重")
else:
    print("肥胖")
```

### 2. 季節判斷
```python
month = 7

if month in [3, 4, 5]:
    print("春天")
elif month in [6, 7, 8]:
    print("夏天")
elif month in [9, 10, 11]:
    print("秋天")
elif month in [12, 1, 2]:
    print("冬天")
else:
    print("月份無效")
```

### 3. 簡單的計算機
```python
num1 = 10
num2 = 5
operator = "+"

if operator == "+":
    result = num1 + num2
elif operator == "-":
    result = num1 - num2
elif operator == "*":
    result = num1 * num2
elif operator == "/":
    if num2 != 0:  # 防止除以零
        result = num1 / num2
    else:
        result = "錯誤：除數不能為零"
else:
    result = "不支援的運算符"

print(f"計算結果：{result}")
```

## 條件判斷的優化技巧 🔍

### 1. 條件順序的優化
```python
# 不好的寫法
if score >= 60:
    print("及格")
elif score >= 80:
    print("良好")  # 永遠不會執行到

# 好的寫法
if score >= 80:
    print("良好")
elif score >= 60:
    print("及格")
```

### 2. 使用字典替代多重條件
```python
# 使用多重條件
if grade == "A":
    score = 90
elif grade == "B":
    score = 80
elif grade == "C":
    score = 70
else:
    score = 60

# 使用字典（更簡潔）
grade_to_score = {
    "A": 90,
    "B": 80,
    "C": 70
}
score = grade_to_score.get(grade, 60)  # 找不到時預設 60
```

## 練習時間 💪

1. 製作一個簡單的遊戲分級系統：
```python
age = 15
violence_level = "低"
educational = True

if age < 12:
    if educational:
        rating = "普遍級"
    else:
        rating = "保護級"
elif age < 15:
    if violence_level == "低":
        rating = "保護級"
    else:
        rating = "輔導級"
else:
    rating = "輔導級"

print(f"遊戲分級：{rating}")
```

2. 根據消費金額計算折扣：
```python
amount = 2500
member = True

if amount >= 5000:
    discount = 0.8  # 8折
elif amount >= 3000:
    discount = 0.85  # 85折
elif amount >= 1000:
    discount = 0.9  # 9折
else:
    discount = 1.0  # 原價

# 會員額外95折
if member:
    discount *= 0.95

final_price = amount * discount
print(f"實付金額：{final_price} 元")
```

## 小提醒 💡

- 條件判斷要由嚴格到寬鬆
- 善用 `elif` 來避免過多的巢狀結構
- 考慮使用字典或其他資料結構來簡化多重條件
- 記得處理所有可能的情況
- 適時加入註解說明判斷邏輯

[上一章：if-else條件判斷](006_if-else條件判斷.md) | [下一章：while迴圈基礎](008_while迴圈基礎.md)

---
下一章，我們將學習Python的while迴圈基礎！ 🚀 