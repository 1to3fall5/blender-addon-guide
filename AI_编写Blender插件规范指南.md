# AI编写Blender插件规范指南

## 目录

- [基本概念](#基本概念)
- [开发环境设置](#开发环境设置)
- [插件结构](#插件结构)
- [命名规范](#命名规范)
- [文件夹组织规范](#文件夹组织规范)
- [关键组件](#关键组件)
- [API使用规范](#api使用规范)
- [界面设计](#界面设计)
- [UI设计可视化](#ui设计可视化)
- [性能优化](#性能优化)
- [测试与调试](#测试与调试)
- [文档规范](#文档规范)
- [代码移植与重构](#代码移植与重构)
- [发布与分发](#发布与分发)
- [常见问题解决](#常见问题解决)
- [示例模板](#示例模板)

## 基本概念

### Blender插件概述

Blender插件（Add-on）是扩展Blender功能的Python脚本集合。AI在编写Blender插件时需要了解以下基本概念：

- **Blender版本兼容性**：明确支持的Blender版本范围，通常在`bl_info`字典中定义
- **Python版本**：Blender内置的Python版本（如Blender 2.93使用Python 3.9）
- **插件类型**：工具类（Tool）、物体生成类（Object）、材质类（Material）等
- **API层级**：bpy、bmesh、bgl等不同模块的作用域和用途

### 插件分类

- **官方插件**：Blender自带的插件
- **社区插件**：第三方开发者创建的插件
- **商业插件**：付费使用的高级功能插件

## 开发环境设置

### 推荐的开发环境

- **编辑器/IDE**：Visual Studio Code + Blender开发插件
- **调试工具**：Blender自带的Python控制台、Visual Studio Code的调试器
- **版本控制**：Git（推荐使用.gitignore排除__pycache__等文件）

### 开发环境配置步骤

1. **安装Blender**：下载并安装目标版本的Blender
2. **设置外部编辑器**：配置Blender使用外部编辑器编写代码
3. **配置插件路径**：找到Blender的插件目录（通常在`用户/AppData/Roaming/Blender Foundation/Blender/版本号/scripts/addons`）
4. **启用开发者选项**：在Blender首选项中启用插件开发相关选项

## 插件结构

### 基本文件结构

```
my_addon/
│
├── __init__.py       # 主要插件文件，包含bl_info和注册/注销函数
├── operators.py      # 操作类定义
├── panels.py         # 界面面板定义
├── properties.py     # 属性定义
├── functions.py      # 核心功能实现
│
├── icons/            # 图标资源
├── presets/          # 预设文件
└── lib/              # 第三方库或依赖
```

### 必要组件

- **bl_info**：插件元数据字典，定义名称、版本、作者等信息
- **类注册和注销**：`register()`和`unregister()`函数
- **操作符（Operator）**：继承自`bpy.types.Operator`的功能类
- **面板（Panel）**：继承自`bpy.types.Panel`的界面类

### bl_info字典示例

```python
bl_info = {
    "name": "插件名称",
    "author": "作者名",
    "version": (1, 0, 0),
    "blender": (2, 93, 0),
    "location": "视图3D > 侧边栏 > 我的插件",
    "description": "插件功能描述",
    "warning": "警告信息",
    "doc_url": "文档URL",
    "category": "插件类别",
}
```

## 命名规范

### 插件命名

- **插件文件夹名**：使用小写字母和下划线，如`my_addon`、`material_library`
- **插件包名**：与文件夹名一致，遵循Python包命名规则
- **插件显示名**：在`bl_info`中定义，可使用空格和大写，如"Material Library"

### 类命名

- **操作符类**：使用`大写字母_OT_功能名`格式，如`OBJECT_OT_add_cube`
  - `OT`表示Operator Type
  - 前缀表示所属类别（如`OBJECT`、`MESH`、`MATERIAL`）
- **面板类**：使用`空间类型_PT_功能名`格式，如`VIEW3D_PT_tools_panel`
  - `PT`表示Panel Type
  - 前缀表示所属空间（如`VIEW3D`、`PROPERTIES`、`NODE_EDITOR`）
- **属性组类**：使用大驼峰命名法（PascalCase），如`MyAddonProperties`
- **其他工具类**：使用大驼峰命名法，如`MeshGenerator`、`TextureManager`

### 标识符命名

- **操作符ID**：使用`类别.操作名`格式，如`object.add_cube`、`mesh.remove_doubles`
  - 类别部分使用小写，表示操作的范围
  - 操作名使用小写和下划线，清晰描述功能
- **属性ID**：使用小写下划线命名法（snake_case），如`my_float_value`、`use_smooth_shading`
- **枚举值**：使用大写下划线格式，如`MODE_EDIT`、`DISPLAY_WIREFRAME`

### 函数和变量命名

- **函数名**：使用小写下划线命名法，如`create_mesh()`、`apply_material()`
- **私有函数**：使用前导下划线，如`_internal_process()`
- **变量名**：使用小写下划线命名法，如`vertex_count`、`material_index`
- **常量**：使用全大写下划线格式，如`MAX_VERTICES`、`DEFAULT_COLOR`

### 命名最佳实践

- 名称应具有描述性，清楚表明用途
- 避免使用缩写，除非是广泛接受的（如UI、ID）
- 保持与Blender内置命名风格的一致性
- 在命名中避免使用Blender关键字，防止冲突

```python
# 命名示例
class OBJECT_OT_add_special_cube(bpy.types.Operator):
    bl_idname = "object.add_special_cube"
    bl_label = "添加特殊立方体"
    
    cube_size: bpy.props.FloatProperty(name="立方体大小")
    material_type: bpy.props.EnumProperty(
        items=[
            ('BASIC', "基础", "基础材质"),
            ('ADVANCED', "高级", "高级材质")
        ]
    )
    
    def execute(self, context):
        # 变量命名示例
        vertex_count = 8
        DEFAULT_COLOR = (1.0, 1.0, 1.0)
        return {'FINISHED'}
```

## 文件夹组织规范

### 插件根目录组织

- 保持根目录简洁，只包含必要的Python模块和资源目录
- 使用标准化的目录结构，便于理解和维护
- 模块化组织代码，按功能拆分为多个Python文件

### 标准目录结构

```
my_addon/
│
├── __init__.py         # 插件入口，包含bl_info和注册函数
├── operators/          # 操作符实现目录
│   ├── __init__.py     # 导入所有操作符
│   ├── create.py       # 创建类操作符
│   └── modify.py       # 修改类操作符
│
├── ui/                 # 界面相关目录
│   ├── __init__.py     # 导入所有UI元素
│   ├── panels.py       # 面板定义
│   └── menus.py        # 菜单定义
│
├── properties/         # 属性定义目录
│   ├── __init__.py
│   └── properties.py   # 属性和属性组定义
│
├── core/               # 核心功能目录
│   ├── __init__.py
│   ├── utils.py        # 实用工具函数
│   └── algorithms.py   # 核心算法实现
│
├── presets/            # 预设目录
│   └── default.py      # 默认预设
│
├── resources/          # 资源目录
│   ├── icons/          # 图标资源
│   ├── templates/      # 模板文件
│   └── config.json     # 配置文件
│
└── lib/                # 第三方库目录
    └── external_module/  # 外部依赖模块
```

### 资源文件组织

- **图标资源**：放置在`resources/icons/`目录，使用统一的格式（建议PNG）
- **预设文件**：放置在`presets/`目录，便于用户选择不同配置
- **配置文件**：如需持久化配置，使用JSON或YAML格式存储在`resources/`目录
- **临时文件**：在操作系统临时目录创建和管理，不要放在插件目录内

### 第三方库处理

- **内置库**：直接放置在`lib/`目录下，确保相对导入路径正确
- **纯Python库**：可直接包含在插件包中
- **带编译组件的库**：需要为不同操作系统提供预编译版本，或在安装时编译
- **常用库处理**：

```python
# 第三方库导入示例
try:
    # 先尝试从标准路径导入
    import numpy as np
except ImportError:
    # 如果失败，从插件lib目录导入
    import os
    import sys
    lib_path = os.path.join(os.path.dirname(__file__), "lib")
    if lib_path not in sys.path:
        sys.path.append(lib_path)
    import numpy as np
```

### 多文件模块组织

- 使用`__init__.py`聚合子模块中的类和函数
- 通过命名空间组织代码，避免导入时的命名冲突
- 示例导入结构：

```python
# __init__.py 示例
from .operators.create import OBJECT_OT_add_special_cube
from .operators.modify import OBJECT_OT_modify_special_cube
from .ui.panels import VIEW3D_PT_special_tools
from .properties.properties import SpecialProperties

# 统一注册所有类
classes = (
    OBJECT_OT_add_special_cube,
    OBJECT_OT_modify_special_cube,
    VIEW3D_PT_special_tools,
    SpecialProperties
)

def register():
    from bpy.utils import register_class
    for cls in classes:
        register_class(cls)

def unregister():
    from bpy.utils import unregister_class
    for cls in reversed(classes):
        unregister_class(cls)
```

### 配置文件使用

- 使用配置文件存储默认设置和用户偏好
- 支持配置文件的保存和加载
- 示例配置管理：

```python
import os
import json

def get_config_path():
    """获取配置文件路径"""
    addon_dir = os.path.dirname(os.path.realpath(__file__))
    return os.path.join(addon_dir, "resources", "config.json")

def load_config():
    """加载配置"""
    config_path = get_config_path()
    if os.path.exists(config_path):
        with open(config_path, 'r') as f:
            return json.load(f)
    return {} # 默认配置

def save_config(config):
    """保存配置"""
    config_path = get_config_path()
    os.makedirs(os.path.dirname(config_path), exist_ok=True)
    with open(config_path, 'w') as f:
        json.dump(config, f, indent=4)
```