---
title: 使用mybatis-plus在多数据源下操作clickhouse
comments: true
mp3: /music/blog.mp3
keywords:
  - mybatis
  - mybatis-plus
  - clickhouse
date: 2021-12-01 19:13:37
updated: 2021-12-01 19:13:37
tags:
categories:
---


## 目的
+ 项目中同时使用多数据源
+ 支持ClickHouse数据的修改和删除
## 背景
+ Mybatis-Plus默认支持ClickHouse的插入和查询
+ dynamic-datasource支持多数据源
## 设计思路
+ 扩展BaseMapper方法用于支持ClickHouse的修改和删除
+ 继承Mybatis-Plus的ServiceImpl重写父类方法根据数据源类型动态执行ClickHouse修改和删除方法

## 实现过程
### 1、添加依赖
```Gradle
dependencies {
    implementation "org.mybatis.spring.boot:mybatis-spring-boot-starter:$mybatisVersion"
    implementation "com.baomidou:mybatis-plus-boot-starter:$mybatisPlusVersion"
    implementation "com.baomidou:dynamic-datasource-spring-boot-starter:$dynamicDatasourceVersion"
    runtimeOnly 'mysql:mysql-connector-java'
    implementation "ru.yandex.clickhouse:clickhouse-jdbc:$clickhouseVersion"
    testImplementation 'org.springframework.boot:spring-boot-starter-test'

    implementation "org.projectlombok:lombok:$lombokVersion"
    annotationProcessor "org.projectlombok:lombok:$lombokVersion"

    testImplementation "org.junit.jupiter:junit-jupiter-api:$junitJupiterVersion"
    testRuntimeOnly "org.junit.jupiter:junit-jupiter-engine:$junitJupiterVersion"
    }
```
主要版本
```Gradle
ext {
    junitJupiterVersion = '5.8.1'
    lombokVersion = '1.18.22'
    clickhouseVersion = '0.3.1-patch'
    mybatisVersion = '2.2.0'
    mybatisPlusVersion = '3.4.3.4'
    dynamicDatasourceVersion = '3.5.0'
}
```

### 2、扩展BaseMapper方法
```Java
package cloud.aispring.mybatisplus.clickhouse.mapper;

import com.baomidou.mybatisplus.core.conditions.Wrapper;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.baomidou.mybatisplus.core.toolkit.Constants;
import java.io.Serializable;
import java.util.Collection;
import java.util.Map;
import org.apache.ibatis.annotations.Param;

/**
 * base mapper 支持 clickhouse自定义方法
 *
 * @author spring.li
 * @version v1.0
 */
public interface ClickhouseBaseMapper<T> extends BaseMapper<T> {

    /**
     * 根据主键id 更新对象
     *
     * @param entity 对象
     * @return boolean
     * @author spring.li
     */
    int updateByIdClickHouse(@Param("et") T entity);


    /**
     * 根据wrapper 更新对象
     *
     * @param entity        对象
     * @param updateWrapper where
     * @return boolean
     * @author spring.li
     */
    int updateClickHouse(@Param("et") T entity, @Param("ew") Wrapper<T> updateWrapper);

    /**
     * 主键删除
     *
     * @param id id
     * @return int 操作影响行
     */
    int deleteByIdClickHouse(Serializable id);

    /**
     * 根据实体(ID)删除
     *
     * @param entity 实体对象
     * @return int 操作影响行数
     * @since 3.4.4
     */
    int deleteByIdClickHouse(T entity);

    /**
     * 删除（根据ID 批量删除）
     *
     * @param idList 主键ID列表(不能为 null 以及 empty)
     * @return int 操作影响行数
     */
    int deleteBatchIdsClickhouse(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);

    /**
     * 根据 columnMap 条件，删除记录
     *
     * @param columnMap 表字段 map 对象
     * @return int 操作影响行数
     */
    int deleteByMapClickhouse(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);

    /**
     * 根据 entity 条件，删除记录
     *
     * @param queryWrapper 实体对象封装操作类（可以为 null,里面的 entity 用于生成 where 语句）
     * @return int 操作影响行数
     */
    int deleteClickhouse(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
}

```

### 3、实现操作ClickHouse方法
1. 定义一个枚举对应所有的方法和执行的SQL
```Java
package cloud.aispring.mybatisplus.clickhouse.enumeration;

import com.baomidou.mybatisplus.core.enums.SqlMethod;
import lombok.Getter;

/**
 * clickhouse 方法 枚举
 *
 * @author spring.li
 * @version v1.0
 * @see SqlMethod
 */
@Getter
public enum ClickhouseSqlMethodEnum {

    /**
     * 物理删除
     */
    DELETE_BY_ID("deleteByIdClickHouse", "根据ID 删除一条数据", "<script>\nALTER TABLE %s DELETE WHERE %s=#{%s}\n</script>"),
    DELETE_BATCH_IDS("deleteBatchIdsClickhouse", "根据 批量 ID 删除数据", "<script>\nALTER TABLE %s DELETE WHERE %s IN (%s)\n</script>"),
    DELETE("deleteClickhouse", "根据 entity 条件删除记录", "<script>\nALTER TABLE %s DELETE %s %s\n</script>"),
    DELETE_BY_MAP("deleteByMapClickhouse", "根据columnMap 条件删除记录", "<script>\nALTER TABLE %s DELETE %s\n</script>"),

    /**
     * 逻辑删除
     */
    LOGIC_DELETE_BY_ID("deleteByIdClickHouse", "根据ID 逻辑删除一条数据", "<script>\nALTER TABLE %s UPDATE %s where %s=#{%s} %s\n</script>"),
    LOGIC_DELETE_BATCH_IDS("deleteBatchIdsClickhouse", "根据ID 批量 逻辑删除数据", "<script>\nALTER TABLE %s UPDATE %s where %s IN (%s) %s\n</script>"),
    LOGIC_DELETE("deleteClickhouse", "根据 entity 条件逻辑删除记录", "<script>\nALTER TABLE %s UPDATE %s %s %s\n</script>"),
    LOGIC_DELETE_BY_MAP("deleteByMapClickhouse", "根据columnMap 条件逻辑删除记录", "<script>\nALTER TABLE %s UPDATE %s %s\n</script>"),

    /**
     * 更新
     */
    UPDATE_BY_ID("updateByIdClickHouse", "根据ID 选择修改数据", "<script>\nALTER TABLE %s UPDATE %s where %s=#{%s} %s\n</script>"),
    UPDATE("updateClickHouse", "根据 whereEntity 条件，更新记录", "<script>\nALTER TABLE %s UPDATE  %s %s %s\n</script>");

    private final String method;
    private final String desc;
    private final String sql;

    ClickhouseSqlMethodEnum(String method, String desc, String sql) {
        this.method = method;
        this.desc = desc;
        this.sql = sql;
    }
}

```
2. 先统一处理Update方法去掉set，原因是ClickHouse的update写法不同
```Java
package cloud.aispring.mybatisplus.clickhouse.injector;

import com.baomidou.mybatisplus.core.injector.AbstractMethod;
import com.baomidou.mybatisplus.core.metadata.TableInfo;
import com.baomidou.mybatisplus.core.toolkit.sql.SqlScriptUtils;

/**
 * clickhouse 更新统一处理
 *
 * @author spring.li
 * @version v1.0
 */
public abstract class AbstractClickHouseUpdate extends AbstractMethod {


    /**
     * SQL Update 语句， 去掉set
     * 错误 : ALTER TABLE tmp UPDATE SET name='123' where id=0
     * 正确 : ALTER TABLE tmp UPDATE name='123' where id=0
     *
     * @param logic  是否逻辑删除注入器
     * @param ew     是否存在 UpdateWrapper 条件
     * @param table  表信息
     * @param alias  别名
     * @param prefix 前缀
     * @return sql
     */
    @Override
    protected String sqlSet(boolean logic, boolean ew, TableInfo table, boolean judgeAliasNull, final String alias, final String prefix) {
        String sqlScript = table.getAllSqlSet(logic, prefix);
        if (judgeAliasNull) {
            sqlScript = SqlScriptUtils.convertIf(sqlScript, String.format("%s != null", alias), true);
        }
        if (ew) {
            sqlScript += NEWLINE;
            sqlScript += SqlScriptUtils.convertIf(SqlScriptUtils.unSafeParam(U_WRAPPER_SQL_SET),
                String.format("%s != null and %s != null", WRAPPER, U_WRAPPER_SQL_SET), false);
        }
        return "<trim prefix=\"\" suffixOverrides=\",\"> " + sqlScript + NEWLINE + "</trim>";
    }


}

```

3. 实现所有的方法，这里以updateById为例
```Java
package cloud.aispring.mybatisplus.clickhouse.injector;

import cloud.aispring.mybatisplus.clickhouse.enumeration.ClickhouseSqlMethodEnum;
import com.baomidou.mybatisplus.core.metadata.TableInfo;
import org.apache.ibatis.executor.keygen.NoKeyGenerator;
import org.apache.ibatis.mapping.MappedStatement;
import org.apache.ibatis.mapping.SqlSource;

/**
 * 根据 id 更新 clickhouse 对象
 *
 * @author spring.li
 * @version v1.0
 */
public class UpdateByIdClickHouse extends AbstractClickHouseUpdate {

    @Override
    public MappedStatement injectMappedStatement(Class<?> mapperClass, Class<?> modelClass, TableInfo tableInfo) {
        ClickhouseSqlMethodEnum sqlMethod = ClickhouseSqlMethodEnum.UPDATE_BY_ID;
        final String additional = optlockVersion(tableInfo) + tableInfo.getLogicDeleteSql(true, true);
        String sql = String.format(sqlMethod.getSql(), tableInfo.getTableName(),
            this.sqlSet(tableInfo.isWithLogicDelete(), false, tableInfo, false, ENTITY, ENTITY_DOT),
            tableInfo.getKeyColumn(), ENTITY_DOT + tableInfo.getKeyProperty(), additional);
        SqlSource sqlSource = languageDriver.createSqlSource(configuration, sql, modelClass);
        return this.addInsertMappedStatement(mapperClass, modelClass,
            sqlMethod.getMethod(), sqlSource, new NoKeyGenerator(), null, null);
    }
}

```

4. 继承DefaultSqlInjector注入上面自定义方法
```Java
package cloud.aispring.mybatisplus.clickhouse.injector;

import com.baomidou.mybatisplus.core.injector.AbstractMethod;
import com.baomidou.mybatisplus.core.injector.DefaultSqlInjector;
import com.baomidou.mybatisplus.core.metadata.TableInfo;
import java.util.List;
import org.springframework.stereotype.Component;


/**
 * 注册 clickhouse方法到 mybatis-plus
 *
 * @author spring.li
 * @version v1.0
 */
@Component
public class ClickHouseSqlInjector extends DefaultSqlInjector {

    @Override
    public List<AbstractMethod> getMethodList(Class<?> mapperClass, TableInfo tableInfo) {
        // 先要通过父类方法，获取到原有的集合，不然会自带的通用方法会失效的
        List<AbstractMethod> methodList = super.getMethodList(mapperClass, tableInfo);

        // 添加自定义方法类
        if (tableInfo.havePK()) {
            methodList.add(new UpdateByIdClickHouse());
            methodList.add(new DeleteBatchIdsClickHouse());
        }
        methodList.add(new UpdateClickHouse());
        methodList.add(new DeleteByIdClickHouse());
        methodList.add(new DeleteClickHouse());
        methodList.add(new DeleteByMapClickHouse());
        return methodList;
    }

}

```

5. 继承ServiceImpl动态执行方法
```Java
package cloud.aispring.mybatisplus.clickhouse.service.impl;

import cloud.aispring.mybatisplus.clickhouse.enumeration.ClickhouseSqlMethodEnum;
import cloud.aispring.mybatisplus.clickhouse.mapper.ClickhouseBaseMapper;
import com.baomidou.dynamic.datasource.DynamicRoutingDataSource;
import com.baomidou.dynamic.datasource.ds.ItemDataSource;
import com.baomidou.dynamic.datasource.toolkit.DynamicDataSourceContextHolder;
import com.baomidou.mybatisplus.core.conditions.Wrapper;
import com.baomidou.mybatisplus.core.enums.SqlMethod;
import com.baomidou.mybatisplus.core.toolkit.StringPool;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.baomidou.mybatisplus.extension.toolkit.SqlHelper;
import com.zaxxer.hikari.HikariDataSource;
import java.io.Serializable;
import java.util.Collection;
import java.util.Map;
import javax.sql.DataSource;
import org.springframework.beans.factory.annotation.Autowired;

/**
 * repository clickhouse 实现根据数据源自动调整内部实现调用mapper方法
 *
 * @author spring.li
 * @version v1.0
 */
public class ClickhouseServiceImpl<M extends ClickhouseBaseMapper<T>, T> extends ServiceImpl<M, T> {

    @Autowired
    private DataSource dynamicRoutingDataSource;

    private boolean notClickhouse() {
        String clickhouseDriverClassName = "ClickHouseDriver";
        if (dynamicRoutingDataSource instanceof DynamicRoutingDataSource) {
            DataSource dataSource = ((DynamicRoutingDataSource) dynamicRoutingDataSource).getDataSource(DynamicDataSourceContextHolder.peek());
            if (dataSource instanceof ItemDataSource) {
                if (((ItemDataSource) dataSource).getDataSource() instanceof HikariDataSource) {
                    return !((HikariDataSource) ((ItemDataSource) dataSource).getDataSource()).getDriverClassName().contains(clickhouseDriverClassName);
                }
            }
        }

        return true;
    }

    /**
     * 获取mapperStatementId
     *
     * @param sqlMethod 方法名
     * @return 命名id
     * @since 3.4.0
     */
    @Override
    public String getSqlStatement(SqlMethod sqlMethod) {
        if (notClickhouse()) {
            return super.getSqlStatement(sqlMethod);
        }
        ClickhouseSqlMethodEnum clickhouseSqlMethod = null;
        if (SqlMethod.UPDATE_BY_ID.equals(sqlMethod)) {
            clickhouseSqlMethod = ClickhouseSqlMethodEnum.UPDATE_BY_ID;
        }
        if (clickhouseSqlMethod == null) {
            return super.getSqlStatement(sqlMethod);
        }
        return mapperClass.getName() + StringPool.DOT + clickhouseSqlMethod.getMethod();
    }

    @Override
    public boolean updateById(T entity) {
        if (notClickhouse()) {
            return super.updateById(entity);
        }
        return SqlHelper.retBool(getBaseMapper().updateByIdClickHouse(entity));
    }

    /**
     * 根据 whereEntity 条件，更新记录
     *
     * @param entity        实体对象
     * @param updateWrapper 实体对象封装操作类 {@link com.baomidou.mybatisplus.core.conditions.update.UpdateWrapper}
     */
    @Override
    public boolean update(T entity, Wrapper<T> updateWrapper) {
        if (notClickhouse()) {
            return super.update(entity, updateWrapper);
        }
        return SqlHelper.retBool(getBaseMapper().updateClickHouse(entity, updateWrapper));
    }

    /**
     * 根据 ID 删除
     *
     * @param id 主键ID
     */
    @Override
    public boolean removeById(Serializable id) {
        if (notClickhouse()) {
            return super.removeById(id);
        }
        return SqlHelper.retBool(getBaseMapper().deleteByIdClickHouse(id));
    }

    /**
     * 根据实体(ID)删除
     *
     * @param entity 实体
     * @since 3.4.4
     */
    @Override
    public boolean removeById(T entity) {
        if (notClickhouse()) {
            return super.removeById(entity);
        }
        return SqlHelper.retBool(getBaseMapper().deleteByIdClickHouse(entity));
    }

    /**
     * 根据 columnMap 条件，删除记录
     *
     * @param columnMap 表字段 map 对象
     */
    @Override
    public boolean removeByMap(Map<String, Object> columnMap) {
        if (notClickhouse()) {
            return super.removeByMap(columnMap);
        }
        return SqlHelper.retBool(getBaseMapper().deleteByMapClickhouse(columnMap));
    }

    /**
     * 根据 entity 条件，删除记录
     *
     * @param queryWrapper 实体包装类 {@link com.baomidou.mybatisplus.core.conditions.query.QueryWrapper}
     */
    @Override
    public boolean remove(Wrapper<T> queryWrapper) {
        if (notClickhouse()) {
            return super.remove(queryWrapper);
        }
        return SqlHelper.retBool(getBaseMapper().deleteClickhouse(queryWrapper));
    }

    /**
     * 删除（根据ID 批量删除）
     *
     * @param idList 主键ID列表
     */
    @Override
    public boolean removeByIds(Collection<? extends Serializable> idList) {
        if (notClickhouse()) {
            return super.removeByIds(idList);
        }
        return SqlHelper.retBool(getBaseMapper().deleteBatchIdsClickhouse(idList));
    }
}

```

### 4、使用方法
1. 新建PO对象
```Java
package cloud.aispring.mybatisplus.clickhouse.test.po;

import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.ToString;
import lombok.experimental.SuperBuilder;

/**
 * 用户
 *
 * @author spring.li
 * @version v1.0
 * @date Created in 2021/11/30 11:14
 */
@Data
@SuperBuilder
@ToString
@TableName("user")
@NoArgsConstructor
@AllArgsConstructor
public class User {

    @TableId("id")
    Integer id;

    @TableField("name")
    String name;

    @TableField("is_deleted")
    Boolean isDeleted;

}

```

2. 新建UserMapper
```Java
package cloud.aispring.mybatisplus.clickhouse.test.mapper;

import cloud.aispring.mybatisplus.clickhouse.mapper.ClickhouseBaseMapper;
import cloud.aispring.mybatisplus.clickhouse.test.po.User;
import org.apache.ibatis.annotations.Mapper;

/**
 * 用户 mapper
 *
 * @author spring.li
 * @version v1.0
 * @date Created in 2021/11/30 11:14
 */
@Mapper
public interface UserMapper extends ClickhouseBaseMapper<User> {

}

```

3. 新建UserService
```Java
package cloud.aispring.mybatisplus.clickhouse.test.service;


import cloud.aispring.mybatisplus.clickhouse.test.po.User;
import com.baomidou.mybatisplus.extension.service.IService;

/**
 * user service
 *
 * @author spring.li
 * @version v1.0
 * @date Created in 2021/11/29 11:24
 */
public interface UserService extends IService<User> {

    void testMySql();

    void testClickhouse();

}

```

4. 新建UserServiceImpl
```Java
package cloud.aispring.mybatisplus.clickhouse.test.service.impl;


import cloud.aispring.mybatisplus.clickhouse.service.impl.ClickhouseServiceImpl;
import cloud.aispring.mybatisplus.clickhouse.test.mapper.UserMapper;
import cloud.aispring.mybatisplus.clickhouse.test.po.User;
import cloud.aispring.mybatisplus.clickhouse.test.service.UserService;
import com.baomidou.dynamic.datasource.annotation.DS;
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.baomidou.mybatisplus.core.conditions.update.LambdaUpdateWrapper;
import com.baomidou.mybatisplus.core.conditions.update.UpdateWrapper;
import java.util.Collections;
import java.util.HashMap;
import java.util.Map;
import java.util.UUID;
import lombok.SneakyThrows;
import org.springframework.stereotype.Service;

@Service
public class UserServiceImpl extends ClickhouseServiceImpl<UserMapper, User> implements UserService {

    @DS("clickhouse")
    @Override
    public void testClickhouse() {
        this.testMySql();
    }

    @SneakyThrows
    private void assertName(User user){
        Thread.sleep(200);
        assert user.getName().equals(this.getById(user.getId()).getName());
    }

    @SneakyThrows
    private void assertNull(Integer userId){
        Thread.sleep(200);
        assert this.getById(userId) == null;
    }

    @DS("master")
    @Override
    public void testMySql() {
        // 清除所有数据
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper.lambda().isNotNull(User::getId);
        this.remove(queryWrapper);
        Integer userId = 0;
        User user = User.builder().id(userId).name(UUID.randomUUID().toString()).build();
        this.save(user);
        assertName(user);
        user.setName(UUID.randomUUID().toString());
        this.updateById(user);
        assertName(user);
        user.setName(UUID.randomUUID().toString());
        this.updateBatchById(Collections.singletonList(user));
        assertName(user);
        user.setName(UUID.randomUUID().toString());
        LambdaUpdateWrapper<User> updateWrapper = (new UpdateWrapper<User>()).lambda();
        updateWrapper.set(User::getName, user.getName());
        updateWrapper.eq(User::getId, user.getId());
        this.update(updateWrapper);
        assertName(user);
        user.setName(UUID.randomUUID().toString());
        this.saveOrUpdateBatch(Collections.singletonList(user));
        assertName(user);
        this.removeById(userId);
        assertNull(userId);
        this.saveOrUpdateBatch(Collections.singletonList(user));
        assertName(user);
        this.removeById(userId);
        assertNull(userId);
        this.save(user);
        this.removeByIds(Collections.singletonList(userId));
        assertNull(userId);
        this.save(user);
        Map<String, Object> map = new HashMap<>();
        map.put("id", user.getId());
        this.removeByMap(map);
        assertNull(userId);
    }
}

```

5. 配置文件
```Yaml
spring:
  datasource:
    dynamic:
      primary: master
      strict: true
      datasource:
        master:
          url: jdbc:mysql://mysql:3306/user?autoReconnect=true&useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai&useSSL=false
          username: root
          password: 123456
          driver-class-name: com.mysql.cj.jdbc.Driver
          hikari:
            connection-test-query: SELECT 1
            connection-timeout: 30000
            idle-timeout: 600000
            max-lifetime: 1800000
            maximum-pool-size: 20
            minimum-idle: 10
        clickhouse:
          url: jdbc:clickhouse://192.168.1.111:30123/default?characterEncoding=utf8&useSSL=false&serverTimezone=UTC&rewriteBatchedStatements=true&allowPublicKeyRetrieval=true
          username: default
          password: ''
          driver-class-name: ru.yandex.clickhouse.ClickHouseDriver

mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

## 注意事项

+ ClickHouse的操作不支持事务
+ 条件删除时必须有条件，没有条件则报错
+ ClickHouseSqlInjector要注入Bean
## 总结
+ 源代码：https://github.com/aispringli/mybatis-plus-clickhouse
+ Jar https://mvnrepository.com/artifact/cloud.aispring/mybatis-plus-clickhouse/3.0
```
// https://mvnrepository.com/artifact/cloud.aispring/mybatis-plus-clickhouse
implementation 'cloud.aispring:mybatis-plus-clickhouse:3.0'
```
