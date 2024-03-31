## 问题说明
基于以下事实和规则，通过谓词演算来推断小狗 Fred 的位置：

1. Fred 是一只柯利狗。
1. Sam 是 Fred 的主人。
1. 今天是星期六。
1. 星期六很冷。
1. Fred 是一只训练有素的狗。
1. 训练有素的柯利狗是很好的狗。
1. 如果一条狗是好狗，而且它有主人，那么它会和他的主人在一起。
1. 如果是周末，而且天气暖和，那么 Sam 就去公园。
1. 如果是周末，而且天气不暖和，那 Sam 就去博物馆。


## 解决步骤
完整的文件请查看 idp 文件：[solve-location.idp](solve-location.idp)

基于 IDP-Z3 的语法定义规则，将上面的语句转成 idp-z3 的规则定义。

### 定义 Vocabulary

首先定义出类型：
* Dog
* People
* Day
* Location

再定义边：

| Edge | 映射 | 说明 |
| ---- | ---- | ---- |
| collie | Dog -> Bool | 是否是柯利狗 |
| masterOf | Dog -> People | 狗的主人 |
| warm | Day -> Bool | 某天天气是否暖和 |
| trained | Dog -> Bool | 狗是否训练有素 |
| goodDog | Dog -> Bool | 狗是否是好狗 |
| locationOfPeople | People -> Location | 某人的位置 |
| locationOfDog: | Dog -> Location | 某只狗的位置 |
| weekend | Day -> Bool | 定义周末|
| today | () -> Day | 定义今天|

### 给出 Structure

| 条件 | 语句 |
| ---- | ---- |
| Fred 是一只柯利狗 | collie := {Fred} |
| Sam 是 Fred 的主人 | masterOf := {Fred -> Sam} |
| 今天是星期六 | today := Saturday |
| 星期六很冷 | warm := {} |
| Fred 是一只训练有素的狗 | trained := {Fred} |
| 周末 | weekend := {Saturday, Sunday} |

### 给出 Theory

| 条件 | 语句 |
| ---- | ---- |
| 训练有素的柯利狗是很好的狗 | !x in Dog: collie(x) and trained(x) => goodDog(x). |
| 如果一条狗是好狗，而且它有主人，那么它会和他的主人在一起 | !x in Dog: !y in People: !z in Location: goodDog(x) and masterOf(x) = y and locationOfPeople(y) = z => locationOfDog(x) = z. |
| 如果是周末，而且天气暖和，那么 Sam 就去公园 | weekend(today()) and warm(today()) => locationOfPeople(Sam) = Park. |
| 如果是周末，而且天气不暖和，那 Sam 就去博物馆 | weekend(today()) and ~warm(today()) => locationOfPeople(Sam) = Museum. |

### 定义 main block

```python
pretty_print(Theory(T, S).expand(10))
```

## 执行结果

执行以下命令：
```shell
idp-engine solve-location.idp
```

得到推理结果：

```python
Model 1
==========
collie := {Fred}.
masterOf := {Fred -> Sam}.
warm := {}.
trained := {Fred}.
goodDog := {Fred}.
locationOfPeople := {Sam -> Museum}.
locationOfDog := {Fred -> Museum}.
weekend := {Saturday, Sunday}.
today := Saturday.
```

即 Fred 在 Museum。
