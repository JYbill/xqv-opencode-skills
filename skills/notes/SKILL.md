---
name: notes
description: 当用户要求补注释、调整注释、review 注释质量、把注释规范沉淀为 skill，或代码改动涉及 if、for、switch、连续判断、三元表达式、数据转换、最终输出模板和 Agent 提示边界时，使用这个 skill。目标是让注释解释重点逻辑，不给简单代码逐行配旁白，也不制造没有意义的段落换行。
---

# notes

这个 skill 用于写代码注释。目标是让后来读代码的人更快理解关键意图，而不是把代码翻译成中文。

## 基本原则

注释只写在值得解释的地方。它应该说明为什么这样写、避免什么误解、字段代表什么业务含义，不能只复述代码表面行为。简单赋值、普通取字段、明显的空值回退，不要单独写注释。能一眼读懂的代码，就让代码自己说话。
多行相似取值如果确实需要说明，优先在这一组代码上方写一行总结注释，不要每一行都配一句旁白。发现无意义段落之间的 `\n\n` 时，改成 `\n`；空行只用于分隔不同业务阶段、不同控制流或确实需要停顿的说明。

## 必须加注释的场景

导出函数、class 的 static 方法，必须在定义上方写简短 JSDoc，说明入参或返回结果的业务含义。
`if`、`for`、`switch`、连续多个判断块、复杂三元表达式只有在判断意图不够直观时才需要加注释。注释要写在块或表达式上方，说明判断意图、循环目的、分支依据、提前退出原因或多个条件之间的业务关系。像 `message.role === "tool"`、`set.has(value)`、`list.includes(value)`、`value.length === 0` 这类一眼能看懂的单条件判断，不要为了满足形式单独加注释。
复杂条件判断要加注释，尤其是同时判断多个字段、数字合法性、权限、状态、时间范围、空值组合、短路逻辑或兜底分支。多个判断连续出现时，只有整组判断的业务顺序不明显时才补一条总注释，不要给每个显而易见的 `has` / `includes` 分支逐句解释。
数字计算要加注释，尤其是评分差值、比例差值、平均值、排名、百分比、精度处理和字符串小数转数字。
字段语义容易混淆时要加注释。比如某个字段是活动次数，不是人数；某个均值是 AI 校均，不是到课率校均。
输出模板、prompt、Agent 原样输出要求要加注释。说明哪些内容给模型看，哪些内容给用户看，为什么要用标签框住最终正文。
数组转 Map、按名称取维度、按业务 key 聚合这类结构转换要加注释。说明转换后是为了按业务语义读取。
字符串拼接、join 分隔符、展示文案合成这类逻辑要加注释。说明最终会拼成什么人能看到的样子，比如 `学生A、学生B、学生C`。

## 禁止写法

不要给一行普通取值写注释，比如 `const questionActivityCount = output.question?.activityCount ?? "计算中";`。
不要给一眼能看懂的单条件判断写注释，比如 `if (message.role === "tool") return;`、`if (ids.has(id)) return true;`、`if (!names.includes(name)) continue;`、`if (items.length === 0) return;`。这类判断让代码自己说话。
不要写“获取某某”“设置某某”“返回某某”这种只复述代码表面的注释。
不要把连续十几个字段取值都逐行解释。普通字段取值保持一行，注释留给上方真正需要解释的分组逻辑。
不要写长段落。一条注释只讲一个重点。不要在注释之间加入没有信息量的空段落；无意义段落之间的 `\n\n` 要改成 `\n`。

## 示例

错误示例。这里把简单取值逐行解释了一遍，注释没有提供新信息，只会打断阅读。

```ts
// 答题次数来自 question.activityCount。
const questionActivityCount = output.question?.activityCount ?? "计算中";

// 讨论活动次数来自 discussion.activityCount。
const discussionActivityCount = output.discussion?.activityCount ?? "计算中";

// 人均发帖数同样只做展示，不在这里重新计算。
const discussionAverageMessageRate = output.discussion?.averageMessageRate ?? "计算中";
```

修复后示例。普通取值和一眼能看懂的单条件判断保持原样，注释留给真正容易出错的判断、循环、分支和计算。

```ts
// 这些字段只是把已有统计结果塞进模板，保持一组注释即可。
const questionActivityCount = output.question?.activityCount ?? "计算中";
const discussionActivityCount = output.discussion?.activityCount ?? "计算中";
const discussionAverageMessageRate = output.discussion?.averageMessageRate ?? "计算中";

const attendanceRateNumber = Number(evaluation.attendanceRate);
const attendanceRateAverageNumber = Number(evaluation.attendanceRateAverage);
// 两个值都是真正的数字时才计算差值，否则保留原始展示文本，避免输出 NaN。
if (Number.isFinite(attendanceRateNumber) && Number.isFinite(attendanceRateAverageNumber)) {
  const attendanceRateComparison = attendanceRateNumber >= attendanceRateAverageNumber ? "高出" : "低于";
}

// 按固定枚举分支生成文案，避免新增状态时误走默认展示。
switch (evaluation.trend) {
  case "up":
    trendText = "上升";
    break;
  case "down":
    trendText = "下降";
    break;
}

// 遍历展示姓名时只拼用户可读文本，不暴露内部 studentId。
for (const studentName of quickAnswerTopStudentNames) {
  quickAnswerTopStudentTexts.push(`${studentName}同学互动积极`);
}

// 先区分是否缺数据，再比较高低，避免三元表达式把两个业务判断压在一起。
const attendanceRateText = Number.isFinite(attendanceRateNumber)
  ? attendanceRateNumber >= attendanceRateAverageNumber
    ? "高于校均"
    : "低于校均"
  : "计算中";

// 有姓名时拼成 学生A、学生B、学生C 这种展示形式，不展示内部 studentId。
const quickAnswerTopStudentText = `${quickAnswerTopStudentNames.join("、")}同学互动积极`;
```
