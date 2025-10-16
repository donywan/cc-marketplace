# Beautiful Code Plugin - 性能监控与日志记录

## 概述

本文档提供了代理系统的性能监控和日志记录实现指南，帮助代理自我监控、自我优化，并为系统运维提供数据支持。

## 性能监控实现

### 代理性能监控模板

每个代理都应包含以下性能监控功能：

#### 1. 执行时间监控
```markdown
## 性能监控 - 执行时间

### 监控指标
- **总执行时间**: 从接收到完成的总时长
- **各阶段耗时**: 思考/搜索/规划各阶段的时间分布
- **工具调用时间**: 各个工具的调用耗时
- **等待时间**: 等待外部响应的时间

### 监控实现
```python
# 伪代码示例
import time
import json

class PerformanceMonitor:
    def __init__(self, agent_name):
        self.agent_name = agent_name
        self.start_time = None
        self.phase_times = {}
        self.tool_calls = {}

    def start_monitoring(self):
        self.start_time = time.time()

    def start_phase(self, phase_name):
        self.phase_times[phase_name] = {"start": time.time()}

    def end_phase(self, phase_name):
        if phase_name in self.phase_times:
            self.phase_times[phase_name]["end"] = time.time()
            self.phase_times[phase_name]["duration"] = (
                self.phase_times[phase_name]["end"] -
                self.phase_times[phase_name]["start"]
            )

    def record_tool_call(self, tool_name, duration):
        if tool_name not in self.tool_calls:
            self.tool_calls[tool_name] = []
        self.tool_calls[tool_name].append(duration)

    def get_performance_report(self):
        total_time = time.time() - self.start_time
        return {
            "agent_name": self.agent_name,
            "total_execution_time_ms": total_time * 1000,
            "phase_breakdown": self.phase_times,
            "tool_call_stats": {
                tool: {
                    "call_count": len(calls),
                    "total_time_ms": sum(calls) * 1000,
                    "avg_time_ms": (sum(calls) / len(calls)) * 1000,
                    "max_time_ms": max(calls) * 1000,
                    "min_time_ms": min(calls) * 1000
                }
                for tool, calls in self.tool_calls.items()
            }
        }
```

#### 2. Token 使用量监控
```markdown
## Token 使用量监控

### 监控指标
- **输入 Token 数**: 用户输入和提示的 token 数量
- **输出 Token 数**: 生成内容的 token 数量
- **总 Token 数**: 输入和输出 token 的总和
- **Token 效率**: 每个 token 产生的有用信息量

### 监控实现
```python
class TokenMonitor:
    def __init__(self):
        self.input_tokens = 0
        self.output_tokens = 0
        self.tool_token_usage = {}

    def record_input_tokens(self, tokens, source="user"):
        self.input_tokens += tokens

    def record_output_tokens(self, tokens, component="response"):
        self.output_tokens += tokens

    def record_tool_tokens(self, tool_name, input_tokens, output_tokens):
        if tool_name not in self.tool_token_usage:
            self.tool_token_usage[tool_name] = {
                "input_tokens": 0,
                "output_tokens": 0
            }
        self.tool_token_usage[tool_name]["input_tokens"] += input_tokens
        self.tool_token_usage[tool_name]["output_tokens"] += output_tokens

    def get_token_report(self):
        return {
            "total_input_tokens": self.input_tokens,
            "total_output_tokens": self.output_tokens,
            "total_tokens": self.input_tokens + self.output_tokens,
            "efficiency_score": self.calculate_efficiency(),
            "tool_breakdown": self.tool_token_usage
        }

    def calculate_efficiency(self):
        # 简单的效率计算：输出token / 总token
        if self.input_tokens + self.output_tokens == 0:
            return 0
        return self.output_tokens / (self.input_tokens + self.output_tokens)
```

#### 3. 错误率监控
```markdown
## 错误率监控

### 监控指标
- **总请求数**: 处理的总请求数量
- **成功请求数**: 成功处理的请求数量
- **失败请求数**: 失败的请求数量
- **错误率**: 失败请求占总请求的百分比
- **错误类型分布**: 不同类型错误的统计

### 监控实现
```python
class ErrorMonitor:
    def __init__(self):
        self.total_requests = 0
        self.successful_requests = 0
        self.failed_requests = 0
        self.error_types = {}

    def record_request(self):
        self.total_requests += 1

    def record_success(self):
        self.successful_requests += 1

    def record_failure(self, error_type, error_message):
        self.failed_requests += 1
        if error_type not in self.error_types:
            self.error_types[error_type] = {
                "count": 0,
                "messages": []
            }
        self.error_types[error_type]["count"] += 1
        self.error_types[error_type]["messages"].append(error_message)

    def get_error_report(self):
        error_rate = (self.failed_requests / self.total_requests * 100) if self.total_requests > 0 else 0
        return {
            "total_requests": self.total_requests,
            "successful_requests": self.successful_requests,
            "failed_requests": self.failed_requests,
            "error_rate_percentage": error_rate,
            "error_breakdown": self.error_types
        }
```

## 日志记录实现

### 统一日志格式
```markdown
## 日志记录规范

### 日志格式标准
所有日志都应采用统一的 JSON 格式：

```json
{
  "timestamp": "2024-01-01T12:00:00.000Z",
  "level": "INFO|WARN|ERROR|DEBUG|CRITICAL",
  "agent": "think|search|code",
  "execution_id": "唯一执行标识符",
  "message": "日志消息内容",
  "context": {
    "phase": "当前执行阶段",
    "tool": "使用的工具（如适用）",
    "duration_ms": "操作耗时",
    "token_usage": "token使用情况",
    "error_details": "错误详情（如适用）",
    "additional_data": "其他相关数据"
  }
}
```

### 日志记录器实现
```python
import logging
import json
import uuid
from datetime import datetime

class AgentLogger:
    def __init__(self, agent_name):
        self.agent_name = agent_name
        self.execution_id = str(uuid.uuid4())
        self.logger = logging.getLogger(f"agent_{agent_name}")

    def _format_log(self, level, message, context=None):
        log_entry = {
            "timestamp": datetime.utcnow().isoformat() + "Z",
            "level": level,
            "agent": self.agent_name,
            "execution_id": self.execution_id,
            "message": message,
            "context": context or {}
        }
        return json.dumps(log_entry, ensure_ascii=False)

    def info(self, message, context=None):
        formatted_log = self._format_log("INFO", message, context)
        self.logger.info(formatted_log)

    def warn(self, message, context=None):
        formatted_log = self._format_log("WARN", message, context)
        self.logger.warning(formatted_log)

    def error(self, message, context=None):
        formatted_log = self._format_log("ERROR", message, context)
        self.logger.error(formatted_log)

    def debug(self, message, context=None):
        formatted_log = self._format_log("DEBUG", message, context)
        self.logger.debug(formatted_log)

    def critical(self, message, context=None):
        formatted_log = self._format_log("CRITICAL", message, context)
        self.logger.critical(formatted_log)

    def tool_call(self, tool_name, duration_ms, success=True, error=None):
        context = {
            "tool": tool_name,
            "duration_ms": duration_ms,
            "success": success
        }
        if error:
            context["error"] = str(error)

        level = "INFO" if success else "ERROR"
        message = f"Tool call: {tool_name}"
        self._log(level, message, context)
```

## 代理集成示例

### Think 代理监控集成
```markdown
## Think 代理监控集成

### 实现示例
```python
class ThinkAgentWithMonitoring:
    def __init__(self):
        self.agent_name = "think"
        self.performance_monitor = PerformanceMonitor(self.agent_name)
        self.token_monitor = TokenMonitor()
        self.error_monitor = ErrorMonitor()
        self.logger = AgentLogger(self.agent_name)

    async def process_request(self, user_query):
        try:
            self.performance_monitor.start_monitoring()
            self.error_monitor.record_request()
            self.logger.info("开始处理用户请求", {"query": user_query})

            # 阶段1：问题理解
            self.performance_monitor.start_phase("problem_understanding")
            thinking_start = time.time()

            # 使用 sequential-thinking 工具
            thinking_result = await self.sequential_thinking(user_query)
            thinking_duration = time.time() - thinking_start

            self.performance_monitor.record_tool_call("sequential-thinking", thinking_duration)
            self.performance_monitor.end_phase("problem_understanding")

            # 记录 token 使用
            self.token_monitor.record_input_tokens(len(user_query.split()))
            self.token_monitor.record_output_tokens(len(thinking_result.split()))

            self.logger.info("完成问题理解", {
                "phase": "problem_understanding",
                "duration_ms": thinking_duration * 1000
            })

            # 阶段2：知识记录
            self.performance_monitor.start_phase("knowledge_recording")
            await self.record_to_memory(thinking_result)
            self.performance_monitor.end_phase("knowledge_recording")

            # 生成性能报告
            perf_report = self.performance_monitor.get_performance_report()
            token_report = self.token_monitor.get_token_report()

            self.error_monitor.record_success()
            self.logger.info("请求处理成功", {
                "performance": perf_report,
                "token_usage": token_report
            })

            return {
                "thinking_result": thinking_result,
                "performance": perf_report,
                "token_usage": token_report
            }

        except Exception as e:
            self.error_monitor.record_failure(type(e).__name__, str(e))
            self.logger.error("请求处理失败", {
                "error_type": type(e).__name__,
                "error_message": str(e)
            })
            raise
```

## 监控数据使用

### 性能优化
1. **识别瓶颈**: 通过执行时间分析找出性能瓶颈
2. **资源优化**: 根据 token 使用情况优化提示设计
3. **错误预防**: 通过错误分析预防常见问题

### 运维监控
1. **健康检查**: 定期检查代理运行状态
2. **容量规划**: 根据使用量进行资源规划
3. **故障排查**: 利用日志快速定位问题

### 质量改进
1. **趋势分析**: 分析性能指标的变化趋势
2. **对比分析**: 对比不同配置的性能表现
3. **A/B 测试**: 测试改进措施的效果

## 配置管理

### 监控配置
```yaml
monitoring:
  enabled: true
  log_level: INFO
  performance_tracking:
    enabled: true
    detailed_timing: true
    tool_call_tracking: true
  token_tracking:
    enabled: true
    efficiency_analysis: true
  error_tracking:
    enabled: true
    error_messages: true
    stack_traces: true
  reporting:
    interval_minutes: 60
    retention_days: 30
    export_format: json
```

通过实施这些监控和日志记录机制，代理系统可以更好地自我监控、自我优化，并为系统运维提供宝贵的数据支持。