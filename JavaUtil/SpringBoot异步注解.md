**AsyncConfigurer.java**

```java

import java.util.concurrent.Executor;
import org.springframework.aop.interceptor.AsyncUncaughtExceptionHandler;
import org.springframework.beans.factory.annotation.Configurable;
import org.springframework.scheduling.annotation.AsyncConfigurer;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

@Configurable
@EnableAsync
public class CustomMultiThreadingConfig implements AsyncConfigurer {
 
    @Override
    public Executor getAsyncExecutor(){
        
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        taskExecutor.setCorePoolSize(5);
        taskExecutor.setMaxPoolSize(10);
        taskExecutor.setQueueCapacity(25);
        taskExecutor.initialize();
        return taskExecutor;
    }
    
    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return AsyncConfigurer.super.getAsyncUncaughtExceptionHandler();
    }
}
```

**使用**
```java

import javax.annotation.Resource;
 
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
 
import com.xcwlkj.fivedollars.service.ArticleService;
import com.xcwlkj.util.wrapper.WrapMapper;
import com.xcwlkj.util.wrapper.Wrapper;
/**
控制层
**/ 
 
@RestController
@RequestMapping(value = "", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
public class ArticleController extends BaseController {
 
    @Resource
    private ArticleService articleService;
 
	
    @PostMapping(value = "/manager/article/queryArticleList")
    public Wrapper<QueryArticleListRespModel> queryArticleList(@RequestBody QueryArticleListReqModel reqModel) {
 
      
        //多线程异步调用测试
        for (int i = 0; i < 10; i++) {
            articleService.executeAsyncTask1(i);
            articleService.executeAsyncTask2(i);
        }
            	
    }
 
}

```
```java

import org.springframework.stereotype.Service;
 
import com.xcwlkj.fivedollars.model.dto.article.AddArticleDTO;
import com.xcwlkj.fivedollars.model.dto.article.DeleteArticleDTO;
import com.xcwlkj.fivedollars.model.dto.article.QueryArticleDetailDTO;
import com.xcwlkj.fivedollars.model.dto.article.QueryArticleListDTO;
import com.xcwlkj.fivedollars.model.dto.article.UpdateArticleDTO;
import com.xcwlkj.fivedollars.model.vo.article.QueryArticleDetailVO;
import com.xcwlkj.fivedollars.model.vo.article.QueryArticleListVO;
 /**
接口
**/
@Service
public interface ArticleService  {
 
 
    void executeAsyncTask1(int i);
 
    void executeAsyncTask2(int i);
}
```
```java
import java.util.Date;
import javax.annotation.Resource;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;
 
@Service("articleService")
public class ArticleServiceImpl  implements ArticleService  {
	
	@Async
    @Override
    public void executeAsyncTask1(int i) {
        
	    System.out.println("executeAsyncTask1======>{}"+i);
    }
	
	@Async
    @Override
    public void executeAsyncTask2(int i) {
	    
	    System.out.println("executeAsyncTask2======>{}"+i);
    }
}

```
