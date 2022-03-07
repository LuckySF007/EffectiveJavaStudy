# Item09. try-finally 보다는 try-with-resources를 사용하라!

- 자바 라이브러리에는 close 메서드를 호출해 직접 닫아줘야 하는 자원이 많다.
- InputStream, OutputStream, java.sql.Connection 등이 좋은 예이다.
- 자원 중 상당수가 안전망으로 finalizer를 사용하고 있으나 아이템8에서도 확인한 것처럼 그리 좋지 못하다.



## try-finally

- 자원을 회수하는 하나의 방식이다.
- finally 구문에서 close를 이용해 자원을 회수한다.
- 하지만 자원이 많아지면 코드가 너무 지저분해지고 가독성 떨어진다.
- 또 여러 에러가 발생하는 경우 예외를 처리하는, 즉 디버깅하는 것이 쉽지 않다.



## try-with-resources

- AutoCloseable 인터페이스 구현이 필요하다. 이 인터페이스는 단순히 close 메서드 하나만 정의한 것이다.
- 해당 구문을 사용 시 코드의 가독성이 상당이 높아진다. 또 문제의 진단도 수월해진다.
- try-finally 에서처럼 catch 구문 사용도 가능하다. 이를 통해 다수의 예외처리도 손쉽게 가능하다.



## 정리

1. 꼭 회수가 필요한 자원을 다룰 때는 try-finally 말고, try-with-resources를 사용하자!
2. try-with-resources 사용 시, 코드가 짧고 간결해지며 만들어지는 예외 정보 또한 훨씬 유용하다.

