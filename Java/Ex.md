# 一些示例代码

## 一. 时间

1. 获取当前时间 (LocalDateTime)

```java
    String currentYear = 
    LocalDateTime.now(ZoneOffset.of("+8"))
                 .format(DateTimeFormatter.ofPattern("yyyy"));
```

## 二. 数据格式

1. 保留两位小数 (BigDecimal)
```java
    BigDecimal b1 = new BigDecimal(0);
    b1.setScale(2,BigDecimal.ROUND_HALF_UP);
```

## 三. 异步多线程

1. 简易线程池处理

```java
ExecutorService executor = Executors.newFixedThreadPool(4);
        //累计发放积分
        CompletableFuture<String> future1 = CompletableFuture.supplyAsync(()->{
            IntegralResult totalNum = this.integralMapper.getIgOverviewTotalNum(param.getGId(),null);
            if(ToolUtil.isNotEmpty(totalNum)){
                return totalNum.getTotalNum();
            } else {
                return "0";
            }
        },executor);

        //发放积分次数
        CompletableFuture<String> future2 = CompletableFuture.supplyAsync(()->{
            IntegralResult frequencyNum = this.integralMapper.getIgOverviewFrequencyNum(param.getGId());
            if(ToolUtil.isNotEmpty(frequencyNum)){
                return frequencyNum.getFrequencyNum();
            } else {
                return "0";
            }
        },executor);

        //兑换商品次数
        CompletableFuture<String> future3 = CompletableFuture.supplyAsync(()->{
            IntegralResult exchangeGoodsNum = this.integralMapper.getIgOverviewExchangeGoodsNum(param.getGId());
            if(ToolUtil.isNotEmpty(exchangeGoodsNum)){
                return exchangeGoodsNum.getExchangeGoodsNum();
            } else {
                return "0";
            }
        },executor);

        //参与居民数
        CompletableFuture<String> future4 = CompletableFuture.supplyAsync(()->{
            IntegralResult residentNum = this.integralMapper.getIgOverviewResidentNum(param.getGId());
            if(ToolUtil.isNotEmpty(residentNum)){
                return residentNum.getResidentNum();
            } else {
                return "0";
            }
        },executor);

        CompletableFuture.allOf(future1, future2, future3, future4);
        IntegralResult ret = new IntegralResult(future1.join(),future2.join(),future3.join(),future4.join());

        executor.shutdown();
```

2. 使用List承接CompleteFuture
```java
ExecutorService executor = Executors.newFixedThreadPool(6);
        List<CompletableFuture<IgUsedResult>> list = new ArrayList<>();
        //近6个月的数据
        for(int i = 0; i< Consts.HalfYearMonthNum; i++){
            int j = i;
            CompletableFuture<IgUsedResult> future = CompletableFuture.supplyAsync(()->{
                String month = this.getBeforeMonth(Consts.HalfYearMonthNum- j);
                String time = this.getBeforeMonthCn(Consts.HalfYearMonthNum- j);
                IgUsedResult ret = new IgUsedResult();
                ret.setTime(time);

                //获得积分
                IntegralResult wined = this.integralMapper.getIgOverviewTotalNum(param.getGId(),month);
                if(ToolUtil.isNotEmpty(wined)){
                    ret.setWin(wined.getTotalNum());
                } else {
                    ret.setWin("0");
                }

                //使用积分
                IgUsedResult used = this.integralMapper.getIgUsed(param.getGId(),month);
                if(ToolUtil.isNotEmpty(used)){
                    ret.setUsed(used.getUsed());
                } else {
                    ret.setUsed("0");
                }
                return ret;
            },executor);
            list.add(future);
        }
        List<IgUsedResult> rets = list.stream().map(CompletableFuture::join).collect(Collectors.toList());
        executor.shutdown();
```

## 四. 鉴权框架 (JWT)

### 1. 实现JWT相关方法

```java
/**
 * JWTUtil
 *
 * @Author: Jane_YH
 * @Version: 1.0
 * @Since: 2022/7/27 15:54
 */
public class JWTUtil {

    // 一个不能暴露的密钥
    private static String SIGNATURE = "token!@#$%^7890";

    /**
     * 生成token
     * @param map
     * @return
     */
    public static String getToken(Map<String,String> map){
        JWTCreator.Builder builder = JWT.create();
        // 设置token有关键值对
        map.forEach((k,v)->{
            builder.withClaim(k,v);
        });
        Calendar instance = Calendar.getInstance();
        // 超时时限
        instance.add(Calendar.SECOND,60);
        builder.withExpiresAt(instance.getTime());
        return builder.sign(Algorithm.HMAC256(SIGNATURE)).toString();
    }

    /**
     * 验证token
     * @param token
     */
    public static void verify(String token){
        JWT.require(Algorithm.HMAC256(SIGNATURE)).build().verify(token);
    }

    /**
     * 获取payload
     * @param token
     * @return
     */
    public static DecodedJWT getToken(String token){
        return JWT.require(Algorithm.HMAC256(SIGNATURE)).build().verify(token);
    }
    
}
```

### 2. IOC思想实现拦截器
```java
/**
 * JWTInterceptor
 *
 * @Author: Jane_YH
 * @Version: 1.0
 * @Since: 2022/7/27 15:49
 */
public class JWTInterceptor implements HandlerInterceptor {

    /**
     * 请求拦截具体操作
     * @param request current HTTP request
     * @param response current HTTP response
     * @param handler chosen handler to execute, for type and/or instance evaluation
     * @return
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception{
        // 从请求头中获取到token
        String token = request.getHeader("token");
        ResponseData res = new ResponseData();
        // 鉴别token是否有效
        // 有效则操作继续往下执行
        // 无效则打印错误信息
        try{
            JWTUtil.verify(token);
            return true;
        }catch (SignatureVerificationException e){
            res.setStatus(300);
            res.setMessage("无效签名");
        }catch (TokenExpiredException e){
            res.setStatus(300);
            res.setMessage("token过期");
        }catch (AlgorithmMismatchException e){
            res.setStatus(300);
            res.setMessage("token算法不一致");
        }catch (Exception e) {
            res.setStatus(300);
            res.setMessage("token失效");
        }
        String json = new ObjectMapper().writeValueAsString(res);
        response.setContentType("application/json;charset=UTF-8");
        response.getWriter().print(json);
        return false;
    }
}
```

### 3. 配置拦截路径
```java
/**
 * InterceptorConfig
 *
 * @Author: Jane_YH
 * @Version: 1.0
 * @Since: 2022/7/27 16:06
 */
@Configuration
public class InterceptorConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry){
        registry.addInterceptor(new JWTInterceptor())
                // 要拦截的路径
                .addPathPatterns("/**")
                // 放行的路径
                .excludePathPatterns("/hello/**");
    }
}
```

### 4. 在controller层中的使用
```java
/**
 * HelloController
 *
 * @Author: Jane_YH
 * @Version: 1.0
 * @Since: 2022/7/27 14:42
 */
@Controller
@RequestMapping(value = "/hello")
public class HelloController {

    @PostMapping(value = "/test")
    @ResponseBody
    public ResponseData helloWorld(@RequestBody User user){
        Map<String,Object> map = new HashMap<>();
        // 用户登录信息正确则生成token
        try{
            Map<String,String> payload = new HashMap<>();
            payload.put("username",user.getUsername());
            String token = JWTUtil.getToken(payload);
            map.put("username",user.getUsername());
            map.put("token",token);
            return new ResponseData(map);
        }catch (Exception e){
            map.put("msg",e.getMessage());
            return new ResponseData(500,"服务器异常",map);
        }
    }
}
```

### 5. 用到的响应实体类
```java
/**
 * ResponseData
 *
 * @Author: Jane_YH
 * @Version: 1.0
 * @Since: 2022/7/27 16:48
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ResponseData<T> implements Serializable {
    private Integer status;
    private String message;
    private T data;

    public ResponseData(T data){
        this.status = 200;
        this.message = "请求成功";
        this.data = data;
    }
}
```