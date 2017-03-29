* [单元测试](#单元测试)
* [UI测试](#ui测试)

### 单元测试

 1. 创建一个被测试类，不要在意它的实现

``` java
public class Calculator {

    public double sum(double a, double b) {
        return 0;
    }

    public double substract(double a, double b) {
        return 0;
    }

    public double divide(double a, double b) {
        return 0;
    }

    public double multiply(double a, double b) {
        return 0;
    }
}
```


 2. Android Studio提供了一个快速创建测试类的方法。只需在编辑器内右键点击Calculator类的声明，选择Go to > Test，然后"Create a new test…", 在打开的对话窗口中，选择JUnit4和"setUp/@Before"，同时为所有的计算器运算生成测试方法

``` java
import org.junit.Before;
import org.junit.Test;

import static org.junit.Assert.*;

public class CalculatorTest {

    private Calculator mCalculator;

    @Before
    public void setUp() throws Exception {
        mCalculator = new Calculator();
    }

    @Test
    public void testSum() throws Exception {
        //expected: 6, sum of 1 and 5
        assertEquals(6d, mCalculator.sum(1d, 5d), 0);
    }

    @Test
    public void testSubstract() throws Exception {
        assertEquals(1d, mCalculator.substract(5d, 4d), 0);
    }

    @Test
    public void testDivide() throws Exception {
        assertEquals(4d, mCalculator.divide(20d, 5d), 0);
    }

    @Test
    public void testMultiply() throws Exception {
        assertEquals(10d, mCalculator.multiply(2d, 5d), 0);
    }
}
```

 3. 终于到运行测试的时候了！右键点击CalculatorTest类，选择Run > CalculatorTest,然后发现输出显示四个测试都失败了，可以修改Calculator的代码让测试通过

``` java
public class Calculator {

    public double sum(double a, double b) {
        return a + b;
    }
    ...
}
```

### UI测试

 1. 创建一个简单的可以交互的待测试UI
 

``` xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools" android:layout_width="match_parent"
    android:layout_height="match_parent" android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:paddingBottom="@dimen/activity_vertical_margin" tools:context=".MainActivity">

    <TextView
        android:id="@+id/textView"
        android:text="Hello_world" android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
    <EditText
        android:hint="Enter your name here"
        android:id="@+id/editText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@+id/textView"/>
    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Say hello!"
        android:layout_below="@+id/editText"
        android:onClick="sayHello"/>
</RelativeLayout>
```


 2. 在androidtest包下创建一个MainActivityInstrumentationTest.java
 
``` java
package android.com.textdemo;

import android.support.test.rule.ActivityTestRule;
import android.support.test.runner.AndroidJUnit4;
import android.test.suitebuilder.annotation.LargeTest;

import org.junit.Rule;
import org.junit.Test;
import org.junit.runner.RunWith;

import static android.support.test.espresso.Espresso.onView;
import static android.support.test.espresso.action.ViewActions.click;
import static android.support.test.espresso.action.ViewActions.closeSoftKeyboard;
import static android.support.test.espresso.action.ViewActions.typeText;
import static android.support.test.espresso.assertion.ViewAssertions.matches;
import static android.support.test.espresso.matcher.ViewMatchers.withId;
import static android.support.test.espresso.matcher.ViewMatchers.withText;

@RunWith(AndroidJUnit4.class)
@LargeTest
public class MainActivityInstrumentationTest {

    private static final String STRING_TO_BE_TYPED = "Peter";

    @Rule
    public ActivityTestRule<MainActivity> mActivityRule = new ActivityTestRule<>(
        MainActivity.class);

    @Test
    public void sayHello(){
        //找到ID为editText的view，输入Peter，然后关闭键盘；
        onView(withId(R.id.editText)).perform(typeText(STRING_TO_BE_TYPED), closeSoftKeyboard()); 
        //点击Say hello!的View，我们没有在布局的XML中为这个Button设置id，因此，通过搜索它上面的文字来找到它
        onView(withText("Say hello!")).perform(click()); 
       //将TextView上的文本同预期结果对比，如果一致则测试通过
        String expectedText = "Hello, " + STRING_TO_BE_TYPED + "!";
        onView(withId(R.id.textView)).check(matches(withText(expectedText)));
    }

}
```


 3. 右键点击域名运行测试，选择Run>MainActivityInstrume...（第二个带Android图标的），这样就会在模拟器或者连接的设备上运行测试，你可以在手机屏幕上看到被执行的动作

