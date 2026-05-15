---
name: notes
description: 当用户要求补注释、调整注释、review 注释质量、把注释规范沉淀为 skill，或代码改动涉及复杂判断、数据转换、最终输出模板和 Agent 提示边界时，使用这个 skill。目标是让注释只解释重点逻辑，不给简单代码逐行配旁白。
---

# notes

这个 skill 用于写代码注释。目标是让后来读代码的人更快理解关键意图，而不是把代码翻译成中文。

处理注释任务时，不只看用户点名的局部位置。需要顺手检查同一改动范围内的方法级 JSDoc、复杂判断、状态切换、数据转换、模型上下文和事件顺序；不要扩散到无关文件。

## 基本原则

注释只写在值得解释的地方。它应该说明为什么这样写、避免什么误解、字段代表什么业务含义，不能只复述代码表面行为。

简单赋值、普通取字段、明显的空值回退，不要单独写注释。能一眼读懂的代码，就让代码自己说话。

多行相似取值如果确实需要说明，优先在这一组代码上方写一行总结注释，不要每一行都配一句旁白。

如果代码能通过局部变量、拆分条件或调整命名读清楚，优先让代码本身变清楚，不要用注释掩盖难读表达。

## 必须加注释的场景

导出函数、class 的 static 方法，必须在定义上方写简短 JSDoc，说明方法用途；入参或返回结果有业务含义时再补充说明。

复杂业务实例方法也要补 JSDoc。简单私有方法如果只是薄透传、取值或短判断，不要为了形式强行写注释。

复杂条件判断要加注释，尤其是 `if` 里同时判断多个字段、数字合法性、权限、状态、时间范围或空值组合。

`if` 的有效代码行超过 10 行，且这个分支承载完整业务处理、状态分流、数据转换或副作用时，要加注释说明分支意图或关键边界。

数字计算要加注释，尤其是评分差值、比例差值、平均值、排名、百分比、精度处理和字符串小数转数字。

字段语义容易混淆时要加注释。比如某个字段是活动次数，不是人数；某个均值是 AI 校均，不是到课率校均。

输出模板、prompt、Agent 原样输出要求要加注释。说明哪些内容给模型看，哪些内容给用户看；如果使用标签、分隔符或原样输出边界，要说明它们用来避免什么误读。

状态切换、事件顺序、流式输出阶段等容易误判先后关系的逻辑要加注释。说明顺序约束或协议边界，不要只写正在执行某步骤。

数组转 Map、按名称取维度、按业务 key 聚合这类结构转换要加注释。说明转换后是为了按业务语义读取。

字符串拼接、join 分隔符、展示文案合成这类逻辑要加注释。说明最终会拼成什么人能看到的样子，比如 `学生A、学生B、学生C`。

## 禁止写法

不要给一行普通取值写注释，比如 `const questionActivityCount = output.question?.activityCount ?? "计算中";`。

不要写“获取某某”“设置某某”“返回某某”“处理某分支”这种只复述代码表面的注释。

不要因为 `if` 超过 10 行就机械加注释。注释必须解释这个长分支的业务意图、边界或容易误读的原因。

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

修复后示例。普通取值保持一组注释，注释留给真正容易出错的判断、计算和长业务分支。

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

const scoreNumber = Number(evaluation.score);
const averageScoreNumber = Number(evaluation.averageScore);
// 成绩有效时才进入完整分析分支，避免无成绩场景输出误导性结论。
if (Number.isFinite(scoreNumber) && Number.isFinite(averageScoreNumber)) {
  const scoreDiff = Math.abs(scoreNumber - averageScoreNumber).toFixed(1);
  const scoreComparison = scoreNumber >= averageScoreNumber ? "高出" : "低于";
  const scoreTrendText = `本次成绩${scoreComparison}平均值 ${scoreDiff} 分`;
  const rankText = evaluation.rank ? `班级排名第 ${evaluation.rank}` : "暂无排名";
  const activityText = evaluation.activityText ?? "暂无互动数据";
  const riskText = evaluation.riskText ?? "暂无风险提示";
  const teacherAdviceText = evaluation.teacherAdviceText ?? "暂无教师建议";
  const studentAdviceText = evaluation.studentAdviceText ?? "暂无学生建议";
  const summaryText = `${scoreTrendText}，${rankText}`;
  const finalText = `${summaryText}。${activityText}。${riskText}。${teacherAdviceText}。${studentAdviceText}`;
  output.push(finalText);
}

// 有姓名时拼成 学生A、学生B、学生C 这种展示形式，不展示内部 studentId。
const quickAnswerTopStudentText = `${quickAnswerTopStudentNames.join("、")}同学互动积极`;
```
