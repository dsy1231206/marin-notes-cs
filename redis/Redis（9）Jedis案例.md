# Redis（9）Jedis案例

##### 一个案例：手机发送验证码，2分钟有效

###### 1、输入手机号，点击发送后生成6位数字，2分钟有效

​	Java里使用Random类生成6位数字

###### 2、验证码2分钟有效

​	把验证码放到redis里面，设置2分钟过期

###### 3、判断验证码是否一致

​	从redis获取验证码和输入的验证码比较

###### 4、每个手机号每天只能输入三次

​	incr每次发送之后+1，大于2的时候就不能发送了

```java
package com.dsy.jedis;

import redis.clients.jedis.Jedis;

import java.util.Random;

public class VerifyCode {
    public static void main(String[] args) {
        String code = getVerifyCode();
        System.out.println(code);

    }

    //生成6位验证码
    public static String getVerifyCode(){
        Random random = new Random();
        StringBuilder sb = new StringBuilder();
        for(int i = 0; i < 6; i++){
            sb.append(random.nextInt(10));
        }
        return sb.toString();
    }

    //每个手机最多发送三次，120秒有效
    public static void verify(String phone){
        //连接redis
        Jedis jedis = new Jedis("192.168.0.1",6379);
        //拼接key
        //手机发送次数key
        String countKey = "VerifyCode" + phone + ":count";
        //验证码key拼接
        String codeKey = "VerifyCode" + phone + ":code";

        //每天只能发送三次
        String count = jedis.get(countKey);
        if(count == null){
            //没有发送过
            jedis.setex(countKey,24 * 60 * 60,"1");
        }else if(Integer.parseInt(count) <= 2){
            //发送次数+1
            jedis.incr(countKey);
        }else if(Integer.parseInt(count) > 2){
            //已经发送了3次，不能发送了
            System.out.println("今日尝试次数超过3次。");
            jedis.close();
        }

        //发送验证码要放到redis里去
        String vCode = getVerifyCode();
        jedis.setex(codeKey,120,vCode);


    }

    //验证码校验
    public static void getRedisCode(String phone,String code){
        //连接redis
        Jedis jedis = new Jedis("192.168.0.1",6379);
        
        //验证码key
        String codeKey = "VerifyCode" + phone + ":code";
        String redisCode = jedis.get(codeKey);
        //判断
        if(redisCode.equals(code)){
            System.out.println("成功");
        }else{
            System.out.println("失败");
        }
        jedis.close();

    }

}
```



##### SpringBoot整合Redis

在config包里写一个RedisConfig，网上可查

SpringBoot自动注册了RedisTemplate这个bean，直接@Autowired注入即可使用