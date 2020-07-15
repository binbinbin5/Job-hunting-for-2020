
[toc]
## 全局异常
Spring MVC全局异常处理：保证了出现异常的适合，异常不会直接显示到前端浏览器上（异常包括sql语句，Java包，数据库IP，第几行之类的问题，一定情况下保证安全）
![](https://raw.githubusercontent.com/binbinbin5/myPics/master/file/20190814191744.png)

1. 包扫描变更，因为是Spring MVC,所以当异常传给dispathcerserlvet的时候，我们对异常进行包装，也就是modeAndView,我们通过mappingJacksonview处理modeAndView,==spring扫描除了controller之外的bean，MVC扫描controller的bean==

```
 <context:component-scan base-package="com.mmall" annotation-config="true">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller" />
    </context:component-scan>
//扫描所有的BEAN 排除Contoller，交给SpringMVC
```

```
<!-- 二期温馨提示,springmvc扫描包指定到controller，防止重复扫描 -->
<context:component-scan base-package="com.mmall.controller" annotation-config="true" use-default-filters="false">
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
//扫描包隔离
```


2. Common下创建一个异常处理类注解@component保证被扫描到,类继承handleExceptionResolver，重写方法返回ModeAndVie的Json给前端。

```
@Slf4j
@Component //作为Bean注入容器
public class ExceptionResolver implements HandlerExceptionResolver{

    @Override
    public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) {
        //打印到后台日志
        log.error("{} Exception",httpServletRequest.getRequestURI(),e);
        
        ModelAndView modelAndView = new ModelAndView(new MappingJacksonJsonView());


        //当使用是jackson2.x的时候使用MappingJackson2JsonView，课程中使用的是1.9。
        
        //封装异常不返回前端
        modelAndView.addObject("status",ResponseCode.ERROR.getCode());
        modelAndView.addObject("msg","接口异常,详情请查看服务端日志的异常信息");
        modelAndView.addObject("data",e.toString());//简单异常
        return modelAndView;
    }

}

```


## 拦统一拦截login

![](https://raw.githubusercontent.com/binbinbin5/myPics/master/file/%E7%BB%9F%E4%B8%80%E6%9D%83%E9%99%90%E6%8B%A6%E6%88%AA.png)


>作用：contrller中各种执行各种业务前需要从redis中拿到token，来判断是否登录，然后反序列化json对象拿到权限进行校验。代码冗余了。

```
//        String loginToken = CookieUtil.readLoginToken(httpServletRequest);
//        if(StringUtils.isEmpty(loginToken)){
//            return ServerResponse.createByErrorMessage("用户未登录,无法获取当前用户的信息");
//        }
//        String userJsonStr = RedisShardedPoolUtil.get(loginToken);
//        User user = JsonUtil.string2Obj(userJsonStr,User.class);
//
//        if(user == null){
//            return ServerResponse.createByErrorCodeMessage(ResponseCode.NEED_LOGIN.getCode(),"用户未登录,请登录");
//        }
//        //校验一下是否是管理员
//        if(iUserService.checkAdminRole(user).isSuccess()){
//            //是管理员
//            //增加我们处理分类的逻辑
//            return iCategoryService.addCategory(categoryName,parentId);
//
//        }else{
//            return ServerResponse.createByErrorMessage("无权限操作,需要管理员权限");
//        }
```
使用拦截器，写一个类集成HandleInterceptor重写preHandle,postHandle,afterHander。特别是preHandle，在执行controller前自动进行校验。


### 出现什么问题问题
拦截器也拦截了登录，出现login.do死循环，所以spring配置跳过login.do的
```
<!--<mvc:exclude-mapping path="/manage/user/login.do"/>-->
```
### 流程

> 概念
- /** 包含子路径全部
- /* 不包含子路径
- / 根目录请求


1. SpringMVC配置文件 初始化类，拦截manage下面所有需要进行权限校验的类，并且为了登录死循环排除login.do（或者代码里进行判断）
```
<mvc:interceptors>
        <!-- 定义在这里的，所有的都会拦截-->
        <mvc:interceptor>
            <!--manage/a.do  /manage/*-->
            <!--manage/b.do  /manage/*-->
            <!--manage/product/save.do /manage/**-->
            <!--manage/order/detail.do /manage/**-->
            <mvc:mapping path="/manage/**"/>//拦截
            <mvc:exclude-mapping path="/manage/user/login.do"/>//排除login.do
            <bean class="com.mmall.controller.common.interceptor.AuthorityInterceptor" />
        </mvc:interceptor>

    </mvc:interceptors>
```
2. 拦截器类，集成HandlerInterceptor重写preHandle,postHandle,afterhandle
    - 拿到contorller方法名
    - 反射解析拿到名字
    - request拿到Map（key和value）
    - ====上面拿到用于日志，下面是真正的拦截器需求====
    - 重置response
    - 从redis中拿到cookie
    - 从cookie拿到loginintoken
    - 反序列化拿到权限判断

```
@Slf4j
public class AuthorityInterceptor implements HandlerInterceptor{

//controller前
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        log.info("preHandle");
        //请求中Controller中的方法名
        HandlerMethod handlerMethod = (HandlerMethod)handler;

        //解析HandlerMethod

        String methodName = handlerMethod.getMethod().getName();
        String className = handlerMethod.getBean().getClass().getSimpleName();

        //解析参数,具体的参数key以及value是什么，我们打印日志
        StringBuffer requestParamBuffer = new StringBuffer();
        Map paramMap = request.getParameterMap();
        Iterator it = paramMap.entrySet().iterator();
        while (it.hasNext()){
            Map.Entry entry = (Map.Entry)it.next();
            String mapKey = (String)entry.getKey();

            String mapValue = StringUtils.EMPTY;

            //request这个参数的map，里面的value返回的是一个String[]
            Object obj = entry.getValue();
            if(obj instanceof String[]){
                String[] strs = (String[])obj;
                mapValue = Arrays.toString(strs);
            }
            requestParamBuffer.append(mapKey).append("=").append(mapValue);
        }

        if(StringUtils.equals(className,"UserManageController") && StringUtils.equals(methodName,"login")){
            log.info("权限拦截器拦截到请求,className:{},methodName:{}",className,methodName);
            //如果是拦截到登录请求，不打印参数，因为参数里面有密码，全部会打印到日志中，防止日志泄露
            return true;
        }

        log.info("权限拦截器拦截到请求,className:{},methodName:{},param:{}",className,methodName,requestParamBuffer.toString());


        User user = null;
        
        String loginToken = CookieUtil.readLoginToken(request);
        if(StringUtils.isNotEmpty(loginToken)){
            String userJsonStr = RedisShardedPoolUtil.get(loginToken);
            user = JsonUtil.string2Obj(userJsonStr,User.class);
        }
        
        //检查用户信息

        if(user == null || (user.getRole().intValue() != Const.Role.ROLE_ADMIN)){
            //返回false.即不会调用controller里的方法
            
            //设置reponse
            response.reset();//geelynote 这里要添加reset，否则报异常 getWriter() has already been called for this response.
            response.setCharacterEncoding("UTF-8");//geelynote 这里要设置编码，否则会乱码
            response.setContentType("application/json;charset=UTF-8");//geelynote 这里要设置返回值的类型，因为全部是json接口。

            PrintWriter out = response.getWriter();//拿到输出对象

            //上传由于富文本的控件要求，要特殊处理返回值，这里面区分是否登录以及是否有权限
            if(user == null){
                if(StringUtils.equals(className,"ProductManageController") && StringUtils.equals(methodName,"richtextImgUpload")){
                    Map resultMap = Maps.newHashMap();
                    resultMap.put("success",false);
                    resultMap.put("msg","请登录管理员");
                    out.print(JsonUtil.obj2String(resultMap));
                }else{
                    out.print(JsonUtil.obj2String(ServerResponse.createByErrorMessage("拦截器拦截,用户未登录")));
                }
            }else{
                if(StringUtils.equals(className,"ProductManageController") && StringUtils.equals(methodName,"richtextImgUpload")){
                    Map resultMap = Maps.newHashMap();
                    resultMap.put("success",false);
                    resultMap.put("msg","无权限操作");
                    out.print(JsonUtil.obj2String(resultMap));
                }else{
                    out.print(JsonUtil.obj2String(ServerResponse.createByErrorMessage("拦截器拦截,用户无权限操作")));
                }
            }
            out.flush();
            out.close();//geelynote 这里要关闭

            return false;

        }
        return true;
    }
//controller后
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        log.info("postHandle");
    }
//所有执行后
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        log.info("afterCompletion");
    }
}

```
> 执行顺序
    
    1. preHandle
    2. 对应拦截的Controller
    3. postHandle
    4. afterHandle

3. 解决拦截登录死循环（二选一）

```
<mvc:exclude-mapping path="/manage/user/login.do"/>
```
or

```
 if(StringUtils.equals(className,"UserManageController") && StringUtils.equals(methodName,"login")){
            log.info("权限拦截器拦截到请求,className:{},methodName:{}",className,methodName);
            //如果是拦截到登录请求，不打印参数，因为参数里面有密码，全部会打印到日志中，防止日志泄露
            return true;
        }

```
## 定时关单

完成作业调度，比如定时任务


#### Spring Schedule Cron表达式

格式： 秒 分 时 日 月 周 年（可选）

- Seconds
- Minutes
- Hours
- Day-of-Month
![](https://raw.githubusercontent.com/binbinbin5/myPics/master/file/20190820211448.png)

![](https://raw.githubusercontent.com/binbinbin5/myPics/master/file/20190820211532.png)

![](https://raw.githubusercontent.com/binbinbin5/myPics/master/file/20190820211752.png)

![](https://raw.githubusercontent.com/binbinbin5/myPics/master/file/20190820211852.png)

#### Spring Schedule Cron生成器

[生成器](http://cron.qqe2.com/)

#### 配置 

- @Component


- @Scheduled(cron="0 */1 * * * ?")

行锁和表锁：
- SELECT...FOR UPDATE
- 使用InnoDB
- 明确的主键 行锁
- 无明确的主键 表锁

#### 集群模式中下订单和关闭订单
![](https://raw.githubusercontent.com/binbinbin5/myPics/master/file/redis%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81.png)
#### Coding

1. spring配置applicationConetxt.xml

```
 <!-- 二期新增spring schedule的时候新增的 -->
    <context:property-placeholder location="classpath:datasource.properties"/>
    <task:annotation-driven/>
```

```
//dao层接口对应mapper
    //二期新增定时关单
    List<Order> selectOrderStatusByCreateTime(@Param("status") Integer status,@Param("date") String date);

    int closeOrderByOrderId(Integer id);
```

```
    //加一个类 void closeOrder(int hour);
    public interface IOrderService {
        //hour个小时以内未付款的订单，进行关闭
        void closeOrder(int hour);
    
    
    }

    //public class OrderServiceImpl implements IOrderService 中增加一个：
    @Override
    public void closeOrder(int hour) {
    //当前时间
        Date closeDateTime = DateUtils.addHours(new Date(),-hour);
        //查找订单
        List<Order> orderList = orderMapper.selectOrderStatusByCreateTime(Const.OrderStatusEnum.NO_PAY.getCode(),DateTimeUtil.dateToStr(closeDateTime));

        for(Order order : orderList){
            List<OrderItem> orderItemList = orderItemMapper.getByOrderNo(order.getOrderNo());
            //拿到即将关闭的订单
            for(OrderItem orderItem : orderItemList){

                //一定要用主键where条件，防止锁表。同时必须是支持MySQL的InnoDB。
                Integer stock = productMapper.selectStockByProductId(orderItem.getProductId());

                //考虑到已生成的订单里的商品，被删除的情况
                if(stock == null){
                    continue;
                }
                
                //更新库存
                Product product = new Product();//新加一个，提高SQL执行效率
                product.setId(orderItem.getProductId());
                product.setStock(stock+orderItem.getQuantity());
                productMapper.updateByPrimaryKeySelective(product);
            }
            orderMapper.closeOrderByOrderId(order.getId());
            log.info("关闭订单OrderNo：{}",order.getOrderNo());
        }
    }
```



```
//超过30分钟，关闭订单，并且商品的容量增加1
@Component
@Slf4j
public class CloseOrderTask {

    @Autowired
    private IOrderService iOrderService;

    @Autowired
    private RedissonManager redissonManager;

    @PreDestroy
    public void delLock(){
    
        RedisShardedPoolUtil.del(Const.REDIS_LOCK.CLOSE_ORDER_TASK_LOCK);

    }

//    @Scheduled(cron="0 */1 * * * ?")//每1分钟(每个1分钟的整数倍)
//创建分布式锁
    public void closeOrderTaskV1(){
        log.info("关闭订单定时任务启动");
        //获取2小时前的订单关闭
        int hour = Integer.parseInt(PropertiesUtil.getProperty("close.order.task.time.hour","2"));
//        iOrderService.closeOrder(hour);
        log.info("关闭订单定时任务结束");
    }


}

```
### 集群下Spring Schedule+分布式锁

命令与时间戳

- setnx:不存在才添加成功，存在就失效（具有原子性）
- getset:获取和设置，先get再set（原子性）
- expire:设置有效期
- del:删除

#### 防止Tomcat shutdown

```
   @PreDestroy
    public void delLock(){
        RedisShardedPoolUtil.del(Const.REDIS_LOCK.CLOSE_ORDER_TASK_LOCK);

    }
```

#### 分布式锁流程图与优化
这个有缺陷：
![](https://raw.githubusercontent.com/binbinbin5/myPics/master/file/20190821152907.png)

```
public void closeOrderTaskV2(){
        log.info("关闭订单定时任务启动");
        long lockTimeout = Long.parseLong(PropertiesUtil.getProperty("lock.timeout","5000"));

        Long setnxResult = RedisShardedPoolUtil.setnx(Const.REDIS_LOCK.CLOSE_ORDER_TASK_LOCK,String.valueOf(System.currentTimeMillis()+lockTimeout));
        if(setnxResult != null && setnxResult.intValue() == 1){
            //如果返回值是1，代表设置成功，获取锁
            closeOrder(Const.REDIS_LOCK.CLOSE_ORDER_TASK_LOCK);
        }else{
            log.info("没有获得分布式锁:{}",Const.REDIS_LOCK.CLOSE_ORDER_TASK_LOCK);
        }
        log.info("关闭订单定时任务结束");
    }
    
    //设置锁的有效期
  private void closeOrder(String lockName){
        RedisShardedPoolUtil.expire(lockName,5);//有效期50秒，防止死锁
        log.info("获取{},ThreadName:{}",Const.REDIS_LOCK.CLOSE_ORDER_TASK_LOCK,Thread.currentThread().getName());
        int hour = Integer.parseInt(PropertiesUtil.getProperty("close.order.task.time.hour","2"));
        iOrderService.closeOrder(hour);
        RedisShardedPoolUtil.del(Const.REDIS_LOCK.CLOSE_ORDER_TASK_LOCK);
        log.info("释放{},ThreadName:{}",Const.REDIS_LOCK.CLOSE_ORDER_TASK_LOCK,Thread.currentThread().getName());
        log.info("===============================");
    }

 @PreDestroy
 //没有使用kill进程的方式关闭tomcat，那么使用shutdown命令关闭
 //缺点是如果进程多那么关闭锁时间比较长，
    public void delLock(){
        RedisShardedPoolUtil.del(Const.REDIS_LOCK.CLOSE_ORDER_TASK_LOCK);

    }
```

![](https://raw.githubusercontent.com/binbinbin5/myPics/master/file/20190821153133.png)

双重防死锁：
```
@Scheduled(cron="0 */1 * * * ?")
    public void closeOrderTaskV3(){
        log.info("关闭订单定时任务启动");
        long lockTimeout = Long.parseLong(PropertiesUtil.getProperty("lock.timeout","5000"));
        Long setnxResult = RedisShardedPoolUtil.setnx(Const.REDIS_LOCK.CLOSE_ORDER_TASK_LOCK,String.valueOf(System.currentTimeMillis()+lockTimeout));
        if(setnxResult != null && setnxResult.intValue() == 1){
            closeOrder(Const.REDIS_LOCK.CLOSE_ORDER_TASK_LOCK);
        }else{
            //未获取到锁，继续判断，判断时间戳，看是否可以重置并获取到锁
            String lockValueStr = RedisShardedPoolUtil.get(Const.REDIS_LOCK.CLOSE_ORDER_TASK_LOCK);
            if(lockValueStr != null && System.currentTimeMillis() > Long.parseLong(lockValueStr)){
                String getSetResult = RedisShardedPoolUtil.getSet(Const.REDIS_LOCK.CLOSE_ORDER_TASK_LOCK,String.valueOf(System.currentTimeMillis()+lockTimeout));
                //再次用当前时间戳getset。
                //返回给定的key的旧值，->旧值判断，是否可以获取锁
                //当key没有旧值时，即key不存在时，返回nil ->获取锁
                //这里我们set了一个新的value值，获取旧的值。
                if(getSetResult == null || (getSetResult != null && StringUtils.equals(lockValueStr,getSetResult))){
                    //真正获取到锁
                    closeOrder(Const.REDIS_LOCK.CLOSE_ORDER_TASK_LOCK);
                }else{
                    log.info("没有获取到分布式锁:{}",Const.REDIS_LOCK.CLOSE_ORDER_TASK_LOCK);
                }
            }else{
                log.info("没有获取到分布式锁:{}",Const.REDIS_LOCK.CLOSE_ORDER_TASK_LOCK);
            }
        }
        log.info("关闭订单定时任务结束");
    }
```



## RESTful配置

requestMapping既有RESTful又有.do：*.do改成"/"

RESTful占位，通过URL定位资源
```
//web.xml配置

    <!--<servlet-mapping>-->
        <!--<servlet-name>dispatcher</servlet-name>-->
        <!--<url-pattern>*.do</url-pattern>-->
    <!--</servlet-mapping>-->


    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```

### requestMapping


```JAVA

    @RequestMapping("detail.do")
    @ResponseBody
    public ServerResponse<ProductDetailVo> detail(Integer productId){
        return iProductService.getProductDetail(productId);
    }
    
//xx.com/product/detail.do?productId=26

    @RequestMapping(value = "/{productId}", method = RequestMethod.GET)
    @ResponseBody
    public ServerResponse<ProductDetailVo> detailRESTful(@PathVariable Integer productId){
        return iProductService.getProductDetail(productId);
    }
//xx.com/product/26
```
xx.com/product/detail.do?productId=26变成xx.com/product/26

并不是所有都可以改成RESTful，比如收货人地址..

### Deatail,List改造成RESTful


```
@RequestMapping("list.do")
    @ResponseBody
    public ServerResponse<PageInfo> list(@RequestParam(value = "keyword",required = false)String keyword,
                                         @RequestParam(value = "categoryId",required = false)Integer categoryId,
                                         @RequestParam(value = "pageNum",defaultValue = "1") int pageNum,
                                         @RequestParam(value = "pageSize",defaultValue = "10") int pageSize,
                                         @RequestParam(value = "orderBy",defaultValue = "") String orderBy){
        return iProductService.getProductByKeywordCategory(keyword,categoryId,pageNum,pageSize,orderBy);
    }


    //http://www.happymmall.com/product/手机/100012/1/10/price_asc
    @RequestMapping(value = "/{keyword}/{categoryId}/{pageNum}/{pageSize}/{orderBy}",method = RequestMethod.GET)
    @ResponseBody
    public ServerResponse<PageInfo> listRESTful(@PathVariable(value = "keyword")String keyword,
                                         @PathVariable(value = "categoryId")Integer categoryId,
                                         @PathVariable(value = "pageNum") Integer pageNum,
                                         @PathVariable(value = "pageSize") Integer pageSize,
                                         @PathVariable(value = "orderBy") String orderBy){
        if(pageNum == null){
            pageNum = 1;
        }
        if(pageSize == null){
            pageSize = 10;
        }
        if(StringUtils.isBlank(orderBy)){
            orderBy = "price_asc";
        }

        return iProductService.getProductByKeywordCategory(keyword,categoryId,pageNum,pageSize,orderBy);
    }
    
    
    
```
product/手机/100012/1/10/price_asc这些资源缺少一个就会定位不准确，这几个任何参数不可以为空，这个就不适合RESTful需要改修。



```
//    http://www.happymmall.com/product/100012/1/10/price_asc
    @RequestMapping(value = "/{categoryId}/{pageNum}/{pageSize}/{orderBy}",method = RequestMethod.GET)
    @ResponseBody
    public ServerResponse<PageInfo> listRESTfulBadcase(@PathVariable(value = "categoryId")Integer categoryId,
                                                @PathVariable(value = "pageNum") Integer pageNum,
                                                @PathVariable(value = "pageSize") Integer pageSize,
                                                @PathVariable(value = "orderBy") String orderBy){
        if(pageNum == null){
            pageNum = 1;
        }
        if(pageSize == null){
            pageSize = 10;
        }
        if(StringUtils.isBlank(orderBy)){
            orderBy = "price_asc";
        }

        return iProductService.getProductByKeywordCategory("",categoryId,pageNum,pageSize,orderBy);
    }



 @RequestMapping(value = "/{keyword}/{pageNum}/{pageSize}/{orderBy}",method = RequestMethod.GET)
    @ResponseBody
    public ServerResponse<PageInfo> listRESTfulBadcase(@PathVariable(value = "keyword")String keyword,
                                                @PathVariable(value = "pageNum") Integer pageNum,
                                                @PathVariable(value = "pageSize") Integer pageSize,
                                                @PathVariable(value = "orderBy") String orderBy){
        if(pageNum == null){
            pageNum = 1;
        }
        if(pageSize == null){
            pageSize = 10;
        }
        if(StringUtils.isBlank(orderBy)){
            orderBy = "price_asc";
        }

        return iProductService.getProductByKeywordCategory(keyword,null,pageNum,pageSize,orderBy);
    }
```

修改BADcase，value = "/keyword/{keyword}/{pageNum}/{pageSize}/{orderBy}",method = RequestMethod.GET

```
 //http://www.happymmall.com/product/keyword/手机/1/10/price_asc
    @RequestMapping(value = "/keyword/{keyword}/{pageNum}/{pageSize}/{orderBy}",method = RequestMethod.GET)
    @ResponseBody
    public ServerResponse<PageInfo> listRESTful(@PathVariable(value = "keyword")String keyword,
                                                       @PathVariable(value = "pageNum") Integer pageNum,
                                                       @PathVariable(value = "pageSize") Integer pageSize,
                                                       @PathVariable(value = "orderBy") String orderBy){
        if(pageNum == null){
            pageNum = 1;
        }
        if(pageSize == null){
            pageSize = 10;
        }
        if(StringUtils.isBlank(orderBy)){
            orderBy = "price_asc";
        }

        return iProductService.getProductByKeywordCategory(keyword,null,pageNum,pageSize,orderBy);
    }


    //http://www.happymmall.com/product/category/100012/1/10/price_asc
    @RequestMapping(value = "/category/{categoryId}/{pageNum}/{pageSize}/{orderBy}",method = RequestMethod.GET)
    @ResponseBody
    public ServerResponse<PageInfo> listRESTful(@PathVariable(value = "categoryId")Integer categoryId,
                                                       @PathVariable(value = "pageNum") Integer pageNum,
                                                       @PathVariable(value = "pageSize") Integer pageSize,
                                                       @PathVariable(value = "orderBy") String orderBy){
        if(pageNum == null){
            pageNum = 1;
        }
        if(pageSize == null){
            pageSize = 10;
        }
        if(StringUtils.isBlank(orderBy)){
            orderBy = "price_asc";
        }

        return iProductService.getProductByKeywordCategory("",categoryId,pageNum,pageSize,orderBy);
    }
```
