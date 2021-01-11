---
title: mermaid图表
date: 2020-12-25 19:11:15
author: zuo_ji
categories:
    - 编程技术
tags:
  - markdown
thumbnail:
blogexcerpt:
---

## 甘特图
```
gantt
dateFormat  YYYY-MM-DD
title Adding GANTT diagram to mermaid
excludes weekdays 2014-01-10

section A section
Completed task            :done,    des1, 2014-01-06,2014-01-08
Active task               :active,  des2, 2014-01-09, 3d
Future task               :         des3, after des2, 5d
Future task2               :         des4, after des2, 5d
```

```mermaid
gantt
dateFormat  YYYY-MM-DD
title Adding GANTT diagram to mermaid
excludes weekdays 2014-01-10

section A section
Completed task            :done,    des1, 2014-01-06,2014-01-08
Active task               :active,  des2, 2014-01-09, 3d
Future task               :         des3, after des2, 5d
Future task2               :         des4, after des2, 5d
```
<!-- more -->
## 时序图
```
sequenceDiagram
    Alice->>Bob: Hello Bob, how are you?
    alt is sick
        Bob->>Alice: Not so good :(
    else is well
        Bob->>Alice: Feeling fresh like a daisy
    end
    opt Extra response
        Bob->>Alice: Thanks for asking
    end
```


```mermaid
sequenceDiagram
    Alice->>Bob: Hello Bob, how are you?
    alt is sick
        Bob->>Alice: Not so good :(
    else is well
        Bob->>Alice: Feeling fresh like a daisy
    end
    opt Extra response
        Bob->>Alice: Thanks for asking
    end
```
```mermaid
sequenceDiagram
    participant A as hue-hr-recruiting
    participant B as AWS Lambda(dev)
    participant C as AWS Codebuild
    participant D as hpm-version
    A->>B: webhook
    B-->>B: Sync source code to s3 bucket.
    B->>+C: Trigger a build
    C-->>-B: success
    B->>D: Update commit sha of `hue-hr-recruiting` in version.yml
   
```
```mermaid
sequenceDiagram
    participant A as hpm-version
    participant B as AWS Lambda(prod)
    participant C as AWS Pipeline
    participant D as AWS Codebuild
    participant E as Slack

    A->>B: webhook: push a new release tag
    loop Every projects in version.yml
        B->>+B: sync source code to s3 bucket in prod env for current tag.
    end   
    B->>+C: Trigger a build
    C->>+E: request approval
    E-->>-C: approval
    loop Every projects in version.yml
        C->>+D: build (buildspec.yml)
        D-->>-C: success
    end
    C-->>-B: success
```



## 思维导图
### 利用mermaid来画思维导图
```mermaid
	graph TD
	A(工业用地效率)-->B1(土地利用强度)
	A-->B2(土地经济效益)
	B1-->C1(容积率)
	B1-->C2(建筑系数)
	B1-->C3(亩均固定资本投入)
	B2-->D1(亩均工业产值)
	B2-->D2(亩均税收)
```

```mermaid
graph LR
KaTex--> A(标记 Accents)
A-->B(撇,估计,均值,向量等写于符号上下的标记)
KaTex--> 分隔符_Delimiters
分隔符_Delimiters-->小中大括号,竖杠,绝对值等分隔符的反斜杠写法
KaTex--> 公式组_Enviroments
公式组_Enviroments-->C(.....)
B --> E(sdfdfdsf)
B --> E2(test)
B --> E3(test sdfsdf)
B --> E4(test dfewrwer)
E4 --> F("test content")
```

{% pullquote mindmap mindmap-md %}
- [Hexo 的思维导图插件](https://hunterx.xyz/hexo-simple-mindmap-plugin-intro.html)
  - 前言
  - 使用方法
    - 一
    - 二
    - 三
  - 太长不看
  - 参考资料
{% endpullquote %}

```math
e^{i\pi} + 1 = 0
```

<!-- $$
f(x) = \int_{-\infty}^\infty \hat f(\xi)\,e^{2 \pi \xi x} \,d\xi
$$ -->

:joy:

---

<!-- <script>
alert("hello")
</script> -->


## 参考文档
1. [mermaid-js](https://mermaid-js.github.io/mermaid/#/sequenceDiagram)
2. [hexo-math](https://github.com/hexojs/hexo-math)