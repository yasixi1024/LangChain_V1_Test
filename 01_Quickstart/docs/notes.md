# 项目环境配置与问题排查笔记

## 1. 通过 Anaconda Navigator 选择 Conda 虚拟环境

在 Anaconda Navigator 中打开终端或启动开发工具前，需要先选择项目对应的 Conda 虚拟环境，否则默认使用的是 `base` 环境。

## 2. 在 PyCharm 中创建 Conda 虚拟环境

可以直接在 PyCharm 的可视化界面中创建 Conda 虚拟环境，无需通过命令行执行 `conda create`。

### 2.1 创建步骤

1. 按 `⌘ ,` 打开设置，搜索 **Python Interpreter／Python 解释器**。
2. 点击 **Add Interpreter → Add Local Interpreter**。
3. 环境类型选择 **Conda**。
4. 选择 **Generate new／New environment（创建新环境）**。
5. 填写以下信息：
   - **Python version**：项目要求的版本，例如 `3.12`。
   - **Environment name**：例如 `knowledge-base`。
   - **Conda executable**：通常会自动识别；如果没有，需要选择本机的 `conda` 可执行文件。
6. 点击 **OK／Create**，等待创建和安装完成。

## 3. 运行 agent.py 时出现依赖版本不兼容

### 3.1 报错现象

在项目根目录执行 `python agent.py` 时，如果出现以下错误，说明 `langgraph` 与 `langgraph-prebuilt` 版本不兼容：

```text
Traceback (most recent call last):
  File "/Users/yxi/yxi-code/git-yasi/LangChain_V1_Test/01_Quickstart/agent.py", line 4, in <module>
    from langchain.agents import create_agent
  File "/opt/anaconda3/envs/01-quickstart/lib/python3.11/site-packages/langchain/agents/__init__.py", line 3, in <module>
    from langchain.agents.factory import create_agent
  File "/opt/anaconda3/envs/01-quickstart/lib/python3.11/site-packages/langchain/agents/factory.py", line 22, in <module>
    from langgraph.prebuilt.tool_node import ToolCallWithContext, ToolNode
  File "/opt/anaconda3/envs/01-quickstart/lib/python3.11/site-packages/langgraph/prebuilt/__init__.py", line 3, in <module>
    from langgraph.prebuilt.chat_agent_executor import create_react_agent
  File "/opt/anaconda3/envs/01-quickstart/lib/python3.11/site-packages/langgraph/prebuilt/chat_agent_executor.py", line 47, in <module>
    from langgraph.prebuilt.tool_node import ToolCallWithContext, ToolNode
  File "/opt/anaconda3/envs/01-quickstart/lib/python3.11/site-packages/langgraph/prebuilt/tool_node.py", line 89, in <module>
    from langgraph.runtime import ExecutionInfo, ServerInfo  # noqa: TC002
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
ImportError: cannot import name 'ExecutionInfo' from 'langgraph.runtime' (/opt/anaconda3/envs/01-quickstart/lib/python3.11/site-packages/langgraph/runtime.py)
```

### 3.2 问题原因

`langgraph-prebuilt` 在导入时使用了 `langgraph.runtime.ExecutionInfo`，但当前安装的 `langgraph` 版本没有提供该类型，因此在导入阶段失败。

### 3.3 解决方式

在项目对应的 Conda 环境中，保留教程使用的 `langchain==1.2.1`，将 `langgraph-prebuilt` 降到 `1.0.8`，并固定 `langgraph==1.0.10`：

```bash
python -m pip install "langchain==1.2.1" "langgraph==1.0.10" "langgraph-prebuilt==1.0.8"
```

### 3.4 验证修复结果

在项目根目录依次执行：

```bash
python -m pip check
python -c "from langchain.agents import create_agent; print('导入成功')"
python agent.py
```

`pip check` 用于检查依赖声明是否满足；导入验证用于确认本次 `ImportError` 是否解决；最后运行脚本验证完整流程。

注意：即使 `pip check` 通过，也不能单独证明代码层面的兼容性，因此还需要执行导入验证。
