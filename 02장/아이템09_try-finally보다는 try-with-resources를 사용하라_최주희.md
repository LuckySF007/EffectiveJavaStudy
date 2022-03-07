# 아이템09.try-finally보다는 try-with-resources를 사용하라

자바 라이브러리에는 close 메서드로 직접 닫아줘야하는 자원이 많다.

ex) InputStream, OutputStream, java.sql.Connection

자원 닫기는 예측할 수 없는 성능 문제로 이어질수있다.

<br>

## try-finally

전통적으로 자원을 닫는 수단으로 사용하였다.

예외가 발생하거나 메서드에서 반한되는 경우를 포함해서 말이다.

```java
//자원이 하나인 경우
static String firstLineOfFile(String path) throws IOException {
	BufferedReader br = new BufferedReader(new FileReader(path));
	try {
		return br.readLine();
	} finally {
		br.close();
	}
}
```

```java
//자원이 둘 이상인 경우
static void copy(String src, String dst) throws IOException {
	InputStream in = new FileInputStream(src);
	try {
		OutputStream out = new FileOutputStream(dst);
		try {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while ((n = in.read(buf)) >= 0)
				out.write(buf, 0, n);
		} finally {
			out.close();
	} finally {
		in.close();
	}
}
```

- 두번째 코드 경우에 자원이 둘 이상이기때문에 코드가 매우 복잡해짐을 볼 수 있다.
- 첫번째 코드 경우에 자원이 하나라도 문제점이 생길 수 있다.
만약, readLine 메서드가 기기의 물리적인 문제로 예외를 던진다면 close 메서드도 실패할 것이다.
이렇게 되면 두번째 예외가 첫번째 예외를 덮어버리며, 첫번째 예외에 관한 정보는 스택 추적 내역에 남지 않아 디버깅을 몹시 어렵게 한다.

<br>

## try-with-resources

위의 문제들을 해결하기 위해 자바7부터 생겼다.

이 구조를 사용하기 위해서는 그 자원이 AutoCloseable 인터페이스를 구현해야한다.

AutoCloseable은 close 메서드 하나 있는 인터페이스이다. (자바 라이브러리들이 이미 많이 구현해둠)

try-with-resources를 사용하면 try문이 종료될때 자동으로 close를 호출해준다.

```java
//자원이 하나인 경우
static String firstLineOfFile(String path) throws Exception {
	try (BufferedReader br = new BufferedReader (
		new FileReader(path))) {
			return br.readLine();
	}
}
```

```java
//자원이 둘 이상인 경우
static void copy(String src, String det) throws IOException {
	try (InputStream in = new FileInputStream(src);
		OutputStream out = new FileOutputStream(dst)) {
		byte[] buf = new byte[BUFFER_SIZE];
		int n;
		while ((n == in.read(buf)) > = 0)
			out.write(buf, 0, n);
}
```

위에 try-finally로 작성했던 코드를 try-with-resources로 바꾼 코드이다.

- 코드가 더 짧고 가독성이 좋으며 문제 진단에도 좋다.
- 여기서도 readLine 메서드와 close 메서드 둘다 예외가 난다면,
close에서 발생한 예외는 숨겨지고 readLine에서 발생한 예외는 기록된다. 
try-finally처럼 close 예외에 덮여지지 않는다.
- 숨겨진 예외도 버려지는게 아니라 스택 추적 내역에 숨겨졌다(suppressed) 꼬리표를 달고 출력된다.
- 자바7에서 Throwable에 추가된 getSuppressed 메서드를 이용하면 코드에도 가져올 수 있다.
- try-with-resources 에서도 try문을 중첩하지 않고도 catch문을 통해 다수의 예외를 처리할 수 있다.

<br>

## 결론

꼭 회수해야하는 자원을 다룰때는 try-finally 보다는 try-with-resources를 사용하자.

가독성도 더 좋으며 예외 정보를 알기에도 더 유용하며 쉽게 자원을 회수할 수 있다.