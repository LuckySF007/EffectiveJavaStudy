# 아이템 04_인스턴스화를 막으려거든 private 생성자를 사용하라

이따금 단순히 static 메소드와 static 필드 만을 담은 클래스를 만들고 싶을떄가 있다.

객체 지향적 사고로는 올바르진 않지만 쓰임새는 있다

java.lang.Math,   java.util.Arrays 또는 java.util.Collections등과 같은것이 예시이다.

마지막으로 final 클래스와 관련한 메서드들을 모아놓을 때도 사용한다.

위에서 언급한 것과 같은 클래스들은 **인스턴스로 만들어서 쓰려는 것이 아니다.**

하지만 생성자를 명시하지 않으면 컴파일러가 자동으로 기본생성자를 만들어준다.

추상 클래스로 만드는 것으로 인스턴스화를 막을 수 없다.

하위 클래스를 만들어 인스턴스화를 하면 하위클래스에서 super()가 호출이 되고 

자동으로 추상 클래스의 생성자 호출이 일어난다.

```java
import java.util.*;
import java.io.*;
public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));

        Mo test = new Suhyeok();
    }
}
abstract class Mo{
    private int x,y;

    public abstract void print();

//    public Mo(){
//        System.out.println("????");
//    }
}

class Suhyeok extends Mo{

    @Override
    public void print() {
        System.out.println("print");
    }
}
```

주석처리 된 부분이 컴파일러가 실행한다

컴파일러가 기본 생성자를 만드는 경우는 명시된 생성자가 없을때이니 해당 클래스를 인스턴스화 하기 싫다면 private 생성자를 추가하는 방법으로 클래스의 인스턴스화를 막을수 있다.

```java
public class UtilityClass{
    // 기본 생성자가 만들어지는 걸 막는다 (인스턴스화 방지)
    
    private UtilityClass(){
        throw new AssertionError();
    }
}
```

이 방식은 상속을 불가능하게 하는 효과도 있다.

자식 클래스가 부모의 생성자에 접근하려고 할 때 private이니 부모 자신만 접근이 가능하므로 불가능 하다.

예외처리는 해당 클래스안에서 실수로 생성자를 호출 할 수 도 있으니 안전하게 적어둔다,