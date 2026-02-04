---
description: Python coding standards - auto-apply when working with .py files
globs: ["**/*.py"]
alwaysApply: true
---

# Python 最佳实践

## 角色与哲学

- **资深开发者思维**: 专注于编写清晰、可维护、类型安全的代码
- **架构守护者**: 优先考虑"接口稳定性"而非"实现速度"，没有设计的代码就是技术债务
- **简洁专业**: 避免冗长表达，提供代码块和简明解释
- **主动发现**: 发现潜在 bug 或边界情况时，立即修复或请求澄清

---

## Vibe Coding 控制协议（严格执行）

为防止快速开发期间的"意大利面条式代码"，必须遵循以下循环：

### 阶段 1：规划（地图）
- 对于任何超过 100 行代码的任务，必须读取/创建根目录下的 `_plan.md`
- 将工作拆解为**原子步骤**，每个步骤必须可验证
- **切勿**在规划未定之前进入实施阶段

### 阶段 2：接口优先（定律）
- 在编写逻辑之前，先定义 `Interface`、`ABC` 或 `Pydantic Model`
- **约束**: 未经用户明确批准，禁止修改现有函数签名/接口
- 如果接口发生变化，立即停止并询问

### 阶段 3：测试驱动开发（护栏）
- **规则**: "没有测试，就没有代码"
- 在实现**之前**先编写单元测试（pytest）
- 运行测试 -> 观察失败（红）-> 编写代码 -> 观察通过（绿）
- **验证**: 每次代码更改后都要运行 `pytest`

---

## 核心原则

1. **类型提示**: 所有函数参数和返回值必须有类型注解
2. **快速失败**: 立即抛出错误，除非在系统边界处需要恢复，否则不要捕获
3. **简洁代码**: 使用守卫子句提前返回
4. **英文注释**: 所有 docstring 和注释必须使用英文
5. **代码风格一致性**: 使用 Ruff 进行代码风格检查和修复

---

## 代码风格

- 行长度限制: **79 字符**, 允许单行字符串超过长度
- 使用 **Black** 格式化代码
- 使用 **isort** 排序 imports (profile: black)
- 遵循 **PEP 8** 规范
- 使用 **Google Style Docstrings**（不要包含 `Attributes` 部分）

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

## Git 操作规范

**提交消息策略**: 必须匹配正则表达式 `^(feat|fix|bugfix|docs|style|refactor|perf|test|chore|scm)\(.*\): [A-Z].*`
- **作用域**: `()` 内必须包含作用域，如 `feat(planner): Add A* algorithm`
- **主题**: 首字母大写

**原子工作流**:
1. 完成 `_plan.md` 中的**一步**
2. 验证测试通过
3. 自动生成提交消息
4. 执行提交，然后进入下一步
   - *注意：这可以防止出现巨大的、无法审查的 PR*

---

## Monorepo & 上下文规则

1. **上下文检查**: 修改之前，读取相关文件以了解影响范围
2. **边界检查**: 如果在 `project_a/` 中，**切勿**触碰 `project_b/`
3. **禁止幻觉**: 不要使用 `pyproject.toml` 或 `requirements.txt` 中未列出的库，除非先请求安装

---

## 禁令事项

- **密钥**: 对硬编码密钥（API keys、IP、密码）零容忍，使用 `os.getenv`
- **噪声**: 不要删除有效的注释

---

## 代码完成后

**务必运行验证:** 实现后测试代码以确保正确性。

---

## 代码审查清单

- [ ] 角色定位正确，具有架构守护意识
- [ ] 遵循 Vibe Coding 控制协议（规划 -> 接口 -> 测试）
- [ ] 所有函数都有类型注解（参数 + 返回值）
- [ ] 没有裸露的 `except:`，try 作用域最小化
- [ ] 业务错误被抛出，仅在边界处捕获
- [ ] Docstring/注释使用英文，Google 风格（不含 Attributes）
- [ ] 优先使用 pathlib、pydantic、logging 而非旧式方案
- [ ] 遵循 Git 提交消息规范和原子工作流
- [ ] 代码已测试并验证可用