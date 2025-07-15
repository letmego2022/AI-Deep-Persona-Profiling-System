

# AI深度人物画像系统 API 文档

**版本**: 1.0.0

**基础 URL**: `http://<your_server_address>:5001`

**概述**: 本系统提供一个交互式的、按需分析的流程，用于从公开数据中生成人物画像。用户通过启动一个“分析会话”来获取第一份画像报告和所有相关数据线索的引用列表，然后可以根据需要逐条请求分析剩余的数据线索。

**认证**: 当前版本的接口是开放的，不需要认证。

---

## 接口列表

1.  [`POST /start_analysis_session`](#post-start_analysis_session): 启动分析会话
2.  [`POST /analyze_next`](#post-analyze_next): 分析下一条数据线索

---

## 1. 启动分析会话

此接口是用户与系统交互的入口。它接收一个手机号，在数据库中查找所有相关的数据线索，对第一条线索进行AI分析，并返回初始报告和所有线索的引用列表。

*   **URL**: `/start_analysis_session`
*   **方法**: `POST`
*   **内容类型**: `application/x-www-form-urlencoded` 或 `multipart/form-data` (标准HTML表单提交)

### 请求参数

| 参数名 | 类型     | 位置 | 是否必须 | 描述                                                              |
| :------- | :------- | :--- | :------- | :---------------------------------------------------------------- |
| `phone`  | `string` | Body | 是       | 需要查询的手机号码。长度应在7到15位之间，且只包含数字。 |

**示例请求 (cURL):**
```bash
curl -X POST http://127.0.0.1:5001/start_analysis_session \
     -d "phone=13800138000"
```

### 响应

#### 成功响应 (HTTP `200 OK`)

**场景1: 找到数据并成功分析第一条**
```json
{
  "success": true,
  "total_count": 15,
  "data_references": [
    { "ref_collection": "collection_a", "ref_id": "65e8a1b2c3d4e5f6a7b8c9d0" },
    { "ref_collection": "collection_b", "ref_id": "65e8a2c3d4e5f6a7b8c9d1" }
    // ... 更多引用
  ],
  "analysis_result": {
    "source_data": {
      "folder_name": "gitee.com",
      "line_content": "User 'dev_master' pushed to branch main in project 'ruoyi-vue-pro'"
    },
    "ai_analysis": {
      "核心摘要": "一位在Gitee上活跃的开发者，可能参与了基于若依框架的项目。",
      "推断画像": {
        "职业角色": ["后端开发者", "Java工程师"],
        "能力技能": ["Java", "Vue", "Git"],
        "兴趣爱好": ["开源社区"]
      },
      "关联网络": {
        "关联人物": ["dev_master"],
        "关联组织": ["Gitee", "ruoyi-vue-pro"]
      },
      "行为模式推断": "正在积极维护或开发一个开源项目。",
      "评估体系": {
        "身份明确度": "低",
        "画像置信度": {
          "总分": 0.8,
          "职业置信度": 0.9,
          "爱好置信度": 0.7,
          "关联网络置信度": 0.85
        }
      }
    }
  }
}
```

**场景2: 未找到任何数据**
```json
{
  "success": true,
  "total_count": 0,
  "data_references": [],
  "analysis_result": null
}
```

#### 失败响应

**请求参数错误 (HTTP `400 Bad Request`)**
```json
{
  "success": false,
  "message": "号码格式不正确哦～"
}
```

---

## 2. 分析下一条数据线索

当用户对当前画像不满意并请求分析下一条时，调用此接口。它接收一个由 `/start_analysis_session` 返回的数据引用，并对该引用指向的数据进行AI分析。

*   **URL**: `/analyze_next`
*   **方法**: `POST`
*   **内容类型**: `application/json`

### 请求体 (JSON)

| 参数名        | 类型     | 是否必须 | 描述                                                           |
| :-------------- | :------- | :------- | :------------------------------------------------------------- |
| `reference`     | `Object` | 是       | 一个包含 `ref_collection` 和 `ref_id` 的对象，用于定位数据。 |

**示例请求体:**
```json
{
  "reference": {
    "ref_collection": "collection_b",
    "ref_id": "65e8a2c3d4e5f6a7b8c9d1"
  }
}
```

**示例请求 (cURL):**
```bash
curl -X POST http://127.0.0.1:5001/analyze_next \
     -H "Content-Type: application/json" \
     -d '{"reference": {"ref_collection": "collection_b", "ref_id": "65e8a2c3d4e5f6a7b8c9d1"}}'
```

### 响应

#### 成功响应 (HTTP `200 OK`)

响应结构与 `/start_analysis_session` 中的 `analysis_result` 字段完全相同。
```json
{
  "success": true,
  "analysis_result": {
    "source_data": {
      "folder_name": "zhaopin.com",
      "line_content": "{\"name\":\"张三\",\"phone\":\"13800138000\",\"position\":\"高级产品经理\",\"company\":\"某知名互联网公司\"}"
    },
    "ai_analysis": {
      "核心摘要": "一位正在求职或更新简历的高级产品经理，明确关联到某公司。",
      "推断画像": {
        "职业角色": ["高级产品经理"],
        "能力技能": ["产品设计", "用户研究", "项目管理"],
        "兴趣爱好": []
      },
      "关联网络": {
        "关联人物": ["张三"],
        "关联组织": ["zhaopin.com", "某知名互联网公司"]
      },
      "行为模式推断": "可能处于积极寻找新工作机会的状态。",
      "评估体系": {
        "身份明确度": "高",
        "画像置信度": {
          "总分": 0.95,
          "职业置信度": 0.98,
          "爱好置信度": 0.2,
          "关联网络置信度": 0.9
        }
      }
    }
  }
}
```

#### 失败响应

**请求体格式错误 (HTTP `400 Bad Request`)**
```json
{
  "success": false,
  "message": "无效的数据引用。"
}
```

**根据引用未找到数据 (HTTP `404 Not Found`)**
```json
{
  "success": false,
  "message": "无法根据引用找到数据。"
}
```

---
这份文档清晰地定义了每个接口的功能、请求方式、参数和所有可能的响应格式，可以作为前后端开发和未来维护的共同依据。
