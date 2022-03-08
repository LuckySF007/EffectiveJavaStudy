# ITEM 08. **try-finally 보다는 try-with-resources를 사용하라.**

자바에는 `close`메서드를 통해 직접 닫아줘야하는 자원들이 많다.  
이는 클라이언트가 놓치기 쉬워 예측못한 성능 문제로 이어지는 경우도 있다. 이런 자원들 중 상당수가 안전망으로 `finalizer`를 활용하고는 있지만, 그리 믿을만 하지 못하다.(Item 08)  

그래서 제대로 닫힘을 보장하는 수단으로 try-finally가 쓰였다.
```java
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readline();
    } finally {
        br.close();
    }
}
```
위 방법도 나쁘지 않지만, 닫아줘야하는 자원이 늘어나면 아래 코드처럼 된다.
```java
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```

위의 코드 같은경우 기기에 물리적인 문제가 생긴다면 `firstLineOfFile`메서드 안의 `readLine`메서드가 예외를 던지고, 같은 이유로 `close`메서드도 실패할 것이다. 이러한 상황이면 두번째 예외가 첫 번째 예외를 완전히 먹어버린다. 이러면 첫번째 예외에 대한 정보가 남지 않아 디버깅에 어려움을 겪는다.

이러한 문제들은 자바 7에 등장한 **try-with-resources**를 통하여 해결되었다. 이 구조를 사용하려면, `AutoCloseable` 인터페이스를 구현해야 한다. 

try-with-resources를 사용하여 위의 코드들을 작성하였다.

```java
/*          code 1          */
static String firstLineOfFile(String path) throws IOException {
    try ( BufferedReader br = new BufferedReader(
            new FileReader(path))){
        return br.readline();
    }
}

/*      code 2          */
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
        OutputStream out = new FileOutputStream(dst)){
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
    }
}
```

try-with-resources를 사용하였을 경우가 읽기도 쉽고, 문제 진단도 좋다.  
보통의 try-finally 문 처럼 try-with-resources에서도 catch절을 사용이 가능하다.