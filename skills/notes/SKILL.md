---
name: notes
description: 当用户要求补注释、调整注释、review 注释质量、把注释规范沉淀为 skill，或代码改动涉及复杂判断、数据转换、最终输出模板和 Agent 提示边界时，使用这个 skill。目标是让注释只解释重点逻辑，不给简单代码逐行配旁白。
---

# notes

这个 skill 用于写代码注释。目标是让后来读代码的人更快理解关键意图，而不是把代码翻译成中文。

## 基本原则

注释只写在值得解释的地方。它应该说明为什么这样写、避免什么误解、字段代表什么业务含义，不能只复述代码表面行为。

简单赋值、普通取字段、明显的空值回退，不要单独写注释。能一眼读懂的代码，就让代码自己说话。

多行相似取值如果确实需要说明，优先在这一组代码上方写一行总结注释，不要每一行都配一句旁白。

## 必须加注释的场景

导出函数、class 的 static 方法，必须在定义上方写简短 JSDoc，说明入参或返回结果的业务含义。

复杂条件判断要加注释，尤其是 `if` 里同时判断多个字段、数字合法性、权限、状态、时间范围或空值组合。

数字计算要加注释，尤其是评分差值、比例差值、平均值、排名、百分比、精度处理和字符串小数转数字。

字段语义容易混淆时要加注释。比如某个字段是活动次数，不是人数；某个均值是 AI 校均，不是到课率校均。

输出模板、prompt、Agent 原样输出要求要加注释。说明哪些内容给模型看，哪些内容给用户看，为什么要用标签框住最终正文。

数组转 Map、按名称取维度、按业务 key 聚合这类结构转换要加注释。说明转换后是为了按业务语义读取。

## 禁止写法

不要给一行普通取值写注释，比如 `const questionActivityCount = output.question?.activityCount ?? "计算中";`。

不要写“获取某某”“设置某某”“返回某某”这种只复述代码表面的注释。

不要把连续十几个字段取值都逐行解释。普通字段取值保持一行，注释留给上方真正需要解释的分组逻辑。

不要写长段落。一条注释只讲一个重点。

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

修复后示例。普通取值保持一行，注释留给真正容易出错的判断和计算。

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
```
