# 16장 SerialDate 리팩터링
[JCommon - SerialDate](https://github.com/jfree/jcommon/blob/master/src/main/java/org/jfree/date/SerialDate.java)  
[JCommon - SerialDateTest](https://github.com/jfree/jcommon/blob/master/src/test/java/org/jfree/date/SerialDateTest.java)

![image](https://user-images.githubusercontent.com/50076031/138548118-5869300c-6e40-47e0-846e-c68474ff40f1.png)


## 고쳐보자

### import
- import 문 와일드카드(*)를 사용하자

```java
// As-Is
import java.io.Serializable;
import java.text.DateFormatSymbols;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.GregorianCalendar;

// To-Be
import java.io.Serializable
import java.text.*;
import java.util.*;
```

- Q) 와일드카드(*)를 사용하는게 좋은 방법일까?

![image](https://user-images.githubusercontent.com/50076031/138548901-e5a2d3c8-6de5-4c6f-8268-59a3f902d3f3.png)

- 이런 예제는?

![image](https://user-images.githubusercontent.com/50076031/138549275-43a450ec-947c-4a9c-9fc8-d731716535c9.png)

https://stackoverflow.com/questions/147454/why-is-using-a-wild-card-with-a-java-import-statement-bad

<br>

### MonthConstants
- 상수 모음인 MonthConstants는 enum으로 변경하자.

![image](https://user-images.githubusercontent.com/50076031/138549086-5151cd97-d2e3-4b6f-b548-f4dc2634020c.png)

<img width="" height="840" alt="캡쳐1" src="https://user-images.githubusercontent.com/50076031/138549091-9b32726d-32ba-498b-bc57-e7c70ae4c756.png">

[이펙티브자바 아이템 22 - 인터페이스는 타입을 정의하는 용도로만 사용하라](https://github.com/Meet-Coder-Study/book-effective-java/blob/main/4%EC%9E%A5/22.%20%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4%EB%8A%94%20%ED%83%80%EC%9E%85%EC%9D%84%20%EC%A0%95%EC%9D%98%ED%95%98%EB%8A%94%20%EC%9A%A9%EB%8F%84%EB%A1%9C%EB%A7%8C%20%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC_%EC%9D%B4%EC%A3%BC%ED%98%84.md#item-22-%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4%EB%8A%94-%ED%83%80%EC%9E%85%EC%9D%84-%EC%A0%95%EC%9D%98%ED%95%98%EB%8A%94-%EC%9A%A9%EB%8F%84%EB%A1%9C%EB%A7%8C-%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC)

<br>

### 주석 제거
![image](https://user-images.githubusercontent.com/50076031/138549501-37526dd9-0560-4528-bce9-764152b7802f.png)

### 상수 -> enum(162라인 ~ )

```java
/* As-Is */
    /** A useful constant for referring to the first week in a month. */
    public static final int FIRST_WEEK_IN_MONTH = 1;

    /** A useful constant for referring to the second week in a month. */
    public static final int SECOND_WEEK_IN_MONTH = 2;

    /** A useful constant for referring to the third week in a month. */
    public static final int THIRD_WEEK_IN_MONTH = 3;

    /** A useful constant for referring to the fourth week in a month. */
    public static final int FOURTH_WEEK_IN_MONTH = 4;

    /** A useful constant for referring to the last week in a month. */
    public static final int LAST_WEEK_IN_MONTH = 0;
    
        ...

/* To-Be */
public enum WeekInMonth {
    FIRST(1), SECOND(2), THIRD(3), FOURTH(4), LAST(0);
    
    public final int index;
    
    WeekInMonth(int index) {
        this.index = index;
    }
}
```

<br>

### 네이밍 & 중복제거(242라인 ~ )
- 파라미터의 s 변수를 좀 더 적합한 변수명으로 변경
- if문 동일한 로직 -> || 연산자 사용
- 조건문 캡슐화(JUnit 들여다보기, 333p)

```java
    public static int stringToWeekdayCode(String s) {
    
        ...
        
        for (int i = 0; i < weekDayNames.length; i++) {
            if (s.equals(shortWeekdayNames[i])) {
                result = i;
                break;
            }
            if (s.equals(weekDayNames[i])) {
                result = i;
                break;
            }
            
            // 1) if문 동일한 로직 -> || 연산자 사용
            if (s.equals(shortWeekdayNames[i]) || s.equals(weekDayNames[i])) {
                result = i;
                break;
            }
            
            // 2) 조건문 캡슐화(JUnit 들여다보기, 333p)
            if (isWeekdayNameIgnoreShort(s, shortWeekdayNames[i], weekDayNames[i])) {
                result = i;
                break;
            }
        return result;
    }
        
    private static boolean isWeekdayNameIgnoreShort(String original, String shortName, String name) {
            return s.equals(shortName) || s.equals(name);
        }
    }
```

<br>

### 파라미터로 플래그(boolean) 넘기지 말자
```java
    public static String monthCodeToString(final int month, 
                                           final boolean shortened) {

        // check arguments...
        if (!isValidMonthCode(month)) {
            throw new IllegalArgumentException(
                "SerialDate.monthCodeToString: month outside valid range.");
        }

        final String[] months;

        if (shortened) {
            months = DATE_FORMAT_SYMBOLS.getShortMonths();
        }
        else {
            months = DATE_FORMAT_SYMBOLS.getMonths();
        }
        return months[month - 1];
    }

    public static String monthCodeShortToString(final int month) {
        ... 
        
        months = DATE_FORMAT_SYMBOLS.getShortMonths();
    }
    
    public static String monthCodeToString(final int month) {
        ... 
        
        months = DATE_FORMAT_SYMBOLS.getMonths();
    }
```

<br>

### 서술적인 표현으로 가독성 높이기
```java
    public static boolean isLeapYear(final int yyyy) {
        if ((yyyy % 4) != 0) {
            return false;
        }
        else if ((yyyy % 400) == 0) {
            return true;
        }
        else if ((yyyy % 100) == 0) {
            return false;
        }
        else {
            return true;
        }
    }
    
    /* 더 헷갈리는 것 같은데 .. */
    public static boolean isLeapYear(int year) {
        final boolean fourth = year % 4 == 0;
        final boolean hundredth = year % 100 == 0;
        final boolean fourHundredth = year % 400 == 0;
        return fourth && (!hundredth || fourHundredth);
    }
    
    public static boolean isLeapYear(int year) {
        if (year % 4 != 0) return false;
        if (year % 100 == 0) return false;
        if (year % 400 == 0) return true;
        return true;
    }   
```

<br>

### 리팩터링 정리
- 주석 제거 & 개선
- 네이밍 & 중복 제거
- 인터페이스 상수 등과 같은 부분 enum 파일로 분리
- 파라미터로 플래그(boolean) 넘기는 메소드 분리
