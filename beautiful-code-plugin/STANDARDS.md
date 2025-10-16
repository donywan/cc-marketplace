# Beautiful Code Plugin - 代理协作标准

## 概述

本文档定义了三个代理（think、search、code）之间的统一数据交换格式、输出标准和协作规范，确保代理之间能够高效、可靠地协作。

## 数据交换格式

### 通用响应格式

所有代理的输出都应遵循以下标准格式：

```json
{
  "metadata": {
    "agent_name": "代理名称",
    "timestamp": "ISO 8601 时间戳",
    "version": "标准版本号",
    "execution_time_ms": "执行时间（毫秒）",
    "token_usage": {
      "input_tokens": "输入token数",
      "output_tokens": "输出token数",
      "total_tokens": "总token数"
    },
    "status": "success|partial_success|error",
    "error_details": "错误详情（仅在status为error时）"
  },
  "data": {
    "核心数据内容": "根据代理类型定义的具体数据结构"
  },
  "collaboration": {
    "upstream_references": ["上游代理输出引用"],
    "downstream_guidance": "给下游代理的指导",
    "memory_entities_created": ["创建的memory实体"],
    "quality_score": "输出质量评分（0-10）"
  }
}
```

### Think 代理输出格式

```json
{
  "metadata": { ... },
  "data": {
    "problem_analysis": {
      "original_query": "用户原始问题",
      "core_problem": "识别的核心问题",
      "real_needs": ["真实需求列表"],
      "constraints": ["约束条件列表"]
    },
    "thinking_process": {
      "analysis_steps": ["分析步骤"],
      "key_insights": ["关键洞察"],
      "logical_flow": "逻辑流程描述"
    },
    "search_guidance": {
      "primary_keywords": ["主要搜索关键词"],
      "secondary_keywords": ["次要搜索关键词"],
      "search_domains": ["推荐搜索领域"],
      "information_priority": "信息优先级指导"
    },
    "risk_assessment": {
      "technical_challenges": ["技术挑战"],
      "complexity_level": "low|medium|high|expert",
      "estimated_difficulty": "难度评估"
    }
  },
  "collaboration": { ... }
}
```

### Search 代理输出格式

```json
{
  "metadata": { ... },
  "data": {
    "search_summary": {
      "search_goals": ["搜索目标"],
      "tools_used": ["使用的工具"],
      "total_results": "搜索结果总数",
      "relevance_score": "相关性评分"
    },
    "knowledge_findings": {
      "key_concepts": [
        {
          "concept": "概念名称",
          "definition": "定义说明",
          "importance": "high|medium|low",
          "sources": ["信息来源"]
        }
      ],
      "technical_solutions": [
        {
          "solution": "解决方案描述",
          "pros": ["优点"],
          "cons": ["缺点"],
          "applicability": "适用场景",
          "confidence": "置信度（0-10）"
        }
      ],
      "best_practices": [
        {
          "practice": "最佳实践描述",
          "implementation": "实施建议",
          "benefits": ["带来的好处"],
          "sources": ["参考来源"]
        }
      ]
    },
    "resource_links": {
      "documentation": ["文档链接"],
      "examples": ["示例链接"],
      "tools": ["工具链接"],
      "community": ["社区资源链接"]
    }
  },
  "collaboration": { ... }
}
```

### Code 代理输出格式

```json
{
  "metadata": { ... },
  "data": {
    "project_overview": {
      "title": "项目标题",
      "description": "项目描述",
      "objectives": ["项目目标"],
      "success_criteria": ["成功标准"]
    },
    "technical_architecture": {
      "architecture_pattern": "架构模式",
      "core_modules": [
        {
          "module_name": "模块名称",
          "responsibilities": ["职责列表"],
          "dependencies": ["依赖项"],
          "interfaces": ["接口定义"]
        }
      ],
      "data_flow": "数据流程描述",
      "technology_stack": {
        "frontend": ["前端技术"],
        "backend": ["后端技术"],
        "database": ["数据库技术"],
        "deployment": ["部署技术"]
      }
    },
    "implementation_plan": {
      "phases": [
        {
          "phase_name": "阶段名称",
          "estimated_days": "预估天数",
          "tasks": [
            {
              "task_name": "任务名称",
              "description": "任务描述",
              "estimated_hours": "预估小时数",
              "dependencies": ["依赖项"],
              "deliverables": ["交付物"]
            }
          ],
          "milestones": ["里程碑"]
        }
      ],
      "total_estimation": {
        "days": "总天数",
        "hours": "总小时数",
        "confidence_level": "low|medium|high"
      }
    },
    "quality_assurance": {
      "code_standards": ["代码标准"],
      "testing_strategy": {
        "unit_tests": "单元测试策略",
        "integration_tests": "集成测试策略",
        "e2e_tests": "端到端测试策略"
      },
      "performance_requirements": ["性能要求"],
      "security_measures": ["安全措施"]
    },
    "risk_management": {
      "identified_risks": [
        {
          "risk": "风险描述",
          "probability": "high|medium|low",
          "impact": "high|medium|low",
          "mitigation": "缓解措施",
          "contingency": "应急计划"
        }
      ],
      "contingency_plans": ["应急计划"]
    }
  },
  "collaboration": { ... }
}
```

## 协作接口规范

### 代理间通信协议

#### Think → Search 接口
```json
{
  "message_type": "search_request",
  "sender": "think",
  "receiver": "search",
  "payload": {
    "search_objectives": ["搜索目标"],
    "keywords": {
      "primary": ["主要关键词"],
      "secondary": ["次要关键词"]
    },
    "search_domains": ["推荐搜索领域"],
    "priority_areas": ["重点关注领域"],
    "exclusion_criteria": ["排除标准"]
  }
}
```

#### Search → Code 接口
```json
{
  "message_type": "knowledge_delivery",
  "sender": "search",
  "receiver": "code",
  "payload": {
    "technical_knowledge": "技术知识摘要",
    "recommended_solutions": ["推荐解决方案"],
    "best_practices": ["最佳实践"],
    "tool_recommendations": ["工具推荐"],
    "resource_summary": "资源摘要",
    "confidence_levels": "各建议的置信度"
  }
}
```

### Memory 知识管理规范

#### 实体命名规范
- **Thinking Process**: `thinking_${timestamp}_${topic}`
- **Problem Pattern**: `pattern_${problem_type}`
- **Technical Solution**: `solution_${technology}_${approach}`
- **Best Practice**: `practice_${domain}_${category}`
- **Tool Recommendation**: `tool_${tool_name}_${use_case}`

#### 关系定义规范
- **relates_to**: 通用关联关系
- **solves**: 解决方案关系
- **improves**: 改进关系
- **requires**: 依赖关系
- **conflicts_with**: 冲突关系
- **recommends**: 推荐关系

## 质量保证标准

### 输出质量评分标准

每个代理都应对自己的输出进行质量评分：

- **9-10分**: 优秀，完全满足需求，可直接使用
- **7-8分**: 良好，基本满足需求，需要微调
- **5-6分**: 一般，部分满足需求，需要改进
- **3-4分**: 较差，勉强满足需求，需要重大改进
- **1-2分**: 很差，不满足需求，需要重新处理

### 数据验证规则

#### 必填字段验证
- `metadata.agent_name`: 非空字符串
- `metadata.timestamp`: 有效的时间戳
- `metadata.status`: 有效的状态值
- `data`: 非空对象

#### 数据格式验证
- 时间戳必须符合 ISO 8601 格式
- token_usage 必须为非负整数
- 评分必须在 0-10 范围内
- 列表字段不能为空

## 性能监控标准

### 关键性能指标 (KPI)

每个代理都应记录以下性能指标：

- **响应时间**: 从接收到输出的总时间
- **Token 使用量**: 输入、输出和总 token 数
- **工具调用次数**: 各工具的调用统计
- **错误率**: 失败请求的百分比
- **重试次数**: 失败重试的统计

### 日志记录规范

#### 日志级别
- **DEBUG**: 详细的调试信息
- **INFO**: 一般信息记录
- **WARN**: 警告信息
- **ERROR**: 错误信息
- **CRITICAL**: 严重错误

#### 日志格式
```json
{
  "timestamp": "时间戳",
  "level": "日志级别",
  "agent": "代理名称",
  "message": "日志消息",
  "context": {
    "execution_id": "执行ID",
    "additional_info": "额外信息"
  }
}
```

## 版本控制

### 标准版本号格式
采用语义化版本号：`MAJOR.MINOR.PATCH`

- **MAJOR**: 不兼容的重大变更
- **MINOR**: 向后兼容的功能新增
- **PATCH**: 向后兼容的问题修复

当前版本：**1.0.0**

### 升级兼容性
- 同一主版本内保证向后兼容
- 主版本变更时提供迁移指南
- 所有变更都应更新版本号和变更日志