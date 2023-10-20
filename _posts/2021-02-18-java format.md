---
layout:     post
title:      java format
subtitle:   JAVA Format 格式化
date:       2021-02-18
author:     zhaoeh
header-img: img/post-bg-swift.jpg
catalog: true
tags:
    - Java初级进阶
---

# 1. java中的format
有时候，我们需要对程序中的文本、数字、日期等类型进行某种格式化，以适应我们自身的阅读和显式习惯。  
在java中进行格式化的类主要是Format类。  
Format类主要有3个子类：  
1.  MessageFormat类（消息文本格式化类）。  
2.  NumberFormat抽象类（数字格式化类）。  
3.  DateFormat抽象类（日期格式化类）。  

# 2. MessageFormat
MessageFormat是Format的子类，在java.text包中，专门用来格式化字符串文本。  
其核心方法如下：  
```java
public static String format(String pattern, Object ... arguments);
```
其中第1个参数表示要操作的字符串文本，第2个参数表示输入参数可以是任意多个或者0个。  
该方法的核心功能就是格式化字符串，使用实际参数替换字符串文本中的占位符<b>{0}</b>之类的。  

案例代码如下：  
```java
package javase.demo18.classlibray.base;
import java.text.MessageFormat;

// MessageFormat进行文本格式化
public class Demo16StudyMessageFormat {
    public static void main(String[] args) {
        String target = "hello,{0}.my home is {1}!";
        System.out.println(target);
        String format = MessageFormat.format(target, "Eric", "西安");
        System.out.println(format);
        target = "hello,{2}.my home is {3}!";
        format = MessageFormat.format(target, "Eric", "西安");
        System.out.println(format);
    }
}
```
运行结果如下：  
```
hello,{0}.my home is {1}!
hello,Eric.my home is 西安!
hello,{2}.my home is {3}!
```
总结：  
1.  MessageFormat类的format()方法接收2个参数，第1个参数表示要格式化的目标字符串，第2个参数是个可变参数，表示传入的用于替换字符串中占位符的参数，可以是无数个或者0个。  
2.  字符串中的占位符使用形式必须遵守<b>{index}</b>的形式，其中index表示要占位符的索引，从0开始匹配。此处占位符必须从{0}，再到{1}，{2}...以此类推。占位符的索引下标必须和format()方法的可变参数下标匹配上才会进行format操作。否则对于不匹配的占位符将不会进行任何format。    

# 3. String类的format方法
String类本身也提供一个static的format()方法。定义如下：  
```java
public static String format(String format, Object... args) {
    return new Formatter().format(format, args).toString();
}
```
String类的format()方法用于创建格式化的字符串以及连接多个字符串对象。  
熟悉C语言的同学应该记得C语言的sprintf()方法，两者有类似之处。  
format()方法有两种重载形式:  
format(String format, Object... args) 新字符串使用本地语言环境，制定字符串格式和参数生成格式化的新字符串。  
format(Locale locale, String format, Object... args) 使用指定的语言环境，指定字符串格式和参数生成格式化的字符串。  
该方法用于显示不同转换符，实现不同数据类型到字符串的转换，如下所示：  
```
%s   字符串类型
%c   字符类型
%b   布尔类型
%d   整数类型（十进制）
%x   整数类型（十六进制）
%o   整数类型（八进制）
%f   浮点类型
%a   十六进制浮点类型
%e   指数类型
%g   通用浮点类型（f和e类型中较短的）
%h   散列码
%%   百分比类型
%n   换行符
%tx  日期与时间类型（x代表不同的日期与时间转换符
```

同时，System.out.printf()方法也完成了对String类的format()方法的调用，使得在直接输出String字符串时，对于格式化更加简便。  
注意：只有printf()方法可以完成对字符串输出的同时进行格式化，print()和println()方法并没有此功能。  

案例如下：  
```java
package zeh.myjavase.code10str.string;

public class Demo10StringFormat {
    public static void main(String[] args) {
        String str = null;
        str = String.format("Hi,%s", "王力");
        System.out.println(str);
        str = String.format("Hi,%s:%s.。。。。%s", "王南", "王力", "王张");
        System.out.println(str);

        //printf()方法是重载的对应format的方法
        //%c表示字符占位
        //%n表示换行占位，不用指定换行符，自动换行
        //%b表示布尔占位
        //%d表示数字占位
        //%x表示16进制占位
        //%o表示8进制占位
        //%f表示float占位
        System.out.printf("字母a的大写是：%c %n", 'A');
        System.out.printf("3>7的结果是：%b %n", 3 > 7);
        System.out.printf("100的一半是：%d %n", 100 / 2);
        System.out.printf("100的16进制数是：%x %n", 100);
        System.out.printf("100的8进制数是：%o %n", 100);
        System.out.printf("50元的书打8.5折扣是：%f 元%n", 50 * 0.85);
        System.out.printf("上面价格的16进制数是：%a %n", 50 * 0.85);
        System.out.printf("上面价格的指数表示：%e %n", 50 * 0.85);
        System.out.printf("上面价格的指数和浮点数结果的长度较短的是：%g %n", 50 * 0.85);
        System.out.printf("上面的折扣是%d%% %n", 85);
        System.out.printf("字母A的散列码是：%h %n", 'A');

        //print和println方法不能使用format
        System.out.print("12345678");
        System.out.println("12345678");
    }
}
```
运行结果：  
```
Hi,王力
Hi,王南:王力.。。。。王张
字母a的大写是：A 
3>7的结果是：false 
100的一半是：50 
100的16进制数是：64 
100的8进制数是：144 
50元的书打8.5折扣是：42.500000 元
上面价格的16进制数是：0x1.54p5 
上面价格的指数表示：4.250000e+01 
上面价格的指数和浮点数结果的长度较短的是：42.5000 
上面的折扣是85% 
字母A的散列码是：41 
1234567812345678

Process finished with exit code 0
```

# 4. NumberFormat抽象类，格式化数字
## 4.1 直接使用NumberFormat
NumberFormat表示数字格式化类，和DateFormat、MessageFormat类一样都是Format类的子类；  
其中DateFormat和NumberFormat都是抽象类。  
NumberFormat类还有个常用的子类：DecimalFormat类。  
NumberFormat对数字进行格式化，只能指定不同的语言环境，按照语言环境的默认格式对数字进行格式化。  
案例：  
```java
package zeh.myjavase.code18classlib.base;

import java.text.NumberFormat;
import java.util.Locale;

public class Demo13StudyNumberFormat {
	public static void main(String args[]) {
		// 使用当前语言环境格式化数字，当前的操作系统是中文语言环境，所以数字显示成了中国的数字格式化形式
		NumberFormat nf = null;
		// 返回当前环境默认语言的数字格式
		nf = NumberFormat.getInstance();
		System.out.println("格式化之后的数字：" + nf.format(10000000));
		System.out.println("格式化之后的数字：" + nf.format(1000.235));

		// 返回当前服务器上所有语言环境的数组
		Locale locale[] = NumberFormat.getAvailableLocales();
		for (int i = 0; i < locale.length; i++) {
			System.out.println(locale[i] + "、");
		}
		System.out.println("*************************************");

		// 实例化指定语言格式的NumberFormat
		NumberFormat nf2 = null;
		nf2 = NumberFormat.getInstance(new Locale("fr", "FR"));
		System.out.println("(法文语言环境下)格式化之后的数字：" + nf2.format(10000000));
		System.out.println("(法文语言环境下)格式化之后的数字：" + nf2.format(1000.235));
	}
}
```
运行结果：  
```
格式化之后的数字：10,000,000
格式化之后的数字：1,000.235
、
ar_AE、
ar_JO、
ar_SY、
hr_HR、
fr_BE、
es_PA、
es_VE、
mt_MT、
bg、
zh_TW、
it、
ko、
uk、
lv、
da_DK、
es_PR、
vi_VN、
en_US、
sr_ME、
sv_SE、
en_SG、
es_BO、
ar_BH、
pt、
ar_SA、
sk、
ar_YE、
hi_IN、
ga、
en_MT、
fi_FI、
et、
cs、
sr_BA_#Latn、
sv、
el、
uk_UA、
fr_CH、
hu、
in、
es_AR、
ar_EG、
ja_JP_JP_#u-ca-japanese、
es_SV、
pt_BR、
be、
cs_CZ、
es、
is_IS、
ca_ES、
pl_PL、
tr、
sr_CS、
hr、
ms_MY、
lt、
es_ES、
bg_BG、
es_CO、
fr、
sq、
ja、
sr_BA、
es_PY、
is、
de、
es_EC、
es_US、
ar_SD、
en、
ro_RO、
en_PH、
ca、
ar_TN、
sr_ME_#Latn、
es_GT、
el_CY、
ko_KR、
sl、
es_MX、
ru_RU、
es_HN、
no_NO_NY、
zh_HK、
hu_HU、
th_TH、
ar_IQ、
es_CL、
ar_MA、
fi、
ga_IE、
mk、
tr_TR、
ar_QA、
et_EE、
sr__#Latn、
pt_PT、
fr_LU、
ar_OM、
th、
sq_AL、
es_DO、
es_CU、
ar、
ru、
en_NZ、
sr_RS、
de_CH、
es_UY、
ms、
el_GR、
iw_IL、
en_ZA、
th_TH_TH_#u-nu-thai、
hi、
fr_FR、
de_AT、
nl、
no_NO、
en_AU、
vi、
fr_CA、
lv_LV、
nl_NL、
de_LU、
es_CR、
ar_KW、
ar_LY、
sr、
it_CH、
mt、
da、
de_DE、
ar_DZ、
sk_SK、
it_IT、
lt_LT、
en_IE、
zh_SG、
en_CA、
ro、
nl_BE、
no、
pl、
zh_CN、
ja_JP、
de_GR、
sr_RS_#Latn、
iw、
en_IN、
ar_LB、
es_NI、
zh、
be_BY、
mk_MK、
sl_SI、
es_PE、
in_ID、
en_GB、
*************************************
(法文语言环境下)格式化之后的数字：10 000 000
(法文语言环境下)格式化之后的数字：1 000,235
```

## 4.2 DecimalFormat类对数字按照指定格式模板进行格式化
DecimalFormat类是NumberFormat类的子类；  
与NumberFormat类相比，DecimalFormat类要比NumberFormat类更加方便，因为可以直接指定按用户自定义的方式进行格式化操作，与SimpleDateFormat类似。  
Decimal格式化模板：  
```
0：代表阿拉伯数字，每一个0表示一位阿拉伯数字，如果该位数字不存在则显示为0
#：代表阿拉伯数字，每一个#表示一位阿拉伯数字，如果该位不存在则不显示
.：小数点分隔符或者货币的小数分隔符
-：代表负号
,：分组分隔符
E：分隔科学计数法中的位数和指数
;：子模式边界，分隔整数和负数子模式
%：前缀或后缀，数字乘以100并显示为百分数
\u2030：前缀或后缀，数字乘以1000并显示为千分数
```
案例如下：  
单独封装一个DecimalFormat实用类用来格式化数字:  
```java
package zeh.myjavase.code18classlib.base;

import java.text.DecimalFormat;

// 专门用来实例化DecimalFormat类对象
class Demo06FormatDemo {
    public void format1(String pattern, double value) {
        DecimalFormat df = null;
        // 按照指定的模板实例化DecimalFormat
        df = new DecimalFormat(pattern);
        // 对指定的数字按照指定的模板进行格式化
        String str = df.format(value);
        System.out.println("使用" + pattern + "模板格式化数字，格式化前：" + value + "-->" + str);
    }
}
```
调用上面封装的数字格式化类，传入指定数字和指定格式化模板进行数字的格式化：  
```java
package zeh.myjavase.code18classlib.base;

public class Demo06StudyDecimalFormat {
    public static void main(String args[]) {
        Demo06FormatDemo demo = new Demo06FormatDemo();
        demo.format1("###,###.###", 111222.34567);
        demo.format1("0000,000.000", 111222.34567);
        demo.format1("###,###.###￥", 111222.34567);
        demo.format1("000,000.000￥", 111222.34567);
        demo.format1("##.###%", 0.345678);// 使用百分数形式
        demo.format1("00.###%", 0.023456);// 使用百分数形式
        demo.format1("###.###\u2030", 0.3455667);// 使用千分数形式
    }
}
```
运行结果：  
```
使用###,###.###模板格式化数字，格式化前：111222.34567-->111,222.346
使用0000,000.000模板格式化数字，格式化前：111222.34567-->0,111,222.346
使用###,###.###￥模板格式化数字，格式化前：111222.34567-->111,222.346￥
使用000,000.000￥模板格式化数字，格式化前：111222.34567-->111,222.346￥
使用##.###%模板格式化数字，格式化前：0.345678-->34.568%
使用00.###%模板格式化数字，格式化前：0.023456-->02.346%
使用###.###‰模板格式化数字，格式化前：0.3455667-->345.567‰
```

# 5. DateFormat抽象类
## 5.1 直接使用DateFormat
实际上java.util包中的Date对象取得是一个很准确的时间，但是因为不符合中国人的阅读习惯所以需要格式化一下。  
DateFormat类专门根据指定的Locale语言环境对日期进行特定的格式化操作。  
DateFormat类和MessageFormat类都是Format类的子类，专门用来格式化数据使用。  
DateFormat类是抽象类，通过静态方法getDateInstance等取得其实例对象。  
SimpleDateFormat类是DateFormat类的子类，比DateFormat更方便，可以自定义日期时间的格式化模板。  
注意：DateFormat没法自定义模板去格式化日期时间，一般还是SimpleDateFormat用的多。  
案例：  
```java
package zeh.myjavase.code18classlib.date;

import org.junit.Test;

import java.text.DateFormat;
import java.util.Date;
import java.util.Locale;

public class Demo06StudyDateAndDateFormat {

	@Test
	public void testDateFormat() {

		// 直接输出date对象，不格式化
		// 根据默认的日期时间显示格式进行显示
		System.out.println("直接输出date，按照 java 的默认格式显示日期：" + new Date());
		
		// 取得Date的DateFormat，只有日期。
		DateFormat df1 = DateFormat.getDateInstance();
		// 取得日期时间的DateFormat，有日期和时间，精确到秒。
		DateFormat df2 = DateFormat.getDateTimeInstance();
		
		// 格式化日期
		System.out.println("默认格式化后的日期：" + df1.format(new Date()));
		// 格式化日期时间
		System.out.println("默认格式化后的日期时间：" + df2.format(new Date()));

		// 指定Locale，显示中文环境下的日期时间显示格式
		// 缺点：DateFormat类虽然能够指定Locale显示中文，但是显示的格式仍然是默认的，而不是随心所欲自定义的格式
		// 取得日期，并指定显示风格
		DateFormat df3 = DateFormat.getDateInstance(DateFormat.YEAR_FIELD,
				new Locale("zh", "CN"));
		// 取得日期时间，并指定日期和时间的显示风格
		DateFormat df4 = DateFormat.getDateTimeInstance(DateFormat.YEAR_FIELD,
				DateFormat.ERA_FIELD, new Locale("zh", "CN"));
		// 根据中文Locale格式化日期
		System.out.println("根据中文环境格式化后的日期：" + df3.format(new Date()));
		// 根据中文locale格式化日期时间
		System.out.println("根据中文环境格式化后的日期时间：" + df4.format(new Date()));
	}
}
```
运行结果：  
```
直接输出date，按照 java 的默认格式显示日期：Wed Nov 24 15:26:59 CST 2021
默认格式化后的日期：2021-11-24
默认格式化后的日期时间：2021-11-24 15:26:59
根据中文环境格式化后的日期：2021年11月24日
根据中文环境格式化后的日期时间：2021年11月24日 下午03时26分59秒 CST
```

## 5.2 使用子类SimpleDateFormat进行日期格式化
SimpleDateFormat类提供的日期格式化更加灵活，方便java的date类型和String类型之间的互相转换。  
SimpleDateFormat类主要用于：将日期从一种格式转换成另外一种显示格式。只是日期格式的不同而已，日期本身的值不发生任何改变。  
首先会定义一个字符串格式的日期和一个日期模板，字符串格式的日期和模板必须相对应才能提取出来；将指定的日期从字符串日期中提取出来后再转换成另外一种显示格式。  

SimpleDateFormat类在使用中经常用于：  
将String类型的日期和Date格式的日期进行互转。
通过中间模板进行提取。  
1.  parse()方法用于从字符串日期中提取成Date类型的日期；  
2.  format()用于将Date类型的日期格式化为指定的字符串类型。  

支持的日期格式字符：  
![][日志字符格式]

案例如下：  
```java
package zeh.myjavase.code18classlib.date;

import org.junit.Test;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class Demo04StudyDateAndSimpleDateFormat {
    @Test
    public void testParse() throws ParseException {
        // 取得当前毫秒数，即自1970年后的毫秒数。
        System.out.println("当前毫秒数：" + System.currentTimeMillis());

        // 定义模板
        String pat = "yyyy-MM-dd";

        System.out.println("默认日期：" + new Date());
        // 根据格式化模板实例化SimpleDateFormat对象
        SimpleDateFormat format = new SimpleDateFormat(pat);

        String dateStr = "2018-09-03";
        // 根据日期字符串解析出指定格式的日期
        Date date = format.parse(dateStr);
        // 对指定格式的日期对象进行格式化
        String formatDate = format.format(date);
        System.out.println(date);
        System.out.println("对提取出来的日期对象格式化：" + formatDate);
    }

    @Test
    public void testFormat() {
        // 定义日期时间的字符串
        String strDate = "2018-10-19 10:11:30.234";
        // 准备第一个模板，与上面的字符串日期格式对应，可以将上面的日期数字按照该模板进行提取
        String pat1 = "yyyy-MM-dd hh:mm:ss.SSS";
        // 准备第二个模板，将按照第一个模板提取的日期数值按照第二个模板进行格式化
        String pat2 = "yyyy年MM月dd号  hh点mm分ss秒SSS毫秒";

        /* 实例化上述的两个模板对象 */
        SimpleDateFormat sdf1 = new SimpleDateFormat(pat1);
        SimpleDateFormat sdf2 = new SimpleDateFormat(pat2);

        Date date = null;
        try {
            // 从字符串日期中按照格式1提取出date对象
            date = sdf1.parse(strDate);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        String formatDate = sdf2.format(date);
        // 将提取出来的日期按照模板2进行格式化
        System.out.println("输出提取出来的日期对象格式化后的日期时间字符串：" + formatDate);
    }

    // 格式化当前日期时间
    @Test
    public void testFormatDate1() {
        Date date = new Date();// 实例化当前时间
        String pat1 = "yyyy-MM-dd hh:mm";
        SimpleDateFormat sdf = new SimpleDateFormat(pat1);// 根据 pat1 模板实例化
        // SimpleDateFormat 对象
        String formatDate = sdf.format(date);
        System.out.println("输出当前日期的格式化字符串：" + formatDate);
    }

    // 格式化当前日期时间，模板格式不同输出的当前日期不同
    @Test
    public void testFormatDate2() {
        // 实例化当前时间
        Date date = new Date();
        String pat1 = "yyyy-MMM-dd";
        // 根据pat1模板实例化SimpleDateFormat对象
        SimpleDateFormat sdf = new SimpleDateFormat(pat1);
        String formatDate = sdf.format(date);
        System.out.println("输出当前日期的格式化字符串：" + formatDate);
    }

    // 自定义模板格式化当前日期。
    @Test
    public void testFormatDateByMyStyle() {
        String myStyle = "yyyy年MM月dd号 hh点mm分ss秒SSS毫秒 ";
        SimpleDateFormat sdf = new SimpleDateFormat(myStyle);
        String dateAfterFormat = sdf.format(new Date());
        System.out.println("自己的模板格式化后的当前日期：" + dateAfterFormat);
    }

    @Test
    public void testFormatDateByMyStyle2() {
        // 自定义的模板，只包含模式字符，则可以自定义
        String myStyle = "yyyy---MMdd";
        SimpleDateFormat sdf = new SimpleDateFormat(myStyle);
        String dateAfterFormat = sdf.format(new Date());
        System.out.println("自己的模板格式化后的当前日期：" + dateAfterFormat);
    }

    // 将date对象只抽取出年月日，完了再转换成date对象。
    // 这种方式转换出的date只能是默认格式。
    // 注意：Java中的date直接输出只能是默认格式，如果需要自定义个数，必须将date格式化为指定的字符串才可。
    // 注意：只能通过SimpleDateFormat先将date对象format成固定格式的字符串，再通过SimpleDateFormat对象将格式化后的字符串解析成date对象。
    @Test
    public void testFormatDate() throws ParseException {
        // 自定义的模板，只包含模式字符，则可以自定义
        String myStyle = "yyyyMMdd";
        SimpleDateFormat sdf = new SimpleDateFormat(myStyle);
        String dateAfterFormat = sdf.format(new Date());
        System.out.println("自己的模板格式化后的当前日期字符串形式：" + dateAfterFormat);
        Date date = sdf.parse(dateAfterFormat);
        dateAfterFormat = sdf.format(date);
        System.out.println("自己的模板格式化后的当前日期：" + dateAfterFormat);

        String myStyle1 = "hh:mm:ss";
        sdf = new SimpleDateFormat(myStyle1);
        dateAfterFormat = sdf.format(new Date());
        System.out.println("自己的模板格式化后的当前日期字符串形式：" + dateAfterFormat);
    }
}
```
运行结果：  
```
当前毫秒数：1637740681414
默认日期：Wed Nov 24 15:58:01 CST 2021
Mon Sep 03 00:00:00 CST 2018
对提取出来的日期对象格式化：2018-09-03

Process finished with exit code 0

输出提取出来的日期对象格式化后的日期时间字符串：2018年10月19号  10点11分30秒234毫秒

Process finished with exit code 0

输出当前日期的格式化字符串：2021-11-24 03:59

Process finished with exit code 0

输出当前日期的格式化字符串：2021-十一月-24

Process finished with exit code 0

自己的模板格式化后的当前日期：2021年11月24号 04点00分10秒672毫秒 

Process finished with exit code 0

自己的模板格式化后的当前日期：2021---1124

Process finished with exit code 0

自己的模板格式化后的当前日期字符串形式：20211124
自己的模板格式化后的当前日期：20211124
自己的模板格式化后的当前日期字符串形式：04:00:40

Process finished with exit code 0
```

案例2：  
```java
package zeh.myjavase.code18classlib.date.util;

import java.text.SimpleDateFormat;
import java.util.Date;

// 日期时间类
public class DateTimeUtil2 {
    // 声明SimpleDateFormat对象
    private SimpleDateFormat sdf = null;

    // 得到完整的日期，格式为：yyyy-MM-dd HH:mm:ss.SSS
    public String getDate() {
        this.sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS");
        return this.sdf.format(new Date());// 按照模板格式化日期
    }

    // 得到完整的日期，格式为：yyyy年MM月dd日HH时mm分ss秒SSS毫秒
    public String getDateComplete() {
        this.sdf = new SimpleDateFormat("yyyy年MM月dd日HH时mm分ss秒SSS毫秒");
        return this.sdf.format(new Date());
    }

    // 得到时间戳，格式为：yyyyMMddHHmmssSSS
    public String getTimeStamp() {
        this.sdf = new SimpleDateFormat("yyyyMMddHHmmssSSS");
        return this.sdf.format(new Date());
    }
}

package zeh.myjavase.code18classlib.date;

import zeh.myjavase.code18classlib.date.util.DateTimeUtil2;

// 取得完整日期-基于SimpleDateFormat类。
// 专门设计一个类，专门用来取得完整日期格式。
public class Demo05StudyDateAndGetDate {
    public static void main(String args[]) {
        DateTimeUtil2 dt2 = new DateTimeUtil2();
        System.out.println("DateTime2取得系统日期：" + dt2.getDate());
        System.out.println("DateTime2取得中文环境下的日期：" + dt2.getDateComplete());
        System.out.println("DateTime2取得时间戳：" + dt2.getTimeStamp());
    }
}
```
运行如下：  
```
DateTime2取得系统日期：2021-11-24 16:20:56.904
DateTime2取得中文环境下的日期：2021年11月24日16时20分56秒905毫秒
DateTime2取得时间戳：20211124162056906
```
  
 [日志字符格式]: data:image/png;base64,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