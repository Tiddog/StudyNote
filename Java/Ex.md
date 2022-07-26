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