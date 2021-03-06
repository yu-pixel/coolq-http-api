# 事件过滤器

将配置项 `use_filter` 设置为 `yes` 即可开启事件过滤器，插件启动时会在 `app\io.github.richardchien.coolqhttpapi` 文件夹下读取 `filter.json` 文件中定义的过滤规则（使用 JSON 编写），若文件不存在，或过滤规则语法错误，则会暂停所有上报。

## 示例

这节首先给出一些示例，演示过滤器的基本用法，下一节将给出具体语法说明。

### 只上报以「!!」开头的消息

```json
{
    "message": {
        ".regex": "^!!"
    }
}
```

### 只上报群组 11111、22222、33333 中不是用户 12345 发送的消息，以及用户 66666 发送的所有消息

```json
{
    ".or": [
        {
            "group_id": {
                ".in": [11111, 22222, 33333]
            }
            "user_id": {
                ".neq": 12345
            }
        },
        {
            "user_id": 66666
        }
    ]
}
```

### 一个更复杂的例子

```json
{
    ".or": [
        {
            "message_type": "private",
            "user_id": {
                ".not": {
                    ".in": [11111, 22222, 33333]
                },
                ".neq": 44444
            }
        },
        {
            "message_type": {
                ".regex": "group|discuss"
            },
            ".or": [
                {
                    "group_id": 12345
                },
                {
                    "message": {
                        ".contains": "通知"
                    }
                }
            ]
        }
    ]
}
```

## 语法说明

过滤规则最外层是一个 JSON 对象，其中的键，如果以 `.`（点号）开头，则表示运算符，其值为运算符的参数，如果不以 `.` 开头，则表示对事件数据对象中相应键的过滤。过滤规则中任何一个对象，只有在它的所有项都匹配的情况下，才会让事件通过（等价于一个 `and` 运算）；其中，不以 `.` 开头的键，若其值不是对象，则只有在这个值和事件数据相应值相等的情况下，才会通过（等价于一个 `eq` 运算符）。

下面列出所有运算符（「要求的参数类型」是指运算符的键所对应的值的类型，「可作用于的类型」是指在过滤时事件对象相应值的类型）：

| 运算符 | 要求的参数类型 | 可作用于的类型 |
| ----- | ------------ | ----------- |
| `.not` | object | 任何 |
| `.and` | object | 若参数中全为运算符，则任何；若不全为运算符，则 object |
| `.or` | array（数组元素为 object） | 任何 |
| `.eq` | 任何 | 任何 |
| `.neq` | 任何 | 任何 |
| `.in` | string/array | 若参数为 string，则 string；若参数为 array，则任何 |
| `.contains` | string | string |
| `.regex` | string | string |

插件在启动时读取过滤规则，如果读到无法识别的运算符，或「要求的参数类型」不符，则认为语法错误，将停止所有上报；在实际运行中执行过滤时，如果事件数据的类型和「可作用于的类型」不符，则认为过滤不通过。

## 过滤时的事件数据对象

过滤器在插件构建好事件数据后运行，各事件的数据字段见 [事件上报](/Post) 及其中的 [事件列表](/Post#事件列表)。

其中，`message` 字段较为特殊，在运行过滤器时，它是字符串类型，也就是说，它没有被拆分成消息段（关于消息段的概念，见 [消息格式](/Message)），而是保持原始的样子（也没有经过 [增强 CQ 码](/CQCode) 的处理）。因此，对 `message` 字段的过滤将按照字符串的规则来，且消息中可能会存在形如 `[CQ:face,id=123]` 的 CQ 码，在编写过滤规则时需要注意一下。
