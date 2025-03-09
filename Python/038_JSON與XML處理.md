[上一章：日期與時間處理](037_日期與時間處理.md) | [下一章：網路程式設計基礎](039_網路程式設計基礎.md)

# Python JSON與XML處理 📄

## JSON 處理基礎

### 1. JSON 基本操作

```python
import json

# Python 對象轉換為 JSON 字符串
data = {
    "name": "Alice",
    "age": 25,
    "city": "Taipei",
    "interests": ["reading", "music", "travel"],
    "is_student": True,
    "height": 165.5
}

# 轉換為 JSON 字符串
json_str = json.dumps(data, ensure_ascii=False, indent=4)
print("JSON 字符串：")
print(json_str)

# JSON 字符串轉換為 Python 對象
parsed_data = json.loads(json_str)
print("\n解析後的數據：")
print(f"姓名：{parsed_data['name']}")
print(f"年齡：{parsed_data['age']}")
```

### 2. 檔案讀寫

```python
import json

# 寫入 JSON 檔案
data = {
    "users": [
        {"id": 1, "name": "Alice", "email": "alice@example.com"},
        {"id": 2, "name": "Bob", "email": "bob@example.com"},
        {"id": 3, "name": "Charlie", "email": "charlie@example.com"}
    ]
}

with open('users.json', 'w', encoding='utf-8') as f:
    json.dump(data, f, ensure_ascii=False, indent=4)

# 讀取 JSON 檔案
with open('users.json', 'r', encoding='utf-8') as f:
    loaded_data = json.load(f)

for user in loaded_data['users']:
    print(f"用戶 {user['id']}: {user['name']} ({user['email']})")
```

## JSON 進階操作

### 1. 自定義編碼器

```python
import json
from datetime import datetime

class DateTimeEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.isoformat()
        return super().default(obj)

# 使用自定義編碼器
data = {
    "name": "Event",
    "date": datetime.now(),
    "location": "Taipei"
}

json_str = json.dumps(data, cls=DateTimeEncoder, indent=4)
print(json_str)
```

### 2. JSON Schema 驗證

```python
from jsonschema import validate

# 定義 Schema
schema = {
    "type": "object",
    "properties": {
        "name": {"type": "string"},
        "age": {"type": "integer", "minimum": 0},
        "email": {"type": "string", "format": "email"},
        "interests": {
            "type": "array",
            "items": {"type": "string"}
        }
    },
    "required": ["name", "age", "email"]
}

# 驗證數據
def validate_user_data(data):
    try:
        validate(instance=data, schema=schema)
        return True
    except Exception as e:
        print(f"驗證錯誤：{e}")
        return False

# 測試
test_data = {
    "name": "Alice",
    "age": 25,
    "email": "alice@example.com",
    "interests": ["reading", "music"]
}

print(f"驗證結果：{validate_user_data(test_data)}")
```

## XML 處理基礎

### 1. 解析 XML

```python
import xml.etree.ElementTree as ET

# 創建 XML 字符串
xml_string = '''
<users>
    <user id="1">
        <name>Alice</name>
        <email>alice@example.com</email>
        <roles>
            <role>admin</role>
            <role>user</role>
        </roles>
    </user>
    <user id="2">
        <name>Bob</name>
        <email>bob@example.com</email>
        <roles>
            <role>user</role>
        </roles>
    </user>
</users>
'''

# 解析 XML
root = ET.fromstring(xml_string)

# 遍歷 XML
for user in root.findall('user'):
    user_id = user.get('id')
    name = user.find('name').text
    email = user.find('email').text
    roles = [role.text for role in user.findall('./roles/role')]
    
    print(f"用戶 {user_id}:")
    print(f"  姓名: {name}")
    print(f"  郵箱: {email}")
    print(f"  角色: {', '.join(roles)}")
```

### 2. 創建和修改 XML

```python
import xml.etree.ElementTree as ET
from xml.dom import minidom

def create_xml_document():
    # 創建根元素
    root = ET.Element('books')
    
    # 添加書籍
    books_data = [
        {
            'title': 'Python 程式設計',
            'author': 'John Smith',
            'year': '2023',
            'price': '29.99'
        },
        {
            'title': '資料科學入門',
            'author': 'Jane Doe',
            'year': '2022',
            'price': '34.99'
        }
    ]
    
    for book_data in books_data:
        book = ET.SubElement(root, 'book')
        for key, value in book_data.items():
            elem = ET.SubElement(book, key)
            elem.text = value
    
    # 格式化 XML
    xml_str = minidom.parseString(ET.tostring(root)).toprettyxml(indent="    ")
    
    # 保存到檔案
    with open('books.xml', 'w', encoding='utf-8') as f:
        f.write(xml_str)

# 創建 XML 檔案
create_xml_document()
```

## XML 進階操作

### 1. XPath 查詢

```python
import xml.etree.ElementTree as ET

# XML 數據
xml_string = '''
<library>
    <category name="programming">
        <book price="29.99">
            <title>Python 基礎教程</title>
            <author>John Smith</author>
            <year>2023</year>
        </book>
        <book price="34.99">
            <title>Python 進階編程</title>
            <author>Jane Doe</author>
            <year>2022</year>
        </book>
    </category>
    <category name="data_science">
        <book price="39.99">
            <title>數據分析入門</title>
            <author>Bob Wilson</author>
            <year>2023</year>
        </book>
    </category>
</library>
'''

root = ET.fromstring(xml_string)

# 使用 XPath 查詢
def xpath_examples(root):
    # 查找所有書籍
    books = root.findall('.//book')
    print(f"總共有 {len(books)} 本書")
    
    # 查找特定價格範圍的書籍
    expensive_books = root.findall('.//book[@price>"30.00"]')
    print("\n價格超過 30.00 的書籍：")
    for book in expensive_books:
        title = book.find('title').text
        price = book.get('price')
        print(f"- {title} (${price})")
    
    # 查找特定年份的書籍
    books_2023 = root.findall('.//book[year="2023"]')
    print("\n2023年出版的書籍：")
    for book in books_2023:
        title = book.find('title').text
        author = book.find('author').text
        print(f"- {title} by {author}")

xpath_examples(root)
```

### 2. XML 驗證

```python
from lxml import etree

# 定義 XSD Schema
xsd_string = '''
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">
    <xs:element name="library">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="book" maxOccurs="unbounded">
                    <xs:complexType>
                        <xs:sequence>
                            <xs:element name="title" type="xs:string"/>
                            <xs:element name="author" type="xs:string"/>
                            <xs:element name="year" type="xs:integer"/>
                            <xs:element name="price" type="xs:decimal"/>
                        </xs:sequence>
                        <xs:attribute name="id" type="xs:string" use="required"/>
                    </xs:complexType>
                </xs:element>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
</xs:schema>
'''

def validate_xml(xml_string, xsd_string):
    try:
        schema_root = etree.XML(xsd_string)
        schema = etree.XMLSchema(schema_root)
        parser = etree.XMLParser(schema=schema)
        etree.fromstring(xml_string, parser)
        return True
    except Exception as e:
        print(f"驗證錯誤：{e}")
        return False

# 測試 XML 驗證
xml_to_validate = '''
<library>
    <book id="1">
        <title>Python 程式設計</title>
        <author>John Smith</author>
        <year>2023</year>
        <price>29.99</price>
    </book>
</library>
'''

print(f"XML 驗證結果：{validate_xml(xml_to_validate, xsd_string)}")
```

## 實際應用範例

### 1. 配置檔案處理

```python
import json
import xml.etree.ElementTree as ET

class ConfigManager:
    def __init__(self):
        self.config = {}
    
    def load_json_config(self, filename):
        """載入 JSON 配置檔案"""
        with open(filename, 'r', encoding='utf-8') as f:
            self.config.update(json.load(f))
    
    def load_xml_config(self, filename):
        """載入 XML 配置檔案"""
        tree = ET.parse(filename)
        root = tree.getroot()
        
        def xml_to_dict(element):
            result = {}
            for child in element:
                if len(child) > 0:
                    result[child.tag] = xml_to_dict(child)
                else:
                    result[child.tag] = child.text
            return result
        
        self.config.update(xml_to_dict(root))
    
    def get_value(self, key, default=None):
        """獲取配置值"""
        return self.config.get(key, default)
    
    def save_json_config(self, filename):
        """保存為 JSON 配置檔案"""
        with open(filename, 'w', encoding='utf-8') as f:
            json.dump(self.config, f, indent=4, ensure_ascii=False)

# 使用示例
config_manager = ConfigManager()
config_manager.config = {
    "database": {
        "host": "localhost",
        "port": 5432,
        "name": "mydb"
    },
    "server": {
        "host": "0.0.0.0",
        "port": 8080
    }
}

# 保存配置
config_manager.save_json_config('config.json')
```

### 2. 數據轉換工具

```python
import json
import xml.etree.ElementTree as ET
from xml.dom import minidom

def json_to_xml(json_data):
    """將 JSON 數據轉換為 XML"""
    def create_element(parent, key, value):
        if isinstance(value, dict):
            elem = ET.SubElement(parent, key)
            for k, v in value.items():
                create_element(elem, k, v)
        elif isinstance(value, list):
            elem = ET.SubElement(parent, key)
            for item in value:
                create_element(elem, 'item', item)
        else:
            elem = ET.SubElement(parent, key)
            elem.text = str(value)
    
    root = ET.Element('root')
    for key, value in json_data.items():
        create_element(root, key, value)
    
    return minidom.parseString(ET.tostring(root)).toprettyxml(indent="    ")

def xml_to_json(xml_string):
    """將 XML 轉換為 JSON"""
    def element_to_dict(element):
        result = {}
        for child in element:
            if len(child) > 0:
                if child.tag in result:
                    if isinstance(result[child.tag], list):
                        result[child.tag].append(element_to_dict(child))
                    else:
                        result[child.tag] = [result[child.tag], element_to_dict(child)]
                else:
                    result[child.tag] = element_to_dict(child)
            else:
                result[child.tag] = child.text
        return result
    
    root = ET.fromstring(xml_string)
    return element_to_dict(root)

# 測試數據轉換
test_data = {
    "person": {
        "name": "Alice",
        "age": 25,
        "contacts": {
            "email": "alice@example.com",
            "phone": "1234567890"
        },
        "interests": ["reading", "music", "travel"]
    }
}

# JSON 轉 XML
xml_output = json_to_xml(test_data)
print("XML 輸出：")
print(xml_output)

# XML 轉回 JSON
json_output = xml_to_json(xml_output)
print("\nJSON 輸出：")
print(json.dumps(json_output, indent=4, ensure_ascii=False))
```

## 練習題

1. **數據格式轉換器**
   實現一個數據格式轉換工具：
   - JSON 轉 XML
   - XML 轉 JSON
   - 支援複雜結構
   - 保持數據完整性

2. **配置管理系統**
   創建一個配置管理系統：
   - 支援多種格式
   - 合併配置文件
   - 驗證配置有效性
   - 自動備份功能

3. **API 響應處理器**
   開發一個 API 響應處理工具：
   - 解析 JSON/XML 響應
   - 數據驗證
   - 錯誤處理
   - 格式轉換

## 小提醒 💡

1. 處理大型文件時注意記憶體使用
2. 正確處理字符編碼
3. 注意數據驗證和安全性
4. 使用適當的解析器
5. 處理異常情況
6. 考慮數據格式的兼容性

[上一章：日期與時間處理](037_日期與時間處理.md) | [下一章：網路程式設計基礎](039_網路程式設計基礎.md) 