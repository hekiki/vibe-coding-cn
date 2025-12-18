# 实战案例：命令行工具

> 难度：⭐⭐ 中等 | 预计时间：1-2 小时 | 技术栈：Python + Click

## 🎯 项目目标

构建一个实用的命令行工具，实现：
- 文件批量重命名
- 支持多种命名规则
- 预览和确认机制
- 彩色输出

## 📋 开始前的准备

### 环境要求
- Python 3.8+
- pip

### 第一步：需求澄清

复制以下提示词给 AI：

```
我想用 Vibe Coding 的方式开发一个文件批量重命名的命令行工具。

技术要求：
- 语言：Python 3.8+
- CLI 框架：Click
- 输出美化：Rich

功能需求：
1. 支持多种重命名规则：
   - 添加前缀/后缀
   - 替换文本
   - 序号命名
   - 日期命名
2. 预览模式（不实际执行）
3. 递归处理子目录（可选）
4. 支持文件类型过滤

请帮我：
1. 确认技术栈是否合适
2. 生成项目结构
3. 一步步指导我完成开发

要求：每完成一步问我是否成功，再继续下一步。
```

## 🏗️ 项目结构

```
file-renamer/
├── renamer/
│   ├── __init__.py
│   ├── cli.py              # CLI 入口
│   ├── core.py             # 核心逻辑
│   └── rules.py            # 重命名规则
├── tests/
│   └── test_core.py
├── pyproject.toml
└── README.md
```

## 🔧 核心代码

### CLI 入口 (renamer/cli.py)

```python
import click
from rich.console import Console
from rich.table import Table
from .core import FileRenamer
from .rules import PrefixRule, SuffixRule, ReplaceRule, SequenceRule

console = Console()

@click.group()
@click.version_option(version='1.0.0')
def cli():
    """文件批量重命名工具"""
    pass

@cli.command()
@click.argument('directory', type=click.Path(exists=True))
@click.option('--prefix', '-p', help='添加前缀')
@click.option('--suffix', '-s', help='添加后缀')
@click.option('--replace', '-r', nargs=2, help='替换文本 (旧文本 新文本)')
@click.option('--sequence', '-n', is_flag=True, help='序号命名')
@click.option('--ext', '-e', multiple=True, help='文件扩展名过滤')
@click.option('--recursive', '-R', is_flag=True, help='递归处理子目录')
@click.option('--dry-run', '-d', is_flag=True, help='预览模式')
def rename(directory, prefix, suffix, replace, sequence, ext, recursive, dry_run):
    """批量重命名文件"""
    renamer = FileRenamer(directory, recursive=recursive, extensions=ext)
    
    # 添加规则
    if prefix:
        renamer.add_rule(PrefixRule(prefix))
    if suffix:
        renamer.add_rule(SuffixRule(suffix))
    if replace:
        renamer.add_rule(ReplaceRule(replace[0], replace[1]))
    if sequence:
        renamer.add_rule(SequenceRule())
    
    if not renamer.rules:
        console.print("[red]错误：请至少指定一个重命名规则[/red]")
        return
    
    # 获取预览
    changes = renamer.preview()
    
    if not changes:
        console.print("[yellow]没有找到匹配的文件[/yellow]")
        return
    
    # 显示预览表格
    table = Table(title="重命名预览")
    table.add_column("原文件名", style="cyan")
    table.add_column("新文件名", style="green")
    
    for old, new in changes:
        table.add_row(old, new)
    
    console.print(table)
    console.print(f"\n共 [bold]{len(changes)}[/bold] 个文件")
    
    if dry_run:
        console.print("[yellow]预览模式，未执行实际操作[/yellow]")
        return
    
    # 确认执行
    if click.confirm('确认执行重命名？'):
        renamer.execute()
        console.print("[green]✓ 重命名完成！[/green]")
    else:
        console.print("[yellow]已取消[/yellow]")

if __name__ == '__main__':
    cli()
```

### 核心逻辑 (renamer/core.py)

```python
import os
from pathlib import Path
from typing import List, Tuple

class FileRenamer:
    def __init__(self, directory: str, recursive: bool = False, extensions: tuple = ()):
        self.directory = Path(directory)
        self.recursive = recursive
        self.extensions = extensions
        self.rules = []
    
    def add_rule(self, rule):
        self.rules.append(rule)
    
    def get_files(self) -> List[Path]:
        pattern = '**/*' if self.recursive else '*'
        files = []
        for path in self.directory.glob(pattern):
            if path.is_file():
                if self.extensions:
                    if path.suffix.lower() in [f'.{e.lower()}' for e in self.extensions]:
                        files.append(path)
                else:
                    files.append(path)
        return sorted(files)
    
    def apply_rules(self, filename: str) -> str:
        result = filename
        for rule in self.rules:
            result = rule.apply(result)
        return result
    
    def preview(self) -> List[Tuple[str, str]]:
        changes = []
        for file in self.get_files():
            old_name = file.stem
            new_name = self.apply_rules(old_name)
            if old_name != new_name:
                changes.append((file.name, new_name + file.suffix))
        return changes
    
    def execute(self):
        for file in self.get_files():
            old_name = file.stem
            new_name = self.apply_rules(old_name)
            if old_name != new_name:
                new_path = file.parent / (new_name + file.suffix)
                file.rename(new_path)
```

### 重命名规则 (renamer/rules.py)

```python
from abc import ABC, abstractmethod

class Rule(ABC):
    @abstractmethod
    def apply(self, filename: str) -> str:
        pass

class PrefixRule(Rule):
    def __init__(self, prefix: str):
        self.prefix = prefix
    
    def apply(self, filename: str) -> str:
        return f"{self.prefix}{filename}"

class SuffixRule(Rule):
    def __init__(self, suffix: str):
        self.suffix = suffix
    
    def apply(self, filename: str) -> str:
        return f"{filename}{self.suffix}"

class ReplaceRule(Rule):
    def __init__(self, old: str, new: str):
        self.old = old
        self.new = new
    
    def apply(self, filename: str) -> str:
        return filename.replace(self.old, self.new)

class SequenceRule(Rule):
    def __init__(self, start: int = 1, padding: int = 3):
        self.counter = start
        self.padding = padding
    
    def apply(self, filename: str) -> str:
        result = f"{str(self.counter).zfill(self.padding)}_{filename}"
        self.counter += 1
        return result
```

## 📦 安装配置 (pyproject.toml)

```toml
[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"

[project]
name = "file-renamer"
version = "1.0.0"
description = "文件批量重命名工具"
requires-python = ">=3.8"
dependencies = [
    "click>=8.0",
    "rich>=13.0",
]

[project.scripts]
renamer = "renamer.cli:cli"
```

## 🚀 使用示例

```bash
# 安装
pip install -e .

# 添加前缀
renamer rename ./photos --prefix "2024_"

# 替换文本
renamer rename ./docs --replace "old" "new"

# 序号命名 + 过滤扩展名
renamer rename ./images --sequence --ext jpg --ext png

# 预览模式
renamer rename ./files --prefix "backup_" --dry-run

# 递归处理
renamer rename ./project --suffix "_v2" --recursive
```

## ✅ 验收清单

- [ ] 命令行帮助信息正确显示
- [ ] 前缀/后缀功能正常
- [ ] 替换功能正常
- [ ] 序号命名功能正常
- [ ] 预览模式不执行实际操作
- [ ] 扩展名过滤正常
- [ ] 递归处理正常

## 💡 进阶挑战

- 添加撤销功能（记录操作日志）
- 支持正则表达式
- 添加日期格式化规则
- 支持配置文件
- 添加交互式模式
