# 🌳 Tree Shaking - "코드를 흔들어 가볍게"

## 주제 선정 이유

프로젝트에서 불필요한 코드로 번들이 커져 성능이 저하되는 문제를 겪고, 이를 해결하기 위해 주제를 선정했고,
이 과정에서 import/export를 기반으로 한 트리셰이킹에 대한 궁금증이 생겨 좀 더 자세히 알고자하여 선택하였습니다.

--- 

## 📖 목차 

1. [번들링이란?](#번들링이란)
2. [번들링 최적화 기법 - Tree Shaking](#번들링-최적화-기법---tree-shaking)
3. [Tree Shaking의 원리와 고려사항](#tree-shaking의-원리와-고려사항)
4. [Tree Shaking in RollupJS](#tree-shaking-in-rollupjs)
5. [정리](#정리)
6. [Q&A](#qa)


## 번들링이란?

### 번들(Bundle)의 의미

**Bundle = 묶음**이라는 뜻으로, 번들러는 **필요한 여러 모듈들을 하나로 묶는 도구**입니다.

### 번들러 없이 개발할 때의 문제점

간단한 계산기 예제를 통해 문제점을 살펴보겠습니다.

#### ❌ 전역 변수 충돌 문제
```javascript
// add.js
const firstInput = document.getElementById('input1');

// minus.js  
const firstInput = document.getElementById('input1'); // 전역 변수 충돌!
```

#### ❌ 네트워크 요청 문제
- 파일마다 개별 HTTP 요청 필요
- 파일 수 증가 시 요청 병목 현상

#### ❌ 수동 의존성 관리의 한계
```html
<!-- HTML에서 스크립트 순서를 수동으로 관리해야 함 -->
<script src="add.js"></script>
<script src="minus.js"></script>
<script src="main.js"></script>
```

**더 많은 기능이 추가되면?**
- ⚠️ 순서 의존성 증가
- ⚠️ 모듈 관계 복잡화
- ⚠️ 모든 코드가 브라우저로 전송 (불필요한 코드 포함)
- ⚠️ 유지보수 불가능

###  번들러를 사용하는 이유

- **요청 수 감소** - 여러 파일을 하나로 합쳐 네트워크 요청 최소화  
- **로딩 속도 향상** - 번들된 파일의 효율적인 로딩  
- **캐싱 최적화** - 번들된 파일 하나만 캐시  
- **유지보수성과 배포 효율성** - 개발할 때는 모듈화, 배포할 때는 성능 최적화

### 번들링 과정
**3단계:**
1. **모듈 탐색** - Entry point부터 의존성 그래프 생성
2. **의존성 구조 정리** - 모듈 간의 관계와 실행 순서 파악
3. **번들 파일 생성** - 하나의 최적화된 파일로 통합

---

## 번들링 최적화 기법 - Tree Shaking

### 주요 최적화 기법들

-  **Tree Shaking**: 사용하지 않는 코드(import) 제거  
-  **Code Splitting**: 한 파일을 여러 개의 작은 파일로 나누기  
-  **Minification**: 공백 / 주석을 없애서 크기 줄이기

### Code Splitting의 효과

**Before (No Code Splitting):**
- 모든 기능을 한 번에 로드
- 초기 로딩 시간 증가

**After (Code Splitting):**
- 필요에 따라 지연 로딩
- 초기 로딩 성능 향상

```javascript
// React에서의 Code Splitting 예시 
import React, { Suspense } from "react";

const LazyComponent = React.lazy(() => import("./LazyComponent"));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <LazyComponent />
    </Suspense>
  );
}
```

### 🌳 Tree Shaking이란?

> **Tree shaking is a term commonly used within a JavaScript context to describe the removal of dead code.**
>
> *사용되지 않는 코드(dead code)를 제거하기*

나무에서 불필요한 잎을 터는(shake) 것처럼, 코드에서 사용하지 않는 부분을 제거한다는 의미입니다.

#### 문제 상황
```javascript
import * as util from '../utilFile';  // 전체 import
```
거대한 유틸리티 라이브러리를 전체 import하고 있습니다. 그러나 실제로는
일부 함수만 사용합니다. 번들 파일의 크기가 증가하고, 그에 따라 로딩 시간도 증가되어 페이지 로딩 속도가 저하 됩니다.

#### 해결책
```javascript
// math.js
export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {  // 사용되지 않음 - Tree Shaking으로 제거 
  return a - b;
}

// main.js
import { add } from './math.js';  // 필요한 부분만 import
console.log(add(2, 3));
```

---

## Tree Shaking의 원리와 고려사항

### 정적 분석(Static Analysis) 기반

Tree Shaking은 **정적 분석**을 통해 동작합니다.

> **정적 분석**: 프로그램을 실행하지 않고 코드를 분석하는 것

Tree Shaking이 제대로 동작하려면, **코드가 예측 가능하고 정적인 방식으로 구성**되어야 합니다.

### 트리쉐이킹을 위한 조건: 정적 import/export 구문

이러한 기능은 2015년에 ES6에서 도입된 **ES Module(ESM)** 문법을 통해 가능해졌습니다. ESM은 import와 export를 명확하게 선언하므로써 Webpack과 같은 번들러가 프로그램 실행 없이도 모듈 간의 구조를 분석할 수 있습니다.

#### 정적인 코드 (ES6 - Tree Shaking 가능)
```javascript
// utils.js
export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}

// main.js (ES6 이후 문법)
import { add } from './utils.js';  // 정적으로 분석 가능
console.log(add(1, 2));
```

#### 동적인 코드 (CommonJS - Tree Shaking 불가능)
```javascript
// 런타임에 모듈 경로가 결정됨
const path = './' + moduleName;
const mod = require(path);  // 정적 분석 불가능

// 런타임에 export가 결정됨
if (Math.random() > 0.5) {
  module.exports = { foo: () => {}, bar: () => {} };
} else {
  module.exports = () => 'Hello';
}
```

이전까지 개발자들은 대표적으로 Webpack에서 기본적으로 제공하는 **CommonJS 방식**의 모듈 시스템을 사용했습니다. 하지만 CommonJS의 `require()`는 런타임 시점에서 어떤 모듈을 가져올지 결정되기 때문에 정적 분석이 어렵고 Tree Shaking이 잘 동작하지 않습니다.

### Babel 설정 시 주의사항

실제 개발 환경에서는 우리가 아무리 코드를 `import/export`로 잘 작성했더라도 Tree Shaking이 제대로 작동하지 않는 경우가 있습니다. 대표적인 경우가 바로 **Babel을 사용하는 경우**입니다.

Babel은 최신 JavaScript 코드를 구형 브라우저에서도 동작하도록 변환해주는 도구입니다. 그런데 Babel은 기본 설정으로 코드를 ES6 → ES5로 바꾸면서 `import` 구문을 `require()`로 변환해버립니다.

```javascript
// Babel 변환 전 (ES6)
import { add } from './math.js';

// Babel 변환 후 (ES5) - Tree Shaking 불가능!
const { add } = require('./math.js');
```

**해결 방법:**


```
// .babelrc

{
  “presets”: [ 
    [
      “@babel/preset-env”,
      {
	    "modules": false 
      }
    ]
 ]
}
```


Babel 설정에서 `modules: false` 옵션을 사용한다면 import/export 구문을 변환하지 않고 그대로 유지하게 됩니다. 이렇게 하면:
1. 번들링 도구가 먼저 Tree Shaking 수행
2. 그 후 Babel이 변환 작업 수행
3. Tree Shaking도 되고, 구형 브라우저 호환도 유지!

### Side Effects 고려

```
// package.json
{
  "sideEffects": false  // 내 프로젝트의 모든 모듈은 side effect가 없다는 설정
}
```

기본적으로 번들러는 매우 조심스럽게 코드가 **부작용이 없다는 확신이** 있어야 코드를 제거합니다. 그렇기에 우리는 정말 안전하다고 생각하는 파일들이 있다면 명시적으로 알려줘야 합니다.

#### Side Effect가 있는 코드 예시
```javascript
// setupTheme.js - import만 해도 실행되는 사이드 이펙트가 있는 코드
const isDark = window.matchMedia('(prefers-color-scheme: dark)').matches;

if (isDark) {
  document.documentElement.classList.add('dark');
} else {
  document.documentElement.classList.remove('dark');
}
```

⚠️ `sideEffects: false` 처리를 해주지 않으면 Tree Shaking의 대상이 될 수 있으므로 **제거 되지 않도록 별도 처리가 필요**합니다.

### Tree Shaking을 제대로 적용하려면

✅ **ES6모듈 구문을 사용해야 한다**  
✅ **컴파일러가 ES모듈을 CommonJS 모듈로 변환하지 않도록 해야한다**  
✅ **Side Effect를 고려하자**

---

## Tree Shaking in RollupJS

저희는 Tree Shaking의 근본적인 구동 방법이 궁금해졌습니다. 이 과정에서 **ESM만을 기반으로 하는 정적 번들러인 Rollup**을 찾아보게 되었습니다.

### Rollup의 특징

✅ **ESM 기반** - ES6 모듈을 기본으로 지원  
✅ **Tree Shaking에 최적화** - 뛰어난 Tree Shaking 성능  
✅ **오픈소스** - GitHub에서 내부 구현 확인 가능

### 들어가기 전에 - 영화 편집 비유

Rollup의 번들링 내용이 굉장히 복잡하고 이해하기 어렵습니다. 그렇기에 들어가기 전에 알고 가면 좋을 내용을 비유를 통해 정리해보겠습니다.

- **Module**: 영화 장면 
- **AST**: 촬영본 - 구조화된 데이터 
- **Graph**: 영화 감독 

### Rollup에서의 Tree Shaking 과정

Rollup의 `Graph.ts` 파일의 `build()` 함수는 **3단계**로 Tree Shaking을 수행합니다.

#### 1. 모듈 로딩 & AST 생성 (`generateModuleGraph()`)

```javascript
// Graph.ts
await this.generateModuleGraph();
```

Graph 파일의 `generateModuleGraph()` 함수는 내부적으로 `addEntryModules()`를 호출해 우리가 작성한 모든 JS 파일을 모듈 단위로 불러옵니다. 각 모듈은 이후 몇 단계를 거쳐 Module 파일의 `setSource()` 함수를 실행하게 됩니다.

**주요 과정:**
- **코드 파싱**: `parseAsync()`로 소스코드를 AST로 변환
- **AST 구성**: Abstract Syntax Tree를 Program 객체로 생성
- **의존성 관계 로딩**: `fetchModuleDependencies()`로 모듈 간 의존 관계 파악

```javascript
// Module.ts - setSource()
const astBuffer = await parseAsync(code, false, this.options);
this.ast = convertProgram(astBuffer, programParent, this.scope);
```

**AST (Abstract Syntax Tree):**
- 우리가 작성한 JS 코드를 트리 형태로 추상화한 구조
- "어떤 변수가 정의됐는지, 어떤 함수가 export됐는지, 또 어떤 모듈을 import하고 있는지" 같은 정보가 모두 들어있음
- 이걸 기반으로 Rollup이 사용된 코드와 아닌 코드를 정적으로 판단


#### 2️. 모듈 정렬 & 참조 바인딩 (`sortModules()`)

```javascript
// Graph.ts
this.sortModules();
```

이 단계에서는 앞서 모듈들이 AST로 구성된 이후, 실제 번들링을 어떤 순서로 수행할지를 결정하게 됩니다.

**두 가지 핵심 작업:**

**A. 실행 순서 정렬 (`analyseModuleExecution`)**
- DFS(깊이 우선 탐색) 방식으로 의존성 분석
- 모듈이 실행되어야 할 순서 결정

```
main.js (import AA.js)
  ↓
AA.js (import BB.js, CC.js)
  ↓         ↓
BB.js     CC.js

실행 순서: CC → BB → AA → main
```

**B. 참조 바인딩 (`bindReferences`)**
- 각 모듈의 AST에서 변수나 함수 이름이 실제 어떤 선언과 연결되는지를 바인딩
- 예를 들어 `add()`라는 함수가 있다면 실제로 어디서 선언됐는지 찾아서 연결
- 나중에 Tree Shaking을 할 때 "이 변수는 어디서 사용됐나? 이 함수는 참조되나?"를 명확하게 판단하기 위해서


#### 3. Tree Shaking 진행 (`includeStatements()`)

```javascript
// Graph.ts
this.includeStatements();
```

이 단계에서는 Rollup이 각 모듈을 순회하면서, 실제로 사용된 코드만 선택적으로 포함시키는 **Tree Shaking이 본격적으로 실행**됩니다.

**핵심 과정:**
- 각 모듈을 순회하면서 실제 사용된 코드만 선택적으로 포함
- `module.include()` 함수가 개별 모듈의 Program을 순회
- 필요한 노드만 `included = true`로 표시

```javascript
// Module.ts
include(): void {
  if (this.ast!.shouldBeIncluded(context)) {
    this.ast!.include(context, false);
  }
}

// Program.ts - include 메서드는 내부 AST 노드들을 재귀적으로 순회
include(context: InclusionContext): void {
  this.included = true;
  for (const node of this.body) {
    if (node.shouldBeIncluded(context)) {
      node.include(context, includeChildrenRecursively);
    }
  }
}
```

---

## 정리

### 핵심 내용

1. **번들러의 필요성과 사용 이유**
   - 전역 변수 충돌 방지
   - 네트워크 요청 최적화
   - 의존성 관리 자동화
   - 개발 환경과 배포 환경의 분리

2. **Tree Shaking의 구체적인 원리**
   - 정적 분석(Static Analysis) 기반
   - ES6 모듈 시스템 의존성
   - Dead code 제거를 통한 번들 크기 최적화

3. **Tree Shaking의 구체적인 구현 방식 (Rollup 기준)**
   - **1단계**: 모듈 로딩 & AST 생성
   - **2단계**: 모듈 정렬 & 참조 바인딩
   - **3단계**: 사용된 코드만 선별적 포함

---

## Q&A
### Q. 팀 내 프로젝트의 코드베이스에서 사이드이펙트가 없다는 걸 감지/확신하려면?
``` 사이드이펙트가 있는지 판단하려면 파일 내용을 직접 확인 해야한다. 
빌드 결과를 webpack-bundle-analyzer 같은 도구로 분석하면  
의도치 않게 코드가 빠졌는지 확인하는데 도움이 될 수 있다.
팀 내에서는 사이드이펙트가 있는 파일을 예외로 잘 정리하고,  
관련 기준을 공유해서 코드리뷰 때도 확인하고,
빌드 결과를 종종 점검하는 방식으로 신뢰도를 높이는 게 좋을 것 같다.
```
---

## 📚 참고자료

- [Tree Shaking - MDN Web Docs](https://developer.mozilla.org/en-US/docs/Glossary/Tree_shaking)
- [Frontend Fundamentals - Bundling Overview](https://frontend-fundamentals.com/bundling/overview.html)
- [Rollup.js Official Documentation](https://rollupjs.org/)
- [Webpack Tree Shaking Guide](https://webpack.js.org/guides/tree-shaking/)
- [AST Explorer](https://astexplorer.net/)

---

제가 틀린 내용이 있거나 궁금한 점이 있다면 편하게  ISSUE 주시면 감사하겠습니다!!
---

<div align="center">
🌳 Tree Shaking! 🌳