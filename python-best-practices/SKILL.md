---
description: Python coding standards - auto-apply when working with .py files
globs: ["**/*.py"]
alwaysApply: true
---

# Python 最佳实践

在处理 Python 代码时，优先遵循仓库现有约定；如果仓库没有明确约定，再使用下面这些默认规则。

## 优先级

- 修改前先读取相关文件和调用点，确认影响范围后再动手
- 将改动控制在当前任务相关的模块和目录内，避免顺手修改无关部分
- 先检查项目已有模式与配置，例如 `pyproject.toml`、`ruff.toml`、测试目录和现有模块结构
- 优先写清晰、可维护、类型明确的代码，避免为了抽象而抽象
- 发现明显 bug、遗漏的边界条件或不一致行为时，直接修复，或在风险较高时明确说明

## 核心原则

- 为新增或修改的函数补全参数和返回值类型；无返回值时显式写 `-> None`
- 使用守卫子句减少嵌套，让失败路径尽早返回
- 默认快速失败，不要吞掉异常；只在外部 IO、库边界或需要转换错误语义时捕获异常，并将 `try` 范围缩到最小
- 注释和 docstring 使用英文，只写对读者有帮助的信息，不复述代码表面含义
- 避免使用裸 `except:`；只有在确有必要时才使用 `except Exception:`，并说明原因
- 不要为了“清理代码”而删除仍然提供有效上下文的注释、文档或错误信息
- 优先复用现有接口与模块边界；如果需要修改公共接口，先评估影响范围并说明
- 不要引入 `pyproject.toml`、`requirements.txt` 或现有代码中未声明的新依赖，除非先得到明确许可
- 对硬编码密钥、密码、令牌和私有地址零容忍；优先使用环境变量或安全配置，例如 `os.getenv`

## 风格默认值

- 如果仓库没有别的规定，遵循 PEP 8 和 `snake_case`
- 优先使用项目已配置的格式化和 lint 工具；如果没有明确工具链，`ruff check` 和 `ruff format` 是合理默认值
- 如果项目 Python 版本允许，优先使用内建泛型，如 `list[str]`、`dict[str, int]`
- Docstring 风格跟随仓库；如果仓库没有统一约定，使用简洁的 Google 风格

```python
def fetch_orders(user_id: str, limit: int = 100) -> list[Order]:
    """Fetch orders for a user.

    Args:
        user_id: Unique identifier of the user.
        limit: Max orders to return.

    Returns:
        Orders owned by the user.
    """
```

## 测试与验证

- 在修复 bug、调整行为或新增逻辑时，补充或更新最相关的测试
- 优先运行最小必要验证，例如单个测试文件、相关测试用例或针对性 lint 命令
- 不强制所有任务都先写测试，但提交前应确认改动已被合理验证

## 简短示例

```python
def get_user(user_id: str) -> User:
    if not user_id:
        raise ValueError("user_id is required")

    user = repository.get(user_id)
    if user is None:
        raise UserNotFoundError(user_id)
    return user
```

这个例子体现了类型标注、守卫子句和快速失败。
