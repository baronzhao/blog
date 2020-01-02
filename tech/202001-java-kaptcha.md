# Google Kaptcha 验证码组件


Kaptcha 是谷歌开源的非常实用的验证码生成工具，基于SimpleCaptcha的开源项目。使用 Kaptcha 生成验证码十分简单并且参数可以进行自定义。只需添加jar包配置下就可以使用，通过配置，可以自己定义验证码大小、颜色、显示的字符等等。

其原理是使用java.awt.Graphics2D实现绘图功能:

- new一个BufferedImage。new BufferedImage(width, height, BufferedImage.TYPE_INT_ARGB);
- 基于此BufferedImage创建一个绘图对象，即使用 createGraphics 方法，得 Graphics2D 实例。
- 使用Graphics2D 实例进行画图，所有绘图坐标基于创建此Graphics2D 的BufferedImage。
- 调用Graphics2D 对象的 dispose() 方法，进行绘图处理，使绘图效果应用到BufferedImage 对象。


下面就来讲一下如何使用kaptcha生成验证码以及在服务器端取出验证码进行校验。

Spring项目使用Kaptcha肯定要依赖kaptcha的jar，maven项目的话直接加入如下依赖

```xml
<dependency>
    <groupId>com.github.penggle</groupId>
    <artifactId>kaptcha</artifactId>
    <version>2.3.2</version>
</dependency>
```

进一步了解bean配置，可以参考官方wiki：https://code.google.com/archive/p/kaptcha/wikis

参考文件备份：[HowToUse](/copy/kaptcha/HowToUse.pdf)、[SpringUsage](/copy/kaptcha/SpringUsage.pdf)、[ConfigParameters](/copy/kaptcha/ConfigParameters.pdf)、[FrequentlyAskedQuestions](/copy/kaptcha/FrequentlyAskedQuestions.pdf)


进一步了解SpringBoot集成Kaptcha，可以参考开源项目README:

https://gitee.com/baomidou/kaptcha-spring-boot-starter

#简单快速集成 Google Kaptcha验证码

## 如何使用

1. 引入 kaptcha-datasource-spring-boot-starter。

```xml
<dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>kaptcha-spring-boot-starter</artifactId>
  <version>${version}</version>
</dependency>
```

2. 在Controller使用`Kaptcha`。

```java
@RestController
@RequestMapping("/kaptcha")
public class KaptchaController {

  @Autowired
  private Kaptcha kaptcha;

  @GetMapping("/render")
  public void render() {
    kaptcha.render();
  }

  @PostMapping("/valid")
  public void validDefaultTime(@RequestParam String code) {
    //default timeout 900 seconds
    kaptcha.validate(code);
  }

  @PostMapping("/validTime")
  public void validCustomTime(@RequestParam String code) {
    kaptcha.validate(code, 60);
  }

}
```

3. 发生错误会抛出异常，建议使用全局异常来处理。

```java
KaptchaException  //super Exception

KaptchaIncorrectException

KaptchaNotFoundException

KaptchaTimeoutException

KaptchaRenderException //If something is wrong then Image.write when render.
```

```java
import com.baomidou.kaptcha.exception.KaptchaException;
import com.baomidou.kaptcha.exception.KaptchaIncorrectException;
import com.baomidou.kaptcha.exception.KaptchaNotFoundException;
import com.baomidou.kaptcha.exception.KaptchaTimeoutException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionHandler {

  @ExceptionHandler(value = KaptchaException.class)
  public String kaptchaExceptionHandler(KaptchaException kaptchaException) {
    if (kaptchaException instanceof KaptchaIncorrectException) {
      return "验证码不正确";
    } else if (kaptchaException instanceof KaptchaNotFoundException) {
      return "验证码未找到";
    } else if (kaptchaException instanceof KaptchaTimeoutException) {
      return "验证码过期";
    } else {
      return "验证码渲染失败";
    }

  }

}
```

4. 自定义验证码参数,以下为默认配置。

```yaml
kaptcha:
  height: 50
  width: 200
  content:
    length: 4
    source: abcdefghjklmnopqrstuvwxyz23456789
    space: 2
  font:
    color: black
    name: Arial
    size: 40
  background-color:
    from: lightGray
    to: white
  border:
    enabled: true
    color: black
    thickness: 1
```