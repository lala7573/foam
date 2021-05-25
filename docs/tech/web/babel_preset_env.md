# @babel/preset-env

- [https://babeljs.io/docs/en/babel-preset-env](https://babeljs.io/docs/en/babel-preset-env)
- syntax transforms에 대한 세세한 설정없이 최신의 js를 쓸 수 있게 해줌

### browserlist integration

- 기본적으로 @babe/preset-env는 targets나 ignoreBrowserslistConfig 옵션이 설정되어있지 않는한 browserlist 를 사용함
- browser-이나 electron 기반의 프로젝트라면 .browserlistrc 파일을 사용한느 것을 추천

### loose [default: false]

- 만약에 사용한다면 babel 7.13부터 지원하는 [https://babeljs.io/docs/en/assumptions](https://babeljs.io/docs/en/assumptions) 으로 넘어가는 것을 고려해 볼 것
- [https://2ality.com/2015/12/babel6-loose-mode.html](https://2ality.com/2015/12/babel6-loose-mode.html)
  - 많은 babel 플러그인들은 2가지 모드가 있음
    - ES6와 최대한 비슷한 시맨티을 따르는 일반적인 모드
    - 단순한 ES5코드를 만드는 loose 모드
  - 일반적으로 loose 모드를 쓰지 않는 것을 권장함
    - 장점: 생성된 코드가 잠재적으로 더 빠르고, 오래된 엔진에 호환성이 더 있다.
    - 단점: 트랜스파일된 ES6코드에서 native ES6로 바꿀때 문제가 생길수 있다. (⇒?)

### modules [default: "auto"]

- "amd" | "umd" | "systemjs" | "commonjs" | "cjs" | "auto" | false
- cjs == commonjs
- false로 세팅하면 ES module를 보존함. native ES module을 브라우저에 바로 보내고 싶다면 이 옵션을 쓸 것. babel을 번들러로 사용하고 있다면 "auto" 옵션이 항상 선호됨.

  - @babel/preset-env 는 es module와 module 기능들이 변경되어야하는지 판단하기 위해 caller데이터를 씀

    - [https://babeljs.io/docs/en/options#caller](https://babeljs.io/docs/en/options#caller)

    ```
    interface CallerData {
      name: string;
      supportsStaticESM?: boolean;
      supportsDynamicImport?: boolean;
      supportsTopLevelAwait?: boolean;
      supportsExportNamespaceFrom?: boolean;
    }

    Copy
    ```

    - 일반적으로 caller 데이터는 번들러 플러그인(babel-loader, @rollup/plugin-babel)에 의해서 정해지기 때문에 직접 변경하는건 추천하지 않음

### debug [default: false]

- preset-env에 켜져있는 polyfills와 transform plugins을 출력함

### useBuiltIns [default: false]

- "usage" | "entry" | false
  - usage: 파일마다 polyfill을 위한 구체적인 import를 추가하라는 옵션. 번들러는 같은 polyfill을 한번만 로딩하는게 장점임 (?)
  - entry: 환경에 따라서 필요한 플러그인들을 entrypoint 파일의 import "core-js"부분을 대체함
  - false: 파일마다 polyfills를 자동적으로 추가하지 말라는 옵션
- usage나 entry를 쓰면 base import(require)를 씀. 즉 core-js가 상대경로를 쓰고 접근가능해야한다.
- @babel/polyfill 은 7.4.0 부터 deprecated되었기 때문에, core-js세팅을 하는 것을 추천함

### corejs [default: "2.0"]

- string | { version: string, proposals: boolean }
- 7.4.0에 추가
- useBuiltIns가 entry이거나 usage일때만 유효함
- 기본적으로 stable ESMAscript 기능의 polyfill들만 주입됨. proposal을 사용하고 싶으면 3가지 옵션이 있다

  - useBuildIns: "entry" 인 경우, 직접 proposal polyfills를 추가함

    `import "core-js/proposals/string-replace-all"`

  - useBuiltIns: "usage" 인 경우 2가지 대안이 있음
    - shippedProposals: true로 설정. 브라우저에 이미 shipped된 proposal의 polyfills 추가
    - { version: "3.8", proposals: true } 해당 버전이 지원하는 모든 proposal의 polyfills 추가
