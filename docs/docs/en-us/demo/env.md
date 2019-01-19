# Try the DTX demo

## 一、Prepare MariaDB (MySQL) Database

for TxClient create db `txlcn-demo` and execute this:      
```$xslt
DROP TABLE IF EXISTS `t_demo`;
CREATE TABLE `t_demo`  (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `demo_field` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `group_id` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `unit_id` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `app_name` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `create_time` datetime(0) NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 26 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

```

for TxManager create db `tx-manager` and execute this:   
```$xslt
DROP TABLE IF EXISTS `t_tx_exception`;
CREATE TABLE `t_tx_exception`  (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `group_id` varchar(32) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  `unit_id` varchar(32) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  `mod_id` varchar(32) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  `transaction_state` tinyint(4) NULL DEFAULT NULL,
  `registrar` tinyint(4) NULL DEFAULT NULL COMMENT '-1 未知 0 Manager 通知事务失败， 1 client询问事务状态失败2 事务发起方关闭事务组失败',
  `ex_state` tinyint(4) NULL DEFAULT NULL COMMENT '0 待处理 1已处理',
  `create_time` datetime(0) NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 967 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;

```

## 二、Launch the TxManager
1. MariaDB launched already.  
2. to launch Redis.  
3. Main configuration of TxManager. [details](setting/manager.html)
```properties
spring.application.name=tx-manager
server.port=8069

spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://ip:port/tx-manager?characterEncoding=UTF-8
spring.datasource.username=user
spring.datasource.password=passwd

mybatis.configuration.map-underscore-to-camel-case=true
mybatis.configuration.use-generated-keys=true

spring.redis.host=127.0.0.1
spring.redis.port=6379
spring.redis.password=

```

4. launch the TxManager
execute the executable jar. [详情](https://bbs.txlcn.org/viewtopic.php?id=12)
![tx-manager](../../../img/docs/tx_manager.png)

## 三、Prepare register central for micro services

* launch ZooKeeper for Dubbo
* launch Consul for SpringCloud

## 四、micro service demo（TxClient）
[Dubbo-Demo](dubbo.html)

[SpringCloud-Demo](springcloud.html)

## 五、check the DTX
（1）normal to commit the DTX

request RestApi`/txlcn?value=the-value` provide by DTX starter。will commit all the local transaction.  
![result](../../../img/docs/result.png)

（2）rollback th DTX

modify the DTX starter's business code，throws BusinessException before return the result，request the restApi again! The DTX starter's local transaction rollback caused BusinessException，Service D、Service E，rollback by TxManager's coordination。  
![error_result](../../../img/docs/error-result.png)
