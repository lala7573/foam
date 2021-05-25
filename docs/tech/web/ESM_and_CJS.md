# ESM, CJS

- [https://redfin.engineering/node-modules-at-war-why-commonjs-and-es-modules-cant-get-along-9617135eeca1](https://redfin.engineering/node-modules-at-war-why-commonjs-and-es-modules-cant-get-along-9617135eeca1)
  - [https://yceffort.kr/2020/08/commonjs-esmodules](https://yceffort.kr/2020/08/commonjs-esmodules) 한글 번역. 뒷부분이 좀 생략되어 있다.
- commonJs에서 require는 동기적으로 동작한다. promise나 callback을 리턴하는 것이아니라, require는 디스크(혹은 심지어 네트워크)에서 코드를 읽고, 바로 스크립트를 실행한다. 이 스크립트는 IO나 다른 side effect를 실행하고, 반환값은 module.exports에 저장한다.
- ESM에서는 모듈 로더가 비동기적으로 동작한다. 모듈 로더는 파싱을 통해서 import와 export 호출을 감지한다. 이때 스크립트를 실행하거나 import하지 않는다. 파싱하는 동안에, ESM loader는 named import에 있는 오타를 즉시 감지하고 예외를 던진다. 이 과정에서 의존 코드를 실행하는 작업은 일어나지 않는다. ESM 모듈 로더는 비동기적으로 스크립트를 다운로드하고 파싱한다. 그리고 아무것도 import하지 않는 스크립트를 찾을때까지 디펜던시들의 module graph 를 구성한다.최종적으로 스크립트는 실행가능하게 된다.
- ESM은 Javascript의 많은 것을 변화시켰다. ESM 스크립트를 사용하면 기본적으로 use strict를 사용하고, this 가 전역객체를 참조하지 않고 scoping이 다르게 동작한다. 이것은 브라우저단에서 <script> 태그가 ESM 이 아니기 때문이다. ESM mode를 선택하려면 type="module" 을 추가해주어야 한다.
- CJS에서 ESM으로 넘어가는 것은 하위호환성 측면에서 문제가 될 것 같다. 참고로, Deno에서는 ESM을 디폴트로 가져간다. 결과적으로 바닥부터 생태계가 변화하고 있다.
- CJS 가 EMS 모듈을 require할 수 없는 이유는, ESM이 top-level await를 지원하는데 반해, CJS는 그렇지 않기 때문이다. Top-level await란 최상위단에서 async function 밖에서 await 키워드를 사용하게 해주는 문법이다. [[tc39/proposal-top-level-await]](https://github.com/tc39/proposal-top-level-await)
- ESM의 multi-phase 로더는 ESM이 top-level await을 가능하게 만들어준다. ESM의 top-level await 기능에 관해 [Rich Harris는 몇가지 이유를 들며 ESM의 top-level await이 footgun이 될 수 있으므로 이 기능을 구현하지 말 것을](https://gist.github.com/Rich-Harris/0b6f317657f5167663b493c722647221) 제안하였고, 이는 proposal stage3에서 직접적으로 이 이슈를 언급하였다. 그에 대해 v8 블로그에서는 이렇게 정리하고 있다. 그리고 Rich는 현재의 top-level await 구현을 인정했다.
  - 우선 Rich Harris가 지적한 문제는,
    - Top-level await가 코드 실행을 블로킹할 수 있다
    - Top-level await가 리소스를 가져오는 것을 블로킹 할 수 있다.
    - commonjs를 위한 명확한 상호 운용성 이야기가 없다.
  - proposal stage 3 버전에서는 이 이슈를 직접적으로 언급한다
    - siblings들이 실행가능하기 때문에, 결정적인 블로킹은 없다
    - Top-level await는 module graphe를 실행하는 과정에서 일어나게 된다. 이 시점에서 모든 리소스들은 이미 fetch되어있고 link되어있다. 리소스를 가져오는 것을 블로킹할 위험이 없다.
    - Top-level await는 ESM 모듈에 제한된다. scripts나 CommonJs 모듈을 위한 지원은 없다.
- CJS에서는 top-level await를 구현하지 않기 때문에 ESM top-level await를 CJS로 transpile을 하는 것도 불가능하다. 대부분의 ESM 스크립트는 top-level await를 사용하고 있지 않다.
- [Nodejs에서 ESM을 어떻게 require할 것인가에 대한 논의](https://github.com/nodejs/modules/issues/454)가 활발하게 진행중이다. 다 읽어보면 require ESM이 금방 가능해보이지 않는다.
- CJS에서 ESM을 import할 수 있지만 좋지는 않다. 단순히 import할 때는 괜찮아 보이지만 export를 해야하는 상황이라면 Promise를 export해야하는 상황이 생긴다. 당신의 라이브러리를 사용하는 유저에게 큰 불편함이 될 것이다.

  ```
  (async () => {
      const {foo} = await import('./foo.mjs');
  })();

  module.exports.foo = (async () => {
      const {foo} = await import('./foo.mjs');
      return foo;
  })();
  ```

- ESM은 CJS 스크립트가 순서대로 실행되지 않는 한 CJS의 named exports를 import할 수 없다. 왜냐하면 CJS 스크립트는 named exports를 실행 중에 연산하기 때문이다. ESM의 named exports는 반드시 parsing 단계에서 연산되어야한다. 다행히 차선책이 있다. 이 코드에 대한 단점은 없고, ESM을 인식하는 CJS 라이브러리는 이 boilerplate를 포함하는 고유의 ESM wrapper를 제공할 수 있다.

  ```
  import _ from './lodash.cjs';
  const {shuffle} = _;
  ```

- 비순차 실행은 동작하지만 더 나쁠 수 있다. 많은 사람들이 비순차적으로 ESM을 import하기전에 CJS를 import하는 방식을 제안했다. 이 방식에서 CJS의 naemd exports는 ESM의 named exports와 동시에 연산될 수 있다. 그러나 이 방식은 [새로운 문제](https://github.com/nodejs/modules/issues/81#issuecomment-391348241)를 야기한다.

  ```
  import {liquor} from 'liquor';
  import {beer} from 'beer';
  ```

  liquor, beer가 둘 다 CJS이었다가, liquor를 CJS에서 ESM으로 변경하게되면 import의 순서를 liquor, beer에서 beer, liquor로 바뀌게 된다. 비순차 실행은 여전히 논란 중이다. [[node/modules: Rediscussion for out-of-order CJS exec]](https://github.com/nodejs/modules/issues/509)

- Dynamic Module은 비순차 실행이나 wrapper script를 대체할 수 있는 proposal이다. 그러나 start exports 구문은 거절당했고 더이상 진행되지 않고 있다. (Dynamic Modules 이슈는 닫혔고, 더이상의 토론은 진행되지 않는다.)
  - ESM 스펙에서, exporter는 정적으로 모든 named export를 정의한다. dynamic module에서는 importer가 import에서 export names를 정의한다. ESM loader는 초기에 dynamic modules(CJS scripts)가 요구된 모든 named exports를 제공할것이라고 믿는다. 그리고 이후에 이 조건이 만족되지 않는 경우 예외를 던지게 된다.
    안타깝게도, 이 기능은 TC39 에 의해 승인을 받아야하는 JS 언어적 변화가 필요했는데, 승인되지 않았다.  
    구체적으로, ESM 스크립트는 foo 가 export하고 있는 모든 이름을 re-export하는 `export export * from './foo.cjs'` 구문을 사용할 수 있다. (이 것을 "start exports" 라고 부른다.)
    dynamic module에서 start export를 할 때 loader가 무엇이 export될지 알 방법이 없다.
    또, dynamic module start exports는 스펙 상의 compliance 이슈를 만든다. 예를 들면 `export * from 'omg'; export * from 'bbq';` 이 동시에 `wtf` 이라는 named export를 사용할 경우이다.
    dynamic modules의 발의자는 dynamic modules에서 star exports를 금지하도록 제안했지만 TC39는 이 proposal을 거절했다. TC39 멤버 중 하나는 이 proposal을 "syntax poisoning" 이라고 언급했다. start exports는 dynamic module에 의해 "poisoned"되기 때문이다.
    이것은 dynamic modules 여정의 끝이 아닐지 모른다. 모든 node modules가 dynamic module이 되어야 한다는 proposal이 있었다. 심지어 순수한 ESM module마저도 ESM multi-phase loader를 버리면서 말이다. 놀랍게도 이것은 조금 나쁜 startup 성능을 제외하고는 사용자 입장에서 보이는 효과가 없다. ESM multi-phase loader는 느린 네트워크상에서 스크립트를 로딩하도록 설계되어있다.
    dynamic modules에 대한 github issue는 닫혔고, 그 이후에 dynamic modules에 대한 토론은 없다.
- 하나의 아이디어가 하나 더있다. [CJS 모듈을 파싱하여 exports를 탐지하는데 최대 노력의 시도](https://github.com/nodejs/node/pull/33416)를 하는 것이다. 그러나 이 접근 방법은 100%로 동작할 수 없다. (마지막 PR은 npm 상의 상위 1000개의 모듈 중 62%에서만 동작했다) 휴리스틱은 신뢰할 수 없기때문에, 몇몇의 노드 모듈 그룹은 이에 반대하고 있다.
- ESM은 require할 수 있다. 그렇지만 가치가 없을지도.. ESM에서 require는 default scope에 있지 않다. 그렇지만 [매우 쉽게 쓸 수 있다](https://nodejs.org/api/modules.html#modules_module_createrequire_filename).

  ```
  import { createRequire } from 'module';
  const require = createRequire(import.meta.url);

  const {foo} = require('./foo.cjs');
  ```

  그렇지만 이 접근법의 문제는 별로 도움이 안된다는 것이다. 아래처럼 쓰는것에 비해 더 많은 코드를 가지고 있을 뿐만 아니라, webpack, rollup같은 번들러가 createRequire 패턴으로 무엇을 해야할 지 모른다.

  ```
  import cjsModule from './foo.cjs';
  const {foo} = cjsModule;
  ```

### CJS와 ESM를 둘다 포함하는 Dual package를 만드는 좋은 방법

1. CJS 버전을 제공하라
   - CJS 유저에게 편의를 제공하며, 오래된 버전의 Node 버전에서 동작한다는 것을 확실하게 한다. (만약 js 로 transpile해야하는 Typescript나 다른 언어를 사용하고 있다면 CJS로 transpile하라)
   - 만약 라이브러리가 default export (unnamed export)를 제공하고 있다면 이로써 끝이다. named export를 제공하고 있다면 아래를 더 읽으라
2. CJS named export를 위한 ESM 를 제공하라

   - CJS 라이브러리를 위한 ESM wrapper 를 작성하는 것은 쉽지만 ESM 라이브러리를 위한 CJS wrapper를 작성하는 것은 불가능하다.

   ```
   import cjsModule from '../index.js';
   export const foo = cjsModule.foo;
   ```

   - esm wrapper를 esm 하위 디렉토리에 두고, package.json에 {"type":"module"}를 추가한다. wrapper 파일명을.mjs로 바꿀 수도 있다. 이것은 Node 14에서 잘 동작한다. 하지만 어떤 툴은 mjs파일에서 동작하지 않기 때문에 글 작성자는 subdirectory를 사용하는 것을 선호한다고 한다.
   - double transpiling을 피하라. 만약 Typescript에서 transpile한다면, CJS나 ESM으로 transpile할 수 있다. 그러나 이것은 유저가 실수로 동시에 ESM 스크립트를 import하고, CJS를 require할 수 있다.
     Node는 일반적으로 모듈을 dedupe하지만, CJS와 ESM이 같은 파일인지 알 수 없다. 그러므로 코드는 두 번 실행될 것이고, 라이브러리의 상태가 2개의 copy로 유지된다. 이것 온갖 종류의 이상한 버그의 원인이 된다.

3. package.json에 exports 맵을 추가하라

   ```
   "exports": {
       "require": "./index.js",
       "import": "./esm/wrapper.js"
   }
   ```

   - exports를 추가하는 것은 항상 sematic major 주요 변경 사항이다. 기본적으로 사용자는 require로 라이브러리 작성자가 내부적으로만 사용하도록 의도한 파일조차 접근할 수 있다. exports는 의도적으로 노출한 entrypoint 들만 require/import 할 수 있다.

- footgun 자기 발에 총쏘는거
- big break
- the last nail in the coffin 관의 마지막 못. 이미 실패하기 시작한 일의 실패를 야기하는 이벤트
- interop ⇒ (computing) Interoperability 정보처리 상호 운용
