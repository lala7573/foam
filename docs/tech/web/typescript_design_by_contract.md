# typescript: design by contract

compilation time에 integer range를 제한하거나, string length를 제한하는 타입이 있는지 찾아봤다.

compilation time에 제한하는 방법은 없는 듯 하다. runtime에 체크하려면 아래와 같이 사용하거나, 이 글의 최 하단에 Subsitution을 항목을 참조하면 될 듯 하다.

[https://www.bussieck.com/typescript-types-with-complex-properties/](https://www.bussieck.com/typescript-types-with-complex-properties/)

opaque type이 뭘까?

- [https://en.wikipedia.org/wiki/Design_by_contract](https://en.wikipedia.org/wiki/Design_by_contract)
- [https://github.com/microsoft/TypeScript/issues/198](https://github.com/microsoft/TypeScript/issues/198)

  This issue was closed.

  ```
  This is very much outside our modern design goals. 6 Jun 2020
  ```

- Substitution
  - [https://github.com/pelotom/runtypes](https://github.com/pelotom/runtypes)
  - [https://github.com/codemix/babel-plugin-contracts](https://github.com/codemix/babel-plugin-contracts)
