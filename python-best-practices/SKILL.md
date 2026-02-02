---
description: Python coding standards - auto-apply when working with .py files
globs: ["**/*.py"]
alwaysApply: true
---

# Python 最佳实践

## 核心原则

1. **类型提示**: 所有函数参数和返回值必须有类型注解
2. **快速失败**: 立即抛出错误，除非在系统边界处需要恢复，否则不要捕获
3. **简洁代码**: 使用守卫子句提前返回
4. **英文注释**: 所有 docstring 和注释必须使用英文
5. **代码风格一致性**: 使用 Ruff 进行代码风格检查和修复

---

## 代码风格

- 行长度限制: **79 字符**
- 使用 **Black** 格式化代码
- 使用 **isort** 排序 imports (profile: black)

---

## 文件命名

- 使用 `snake_case`: `user_service.py`, `data_utils.py`
- 测试文件: `test_<module>.py`

---

## 类型注解

```python
# ✅ Good
def get_user(user_id: str) -> User:
    user = db.get(user_id)
    if user is None:
        raise UserNotFoundError(user_id)
    return user
```

- 无返回值的函数使用 `-> None`
- 优先使用 `list`, `dict` 而非 `List`, `Dict` (Python 3.9+)

---

## 错误处理

**快速失败 - 不要吞掉错误:**

```python
# ❌ Bad: hides bugs
def get_user(user_id: str) -> User | None:
    try:
        return db.get(user_id)
    except Exception:
        return None

# ✅ Good: let errors propagate and expose the problem directly
def get_user(user_id: str) -> User:
    user = db.get(user_id)
    if user is None:
        raise UserNotFoundError(user_id)  # Direct exposure
    return user
```

**除非绝对必要，否则避免使用 `try`, 保持 `try` 的作用域尽可能小:**

- ✅ **不可避免的外部 IO**: 网络请求、文件系统访问、数据库事务
- ✅ **库边界**: 当第三方库可能抛出可恢复的意外错误时
- ❌ **业务逻辑**: 不要用 `try` 包装业务规则，让异常向上冒泡，在 API/UI 边界处理
- ❌ **开发/测试**: 不要在测试脚本或 `__main__` 块中捕获错误

---

## 函数设计

**使用守卫子句提前返回:**

```python
# ❌ Bad: nested
def process(order: Order) -> Result:
    if order is not None:
        if order.items:
            total = sum(i.price for i in order.items)
            if total > 0:
                return Result(success=True, total=total)
    return Result(success=False)

# ✅ Good: guard clauses
def process(order: Order) -> Result:
    if order is None:
        return Result.failure("Invalid")
    if not order.items:
        return Result.failure("No items")
    
    return Result.success(total=_calc_total(order.items))
```

**命名规范:**
- 动作: `get_`, `create_`, `update_`, `delete_`
- 布尔: `is_valid()`, `has_permission()`, `can_edit()`

---

## 文档（仅限英文）

**Docstring - Google 风格:**

```python
def fetch_orders(user_id: str, *, limit: int = 100) -> list[Order]:
    """Fetch orders for a user.

    Args:
        user_id: Unique identifier of the user.
        limit: Max orders to return.

    Returns:
        List of Order objects.

    """
```

**注释 - 只在关键处添加, 避免冗余:**

```python
# ❌ Bad
# Check if admin
if user.role == "admin":

# ✅ Good
# Bypass rate limit for health checks
if request.source == "internal":
```

---

## 设计模式

**Dataclass / Pydantic:**

```python
@dataclass
class User:
    id: str
    name: str
    email: str

class UserConfig(BaseModel):
    name: str = Field(..., min_length=1)
    retries: int = Field(default=3, ge=1, le=10)
```

**Protocol (接口):**

```python
class Repository(Protocol):
    def get(self, id: str) -> Entity | None: ...
    def save(self, entity: Entity) -> None: ...
```

**依赖注入:**

```python
class UserService:
    def __init__(self, repo: UserRepository, cache: Cache) -> None:
        self._repo = repo
        self._cache = cache
```

**工厂模式:**

```python
class SerializerFactory:
    _registry: ClassVar[dict[str, type[Serializer]]] = {}

    @classmethod
    def register(cls, name: str, serializer: type[Serializer]) -> None:
        cls._registry[name] = serializer

    @classmethod
    def create(cls, name: str) -> Serializer:
        return cls._registry[name]()
```

**策略模式:**

```python
class PricingStrategy(Protocol):
    def calculate(self, price: Decimal, qty: int) -> Decimal: ...

class Order:
    def __init__(self, pricing: PricingStrategy) -> None:
        self._pricing = pricing
```

**自定义异常:**

```python
class AppError(Exception):
    """Base application error."""

class UserNotFoundError(AppError):
    def __init__(self, user_id: str) -> None:
        self.user_id = user_id
        super().__init__(f"User not found: {user_id}")
```

---

## 代码完成后

**务必运行验证:** 实现后测试代码以确保正确性。

---

## 代码审查清单

- [ ] 所有函数都有类型注解（参数 + 返回值）
- [ ] 没有裸露的 `except:`，try 作用域最小化
- [ ] 业务错误被抛出，仅在边界处捕获
- [ ] Docstring/注释使用英文，Google 风格
- [ ] 代码已测试并验证可用
