# radar-cryptogram

配置文件密码加密步骤

1、添加依赖（运维无需关注）

        <!--配置文件密码加密-->
        <dependency>
            <groupId>com.github.ulisesbocchio</groupId>
            <artifactId>jasypt-spring-boot</artifactId>
            <version>1.18</version>
        </dependency>
        <dependency>
            <groupId>org.jasypt</groupId>
            <artifactId>jasypt</artifactId>
            <version>1.9.2</version>
        </dependency>




2、在application.yml文件中添加以下配置信息

    # 配置文件密码加密配置
    jasypt:
      encryptor:
        password: EbfYkitulv73I2p0mXI50JMXoaxZTKJ7 # 秘钥
        algorithm: PBEWithMD5AndDES # 加密算法
        iv-generator-classname: org.jasypt.iv.NoIvGenerator

（1）从3.0.0jasypt-spring-boot 版本开始，默认的加密/解密算法已更改为PBEWITHHMACSHA512ANDAES_256；

（2）上述algorithm 不配置的话，其默认的秘钥也是 PBEWithMD5AndDES；

3、获取密文

    java -cp /Users/shiyanfei/zlb/repository/repository-zlb/org/jasypt/jasypt/1.9.2/jasypt-1.9.2.jar org.jasypt.intf.cli.JasyptPBEStringEncryptionCLI input="Mysql@1234" password=idss@2021 algorithm=PBEWithMD5AndDES

终端执行上述命令会生成密文，其中：

- /Users/shiyanfei/zlb/repository/repository-zlb/org/jasypt/jasypt/1.9.2/ 是 jasypt-1.9.2.jar 的路径（Linux环境中应该是在/lib包下面），根据需求修改；
- input 是明文密码，每一个密码都需要执行一次；
- password 是秘钥。



4、修改原来的密码配置

原来的明文密码，改为 ENC(xxx) ,其中xxx是密文。

1）MySQL

    spring:
      datasource:
        url: jdbc:mysql://10.10.26.129:3306/ueba?autoReconnect=true&useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true&&useSSL=false
        driver-class-name: com.mysql.jdbc.Driver
        username: root
        password: ENC(2RP1Vdsa+2wdSOgu2biAJkTCU9fnkUGD) 

2）Redis

    spring:
      redis:
        database: 0
        host: 10.20.24.48
        port: 6379
        password: ENC(JjPTg5GOsjV9ZBIQ2CaHr+96UgMKBgIT) 



5、添加注解

启动类上添加@EnableEncryptableProperties







优化

1、独立小工具生成密文（秘钥写死在代码中）；

2、以上2中的配置使用自动配置，代码是：

    /************************ CHANGE REPORT HISTORY ******************************\
     ** Product VERSION,UPDATED BY,UPDATE DATE                                     *
     *   DESCRIPTION OF CHANGE: modify(M),add(+),del(-)                            *
     *-----------------------------------------------------------------------------*
     * V3.0.12,shiyanfei,2021-09-14
     * create 
     *
     *************************** END OF CHANGE REPORT HISTORY ********************/
    package com.idss.radar.common.ums.bean;
    
    import org.jasypt.encryption.StringEncryptor;
    import org.jasypt.encryption.pbe.PooledPBEStringEncryptor;
    import org.jasypt.encryption.pbe.config.SimpleStringPBEConfig;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    
    /**
     * @author : shiyanfei
     * @description : <p></p>
     * @see : com.idss.radar.common.ums.bean
     * @since : 2021-09-14
     */
    @Configuration
    public class EncryptorConfig {
    
        @Bean("jasyptStringEncryptor")
        public StringEncryptor jasyptStringEncryptor() {
            PooledPBEStringEncryptor encryptor = new PooledPBEStringEncryptor();
            SimpleStringPBEConfig config = new SimpleStringPBEConfig();
            config.setPassword("EbfYkitulv73I2p0mXI50JMXoaxZTKJ7");
            config.setAlgorithm("PBEWithMD5AndDES");
    //        config.setKeyObtentionIterations("1000");
            config.setPoolSize("1");
    //        config.setProviderName("SunJCE");
    //        config.setSaltGeneratorClassName("org.jasypt.salt.RandomSaltGenerator");
    //        config.setIvGeneratorClassName("org.jasypt.iv.RandomIvGenerator");
    //        config.setStringOutputType("base64");
            encryptor.setConfig(config);
            return encryptor;
        }
    }



3、密文生成程序



