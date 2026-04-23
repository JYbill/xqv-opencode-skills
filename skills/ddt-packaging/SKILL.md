---
name: ddt-packaging
description: Detect and correct over-packaging in code. Use this skill whenever the user complains about over-abstraction, over-packaging, over-encapsulation, too many tiny helpers, unnecessary type splitting, readability regressions from excessive method extraction, or asks to inline small single-use functions or merge prematurely split types.
---

# ddt-packaging

DDT means dont-do-that packaging.

这个 skill 用来约束一种常见坏味道，就是明明代码很短、语义很直接、调用关系也很单一，却还要硬拆一层函数、硬拆一层类型，最后让人类阅读路径变长，理解成本变高。

## 什么时候触发

当用户表达下面这类意思时，优先使用这个 skill。

- 说代码过度封装、过度抽象、拆得太碎
- 指出某个函数只有 2 到 5 行，而且只被一个地方调用
- 指出某个类型本来一份就够，却被拆成多个薄类型来回跳转
- 要求把 helper 内联回主流程
- 要求把薄类型合并，减少阅读跳转
- 要求代码更贴近人类阅读，而不是为了形式上的分层去拆分

## 核心原则

先判断拆分有没有真实收益，再决定保留还是收回。

如果一个函数只有很少几行，只有一个调用方，而且函数名也没有提供新的业务语义，默认这是过度封装，应该直接内联回调用处。不要为了看起来更模块化就保留这种薄函数。

如果一个类型只是从另一个类型里薄薄切出一层，没有形成稳定复用，也没有显著降低理解成本，默认这是过度拆分，应该合并回更直接的类型定义。只有在某个子类型真的被多个地方独立使用，或者拆出来以后能明显降低主类型复杂度时，才保留拆分。

代码优先服务人类阅读，不优先服务形式上的解耦。局部轻微耦合通常比多跳一层更好读。

## 函数判断规则

遇到小函数时，按这个顺序判断。

先看它是不是只有 2 到 5 行的简单逻辑。像简单赋值、一次分支、一次透传、一次 map、一次字符串判断，这类默认不值得单独抽函数。

再看它是不是只有一个调用方。如果只有一个调用方，而且调用点离定义点很近，优先内联。

再看函数名有没有补充新的业务语义。如果函数名只是把实现重念一遍，比如 parseRound、rememberRound、buildPayload 这种薄包装名，而代码本身已经一眼能懂，优先内联。

再看抽出来之后是否真的减少主流程认知负担。如果主流程仍然必须点进去看细节，那这层抽取通常没有价值。

只有下面这些情况才建议保留独立函数。

函数被多个地方复用。函数承载了清晰、稳定、不可忽略的业务语义。函数内部逻辑已经长到会压坏主流程阅读。或者函数需要独立测试、独立注释、独立复用。

## 类型判断规则

遇到类型拆分时，按这个顺序判断。

先看一份类型能不能直接把当前问题表达清楚。如果能，就不要为了层次感继续拆。

再看子类型是不是只服务一个父类型，而且只在一个地方被消费。如果是，优先并回去。

再看拆分后的收益是不是真实存在。如果拆完以后只是名字变多、跳转变多、理解路径变长，没有形成复用或明显压缩复杂度，就属于过度拆分。

只有下面这些情况才建议拆出子类型。

子类型会被多个模块共享。某个嵌套结构本身就有稳定语义。主类型已经长到影响阅读，拆出来能明显提升可读性。或者这个子类型本身就是一个独立边界，比如 API 返回体、数据库行结构、消息体结构。

## 执行动作

当确认存在过度封装时，直接做这些事。

把单一调用方的小函数内联回调用处。删掉无收益的 helper。把过薄的类型合并回主类型。删掉只是转手透传的中间层。保留必要注释，但只解释关键意图，不重复代码表面行为。

如果用户是在 review 代码而不是要求直接修改，就明确指出哪些函数或类型属于过度封装，并给出一句话理由，重点讲清楚为什么这层拆分让阅读变差。

## 输出要求

给出的建议或修改，默认遵守这些表达习惯。

直接指出该内联的点和该合并的点。不要泛泛而谈抽象层次。不要为了凑整再引入新的 helper。不要把刚收回去的逻辑换个名字再包一层。

如果要解释原因，用一句短话说清楚就够了，比如只有一个调用方且逻辑过薄，内联后更好读。或者这个子类型没有独立复用价值，合并后阅读路径更短。

## 反模式清单

下面这些默认都当作 dont-do-that packaging。

只有一个调用方的 2 到 5 行薄函数。

只做一次字段拷贝、一次简单判断、一次字符串匹配、一次 map 包装的 helper。

为了拆而拆的 parseX、buildX、normalizeX、rememberX、toX 这类薄包装函数。

只包一层取值、改名、透传的 getter、selector、mapper、convert 函数。

本来一个类型能表达清楚，却拆成主类型加一层子类型再加一层别名类型。

没有复用价值，却只为了看起来工整而把嵌套对象强行拆出去。

## 例子

例子一。

下面这种写法就是典型的薄函数过度封装。`handleMessageEvent()` 只有两行真正逻辑，而且只在一个 `case` 里被调用。

```ts
case AgentFlowEnum.MESSAGE: {
  this.handleMessageEvent(chunk, messageData);
  break;
}

private handleMessageEvent(chunk: DifyMsgSSERes, messageData: MessagePayload) {
  messageData.data.answer = chunk.answer ?? "";
  this.writeMessagePayload(messageData);
}
```

更合适的写法是直接内联回 `switch` 分支，阅读路径更短。

```ts
case AgentFlowEnum.MESSAGE: {
  messageData.data.answer = chunk.answer ?? "";
  this.writeMessagePayload(messageData);
  break;
}
```

例子二。

下面这种写法也是典型的为了拆而拆。`parseRound()` 只有正则匹配和 `parseInt()`，而且只被 `resolveRound()` 用一次。

```ts
private resolveRound(logData: AgentMessageData): number {
  const round = this.parseRound(logData.label);
  if (round !== null) {
    return round;
  }

  if (logData.parent_id !== null) {
    const parentRound = this.roundMap.get(logData.parent_id);
    if (parentRound !== undefined) {
      return parentRound;
    }
  }

  throw new Error(`agent_log round not found: ${logData.label}`);
}

private parseRound(label: string | undefined): number | null {
  if (!label) return null;
  const matched = label.trim().match(/^ROUND (\d+)$/u);
  if (!matched) {
    return null;
  }
  return Number.parseInt(matched[1], 10);
}
```

更合适的写法是把这几行逻辑直接收回 `resolveRound()`。

```ts
private resolveRound(logData: AgentMessageData): number {
  const matched = logData.label.trim().match(/^ROUND (\d+)$/u);
  if (matched) {
    return Number.parseInt(matched[1], 10);
  }

  if (logData.parent_id !== null) {
    const parentRound = this.roundMap.get(logData.parent_id);
    if (parentRound !== undefined) {
      return parentRound;
    }
  }

  throw new Error(`agent_log round not found: ${logData.label}`);
}
```

例子三。

下面这种类型拆分也属于过度。读代码的人需要先看 `DifyMsgSSERes`，再跳到 `DifyWorkflowEventData`，再跳到 `AgentMessageData`，但实际这里并没有形成稳定复用边界。

```ts
export type AgentMessageData = {
  id: string;
  parent_id: string | null;
  label: string;
  status: string;
};

export type DifyWorkflowEventData = AgentMessageData & {
  title?: string;
  outputs?: {
    structured_output?: Record<string, any>;
    [key: string]: any;
  };
  [key: string]: any;
};

export type DifyMsgSSERes = {
  event: string;
  data: DifyWorkflowEventData;
};

const logData = chunk.data as AgentMessageData;
```

更合适的写法是用一份类型直接把当前边界讲清楚，消费时也不需要再额外断言。

```ts
export type AgentMessageData = {
  id: string;
  parent_id: string | null;
  label: string;
  status: string;
  title?: string;
  outputs?: {
    structured_output?: Record<string, any>;
    [key: string]: any;
  };
  [key: string]: any;
};

export type DifyMsgSSERes = {
  event: string;
  data: AgentMessageData;
};

const logData = chunk.data;
```

例子四。

下面这种 getter、selector、mapper、convert 风格的方法，如果本质上只是包一层取值改名透传，也属于过度封装。

```ts
const labels = toolList.map((tool) => this.mapToolLabel(tool));

private mapToolLabel(tool: AgentTool) {
  return tool.label;
}
```

更合适的写法是直接把这层薄包装收掉。

```ts
const labels = toolList.map((tool) => tool.label);
```

## 决策偏好

拿不准时，优先少一层。

能直接放在当前上下文里读懂，就不要额外封装。

能用一个类型讲清楚，就不要先拆再绕回来。
