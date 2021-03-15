# 4. Middleware
`Schema#pre()`와 `Schema#post()` 함수를 이용해서 미들웨어를 등록할 수 있다. pre는 쿼리 실행 전, post는 쿼리 실행 후에 동작하게 된다.
Error handling 미들웨어는 error가 발생할 때만 post() 미들웨어의 특수 타입이다. 3개의 파라미터를 처리하는 함수를 2번째 인자로 주면 error handling 미들웨어로 동작한다. 로깅하거나 에러 메세지를 더 읽기 좋게 만들 수 있다.

## 종류
- Document middleware
- Model middleware
- Aggregation middleware
- Query middleware

  You register middleware using the Schema#pre() and   functions. Mongoose executes pre() middleware before the wrapped function, and   middleware after the wrapped function.
Error handling middleware is a special type of post() middleware that only executes if an error occurs. Error handling middleware is useful for logging errors and making error messages more human readable.