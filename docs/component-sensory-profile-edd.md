# Schema: Component Sensory Profile (组件感官档案)

* **Version:** 3.0
* **Last Updated:** 2025-11-05

## 1. 核心理念与设计哲学 (Core Philosophy & Design Principles)

`Component Sensory Profile` (CSP) 是本认知无障碍营养系统的核心数据结构。其设计目标是超越传统的“标签映射”，转而为每个食物组件（Component）建立一个**动态的、充满语境的、多维度的感官身份**。

该 Schema 遵循以下核心原则：

1. **语境至上 (Context is King):** 一个感官属性的“价值”（好/坏）不是绝对的，而是由其来源和组件状态决定的。本 Schema 必须能精确地描述这种语境。
2. **动态而非静态 (Dynamic over Static):** 组件的感官特征会随时间变化。本 Schema 必须能够描述和预测这些“状态转换”。
3. **人类可读，机器可解 (Human-Readable, Machine-Parsable):** Schema 的定义应清晰、自解释，便于未来维护和扩展，同时保持严格的结构化以供系统使用。
4. **赋能预测 (Designed for Prediction):** Schema 的结构旨在为我们的人工智能/启发式引擎提供高质量的、结构化的数据，以实现“组件冲突预测”、“为您起草”和“拯救冰箱”等高级功能。

---

## 2. 根对象定义 (Root Object Definition)

根对象是一个**组件感官档案 (Component Sensory Profile)**的集合。在实践中，这通常是一个YAML或JSON文件的顶级键，例如 `component_sensory_profiles`。它包含一个 `profile` 对象的数组。

### `profile` (档案)

| 键 (Key)             | 数据类型                       | 必须? | 描述                                      |
|:--------------------|:---------------------------|:----|:----------------------------------------|
| `profile_id`        | `String`                   | ✅ 是 | 档案的唯一标识符。建议使用组件ID，例如 `veg_001`。         |
| `component_name`    | `String`                   | ✅ 是 | 组件的人类可读名称，例如 `长叶生菜 (Romaine Lettuce)`。  |
| `storage_notes`     | `String`                   | ❌ 否 | 关于如何最佳储存该组件以延长其理想状态的文本提示。               |
| `base_state`        | `State Object`             | ✅ 是 | 定义组件的“基础”或“初始”感官状态（例如购买时的状态）。           |
| `state_transitions` | `Array<Transition Object>` | ❌ 否 | 定义从一个状态到另一个状态的转换规则。对于感官稳定的组件（如盐），此项可省略。 |

---

## 3. 核心子对象定义 (Core Sub-Object Definitions)

### 3.1 `State Object` (状态对象)

`State Object` 描绘了一个组件在特定生命周期阶段的完整感官快照。

| 键 (Key)              | 数据类型                      | 必须? | 描述                                                        |
|:---------------------|:--------------------------|:----|:----------------------------------------------------------|
| `state_id`           | `String`                  | ✅ 是 | 状态的唯一标识符 (`optimal`, `degraded`, `terminal`, `unripe` 等)。 |
| `lifespan_days`      | `Integer`                 | ✅ 是 | 该状态在理想储存条件下的预计持续天数。                                       |
| `user_summary`       | `String`                  | ✅ 是 | 对该状态的一句简短、友好的总结，用于在UI中显示。                                 |
| `sensory_attributes` | `Array<Attribute Object>` | ✅ 是 | 构成该状态感官档案的属性对象数组。                                         |

### 3.2 `Attribute Object` (属性对象)

`Attribute Object` 是我们 Schema 中最小的、也是最核心的原子单位。它取代了简单的“标签”。

| 键 (Key)     | 数据类型     | 必须? | 描述                                                         |
|:------------|:---------|:----|:-----------------------------------------------------------|
| `tag_id`    | `String` | ✅ 是 | 引用主 `Sensory Lexicon` 中定义的唯一标签ID。                          |
| `source`    | `Enum`   | ✅ 是 | 属性的来源，用于理解其语境。**可选值:** `inherent`, `transition_gain`。      |
| `valence`   | `Enum`   | ✅ 是 | 属性在该状态下的价值（效价）。**可选值:** `positive`, `neutral`, `negative`。 |
| `intensity` | `Enum`   | ✅ 是 | 属性的感知强度。**可选值:** `weak`, `medium`, `strong`。               |

### 3.3 `Transition Object` (转换对象)

`Transition Object` 定义了从一个状态到另一个状态的“故事”和“规则”。

| 键 (Key)             | 数据类型                      | 必须? | 描述                                 |
|:--------------------|:--------------------------|:----|:-----------------------------------|
| `from_state`        | `String`                  | ✅ 是 | 转换的起始状态ID (e.g., `optimal`)。       |
| `to_state`          | `String`                  | ✅ 是 | 转换的目标状态ID (e.g., `degraded`)。      |
| `user_alert`        | `String`                  | ✅ 是 | 当转换发生时，系统向用户显示的、充满同理心的自然语言通知。      |
| `attributes_lost`   | `Array<String>`           | ❌ 否 | 在此转换中**失去**的`tag_id`列表。用于UI对比显示。   |
| `attributes_gained` | `Array<Attribute Object>` | ❌ 否 | 在此转换中**新获得**的`Attribute Object`列表。 |

---

## 4. 完整示例 (Complete Example)

以下是一个使用本 Schema 定义的“香蕉”组件的完整示例：

```json
{
  "profile_id": "fruit_001",
  "component_name": "香蕉 (Banana)",
  "storage_notes": "室温储存以催熟。放入冰箱会使皮变黑，但减缓果肉成熟。",
  "base_state": {
    "state_id": "unripe",
    "lifespan_days": 2,
    "user_summary": "略带青色，口感紧实，甜味较弱。",
    "sensory_attributes": [
      {
        "tag_id": "mouthfeel_hardness_05",
        "source": "inherent",
        "valence": "neutral",
        "intensity": "strong"
      },
      {
        "tag_id": "taste_basic_01",
        "source": "inherent",
        "valence": "positive",
        "intensity": "weak"
      }
    ]
  },
  "state_transitions": [
    {
      "from_state": "unripe",
      "to_state": "optimal",
      "user_alert": "您的香蕉现在是完美的亮黄色，达到了最佳甜度和口感！",
      "attributes_lost": [],
      "attributes_gained": [
        {
          "tag_id": "mouthfeel_hardness_03",
          "source": "transition_gain",
          "valence": "positive",
          "intensity": "medium"
        },
        {
          "tag_id": "taste_basic_01",
          "source": "transition_gain",
          "valence": "positive",
          "intensity": "strong"
        }
      ]
    },
    {
      "from_state": "optimal",
      "to_state": "degraded",
      "user_alert": "香蕉皮上出现了黑斑，果肉非常柔软，非常适合用来做香蕉面包或奶昔。",
      "attributes_lost": [
        "mouthfeel_hardness_05"
      ],
      "attributes_gained": [
        {
          "tag_id": "mouthfeel_hardness_02",
          "source": "transition_gain",
          "valence": "neutral",
          "intensity": "strong"
        },
        {
          "tag_id": "aroma_fermented_02",
          "source": "transition_gain",
          "valence": "neutral",
          "intensity": "weak"
        }
      ]
    }
  ]
}
```
