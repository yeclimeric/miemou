# karpathy 四原则示例

> 源自开源 skill **karpathy-guidelines**（[multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills)，MIT License）的 `EXAMPLES.md`，完整迁移。
> 与 `references/member-guidelines.md` §1 配套使用：规则正文见 §1，本文件是四原则的真实代码示例。

真实代码示例，逐条演示四原则。每个示例展示 LLM 常见的错误做法与修正方式。

---

## 1. 三思而后写（Think Before Coding）

### 示例 1：隐藏假设

**用户请求**："加一个导出用户数据的功能"

**❌ LLM 常见做法（臆测假设）**

```python
def export_users(format='json'):
    """Export all users to a file."""
    users = User.query.all()
    
    if format == 'json':
        with open('users.json', 'w') as f:
            json.dump([u.to_dict() for u in users], f)
    elif format == 'csv':
        with open('users.csv', 'w') as f:
            writer = csv.DictWriter(f, fieldnames=['id', 'email', 'name'])
            writer.writeheader()
            writer.writerows([u.to_dict() for u in users])
    
    return f"Exported {len(users)} users"
```

**问题：**

- 臆测了要导出**全部**用户（分页呢？隐私呢？）
- 没问就把文件保存位置定了
- 臆测了要包含哪些字段
- 没核对实际数据结构就定了 CSV 列名

**✅ 正确做法（呈现假设）**

```
实现之前，我需要澄清：

1. **范围**：导出全部用户还是筛选后的子集？（涉及隐私）
2. **形式**：你说"导出"——指的是：
   - 浏览器下载文件？
   - 后台任务完成后邮件通知？
   - API 端点返回数据？
3. **字段**：哪些用户字段？（有些可能是敏感信息）
4. **量级**：通常多少用户？（影响方案选型）

最简单的做法：加一个分页返回 JSON 的 API 端点。
文件导出需要更多信息。

你的偏好是？
```

### 示例 2：多种解读并存

**用户请求**："让搜索变快"

**❌ LLM 常见做法（默默挑一个）**

```python
# Adds caching, database indexes, and async processing
# without asking which "faster" matters
@lru_cache(maxsize=1000)
async def search(query: str) -> List[Result]:
    # ... 200 lines of optimization code
```

**✅ 正确做法**

```
"让搜索变快"可以指：

1. **响应更快**（< 100ms 而不是 500ms）
   - 加数据库索引
   - 缓存高频查询
   - 预估工时：2 小时

2. **扛住更多并发搜索**（吞吐量）
   - 加 async/await
   - 连接池
   - 预估工时：4 小时

3. **感知上更快**（体验）
   - 立即展示部分结果
   - 渐进式加载
   - 预估工时：3 小时

当前典型查询耗时约 500ms。
你的场景里哪个维度最重要？
```

---

## 2. 简单优先（Simplicity First）

### 示例 1：过度抽象

**用户请求**："加一个计算折扣的函数"

**❌ LLM 常见做法（过度工程）**

```python
from abc import ABC, abstractmethod
from enum import Enum
from typing import Protocol, Union
from dataclasses import dataclass

class DiscountStrategy(ABC):
    @abstractmethod
    def calculate(self, amount: float) -> float:
        pass

class PercentageDiscount(DiscountStrategy):
    def __init__(self, percentage: float):
        self.percentage = percentage
    
    def calculate(self, amount: float) -> float:
        return amount * (self.percentage / 100)

class FixedDiscount(DiscountStrategy):
    def __init__(self, fixed_amount: float):
        self.fixed_amount = fixed_amount
    
    def calculate(self, amount: float) -> float:
        return min(self.fixed_amount, amount)

@dataclass
class DiscountConfig:
    strategy: DiscountStrategy
    min_purchase: float = 0.0
    max_discount: float = float('inf')

class DiscountCalculator:
    def __init__(self, config: DiscountConfig):
        self.config = config
    
    def apply_discount(self, amount: float) -> float:
        if amount < self.config.min_purchase:
            return 0
        discount = self.config.strategy.calculate(amount)
        return min(discount, self.config.max_discount)

# Usage requires 30+ lines of setup for a simple calculation
```

**✅ 正确做法（简单）**

```python
def calculate_discount(amount: float, percent: float) -> float:
    """Calculate discount amount. percent should be 0-100."""
    return amount * (percent / 100)

# Usage
discount = calculate_discount(100.0, 10.0)  # $10 off
```

**何时加复杂度**：真的需要多种折扣类型时。需求来了再重构。

### 示例 2：投机性功能

**用户请求**："把用户偏好存到数据库"

**❌ LLM 常见做法（没人要求的功能）**

```python
class PreferenceManager:
    def __init__(self, db, cache=None, validator=None):
        self.db = db
        self.cache = cache or InMemoryCache()
        self.validator = validator or DefaultValidator()
    
    def save(self, user_id: int, prefs: dict, 
             merge: bool = True, 
             validate: bool = True,
             notify: bool = False) -> bool:
        """Save preferences with optional merging, validation, and notifications."""
        
        if validate:
            errors = self.validator.validate(prefs)
            if errors:
                raise ValidationError(errors)
        
        existing = self.get(user_id) if merge else {}
        
        if merge:
            existing.update(prefs)
            prefs = existing
        
        self.db.execute(
            "UPDATE users SET preferences = ? WHERE id = ?",
            (json.dumps(prefs), user_id)
        )
        
        if self.cache:
            self.cache.set(f"prefs:{user_id}", prefs)
        
        if notify:
            self.notify_preference_change(user_id, prefs)
        
        return True
    
    def notify_preference_change(self, user_id: int, prefs: dict):
        # Another 30 lines for a feature nobody asked for
        pass
```

**✅ 正确做法（只做被要求的）**

```python
def save_preferences(db, user_id: int, preferences: dict):
    """Save user preferences to database."""
    db.execute(
        "UPDATE users SET preferences = ? WHERE id = ?",
        (json.dumps(preferences), user_id)
    )
```

**以后需要再加**：缓存（性能成为问题时）、校验（出现脏数据时）、合并（需求出现时）。

---

## 3. 外科手术式改动（Surgical Changes）

### 示例 1：顺手重构

**用户请求**："修掉空邮箱导致校验器崩溃的 bug"

**❌ LLM 常见做法（改得太多）**

```diff
  def validate_user(user_data):
-     # Check email format
-     if not user_data.get('email'):
+     """Validate user data."""
+     email = user_data.get('email', '').strip()
+     
+     # Validate email
+     if not email:
          raise ValueError("Email required")
-     
-     # Basic email validation
-     if '@' not in user_data['email']:
+     if '@' not in email or '.' not in email.split('@')[1]:
          raise ValueError("Invalid email")
      
-     # Check username
-     if not user_data.get('username'):
+     # Validate username
+     username = user_data.get('username', '').strip()
+     if not username:
          raise ValueError("Username required")
+     if len(username) < 3:
+         raise ValueError("Username too short")
+     if not username.isalnum():
+         raise ValueError("Username must be alphanumeric")
      
      return True
```

**问题：**

- 在修 bug 之外"改进"了邮箱校验
- 加了没人要求的用户名校验
- 改了注释
- 加了 docstring

**✅ 正确做法（外科手术式）**

```diff
  def validate_user(user_data):
      # Check email format
-     if not user_data.get('email'):
+     email = user_data.get('email', '')
+     if not email or not email.strip():
          raise ValueError("Email required")
      
      # Basic email validation
-     if '@' not in user_data['email']:
+     if '@' not in email:
          raise ValueError("Invalid email")
      
      # Check username
      if not user_data.get('username'):
          raise ValueError("Username required")
      
      return True
```

**只改了**：修复空邮箱处理的那几行。

### 示例 2：风格漂移

**用户请求**："给上传函数加日志"

**❌ LLM 常见做法（全部重排）**

```diff
- def upload_file(file_path, destination):
+ def upload_file(file_path: str, destination: str) -> bool:
+     """Upload file to destination with logging."""
+     logger.info(f"Uploading {file_path} to {destination}")
+     
      try:
-         with open(file_path, 'rb') as f:
-             data = f.read()
-         
-         response = requests.post(destination, files={'file': data})
-         
-         if response.status_code == 200:
-             return True
-         else:
-             return False
+         with open(file_path, "rb") as f:
+             data = f.read()
+         response = requests.post(destination, files={"file": data})
+         success = response.status_code == 200
+         
+         if success:
+             logger.info(f"Upload successful: {file_path}")
+         else:
+             logger.error(f"Upload failed: {file_path}, status={response.status_code}")
+         
+         return success
      except Exception as e:
-         print(f"Error: {e}")
+         logger.exception(f"Upload error for {file_path}: {e}")
          return False
```

**问题：**

- 改了引号风格（'' → ""）
- 加了没人要求的类型标注
- 加了 docstring
- 重排了空白
- 改了布尔返回逻辑

**✅ 正确做法（遵循既有风格）**

```diff
+ import logging
+ 
+ logger = logging.getLogger(__name__)
+ 
  def upload_file(file_path, destination):
+     logger.info(f'Starting upload: {file_path}')
      try:
          with open(file_path, 'rb') as f:
              data = f.read()
          
          response = requests.post(destination, files={'file': data})
          
          if response.status_code == 200:
+             logger.info(f'Upload successful: {file_path}')
              return True
          else:
+             logger.error(f'Upload failed: {file_path}, status={response.status_code}')
              return False
      except Exception as e:
-         print(f"Error: {e}")
+         logger.exception(f'Upload error for {file_path}')
          return False
```

**保持了**：单引号、无类型标注、既有布尔模式、空白风格。

---

## 4. 目标驱动执行（Goal-Driven Execution）

### 示例 1：含糊 vs 可验证

**用户请求**："修复认证系统"

**❌ LLM 常见做法（含糊路径）**

```
我将这样修复认证系统：
1. 审查代码
2. 找出问题
3. 做出改进
4. 测试改动

[没有清晰的成功标准就开始改代码]
```

**✅ 正确做法（可验证目标）**

```
要"修复认证"，我需要先定义成功标准。
我们要解决的具体问题是什么？

如果问题是"改密码后用户仍保持登录"：

计划：
1. 写测试：改密码 → 验证旧会话失效
   验证：测试失败（复现了 bug）

2. 实现：改密码时使会话失效
   验证：测试通过

3. 检查边界：多个活跃会话、并发改密
   验证：补充测试通过

4. 验证无回归：既有认证测试全绿
   验证：完整测试套件通过

当前认证测试覆盖率：[展示覆盖率]
你遇到的具体认证问题是哪个？
```

### 示例 2：多步任务逐步验证

**用户请求**："给 API 加限流"

**❌ LLM 常见做法（一次全做）**

```python
# 一个 300 行的提交里实现了完整限流：Redis、多策略、
# 配置系统、监控，没有清晰的验证步骤
```

**✅ 正确做法（增量验证）**

```
限流计划：

1. 加基础内存限流（单端点）
   验证：
   - 测试：100 次请求 → 前 10 次成功，其余 429
   - 手动：curl 该端点 11 次，看到限流报错

2. 抽成中间件（应用到所有端点）
   验证：
   - 测试：限流对 /users 与 /posts 生效
   - 既有端点测试仍然通过

3. 加 Redis 后端（多服务器场景）
   验证：
   - 测试：限流计数在应用重启后保持
   - 测试：两个应用实例共享限流计数

4. 加配置（按端点配速率）
   验证：
   - 测试：/search 10 次/分，/users 100 次/分
   - 配置文件解析正确

每一步都可独立验证、独立上线。
从第 1 步开始？
```

### 示例 3：测试先行验证

**用户请求**："分数并列时排序会乱"

**❌ LLM 常见做法（不复现就修）**

```python
# 没确认 bug 就直接改排序逻辑
def sort_scores(scores):
    return sorted(scores, key=lambda x: (-x['score'], x['name']))
```

**✅ 正确做法（先复现）**

```python
# 1. 先写一个复现问题的测试
def test_sort_with_duplicate_scores():
    """Test sorting when multiple items have same score."""
    scores = [
        {'name': 'Alice', 'score': 100},
        {'name': 'Bob', 'score': 100},
        {'name': 'Charlie', 'score': 90},
    ]
    
    result = sort_scores(scores)
    
    # The bug: order is non-deterministic for duplicates
    # Run this test multiple times, it should be consistent
    assert result[0]['score'] == 100
    assert result[1]['score'] == 100
    assert result[2]['score'] == 90

# Verify: Run test 10 times → fails with inconsistent ordering

# 2. 现在用稳定排序修复
def sort_scores(scores):
    """Sort by score descending, then name ascending for ties."""
    return sorted(scores, key=lambda x: (-x['score'], x['name']))

# Verify: Test passes consistently
```

---

## 反模式速查

| 原则 | 反模式 | 修正 |
|---|---|---|
| 三思而后写 | 默默臆测文件格式、字段、范围 | 显式列出假设，请求澄清 |
| 简单优先 | 单个折扣计算上用策略模式 | 一个函数，直到真的需要复杂度 |
| 外科手术式改动 | 修 bug 时顺手改引号、加类型标注 | 只改修复报告问题所涉及的行 |
| 目标驱动 | "我会审查并改进代码" | "为 bug X 写测试 → 让它通过 → 验证无回归" |

## 关键洞见

"过度复杂"的示例并非一眼就错——它们遵循设计模式与最佳实践。问题出在**时机**：在需要之前就引入复杂度，会：

- 让代码更难理解
- 引入更多 bug
- 实现耗时更长
- 更难测试

"简单"的版本则：

- 更容易理解
- 实现更快
- 更容易测试
- 需要时可以再重构

**好代码是用简单方式解决今天问题的代码，不是超前解决明天问题的代码。**
