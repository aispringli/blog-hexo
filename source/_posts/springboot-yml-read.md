---
title: Spring Boot读取自定义yml文件
comments: true
mp3: 'http://link.hhtjim.com/163/425570952.mp3'
cover: /img/cover/springboot-yml-read.jpeg
tags:
  - SpringBoot
  - yml
categories:
  - SpringBoot
date: 2020-05-27 22:06:28
updated: 2020-05-27 22:06:28
keywords:
---

背景：Spring项目模块块开发，准备定义一个公共模块，定义常用的错误代码和提升信息，定义信息放在ErrorCodes.yml文件，直接用Spring Boot读取，子模块能够使用公共模块并能够扩展ErrorCodes-*.yml文件

## 使用教程

{% asset_img springboot-yml-read.jpeg %}

###  新建文件
在resources资源文件夹下新建文件 ErrorCodes.yml，格式如下

``` yml
aispring:
  errors:
    - code: 0
      message: success
    - code: -1
      message: 未知错误
```

这个时候直接用Spring Boot像读取application.yml中的变量是读取不到的，因为Spring Boot暂只支持直接读取application*.yml，像我们自定义的这种文件默认会当作properties来读取，需要我们自己写一个PropertySourceFactory自定义读取文件

### 自定义读取

``` java
package cloud.aispring.core.factory;

import org.springframework.beans.factory.config.YamlPropertiesFactoryBean;
import org.springframework.boot.env.YamlPropertySourceLoader;
import org.springframework.core.env.PropertiesPropertySource;
import org.springframework.core.env.PropertySource;
import org.springframework.core.io.support.DefaultPropertySourceFactory;
import org.springframework.core.io.support.EncodedResource;
import org.springframework.lang.Nullable;

import java.io.IOException;
import java.util.Arrays;
import java.util.List;
import java.util.Optional;
import java.util.Properties;

/**
 * @author spring.li
 * @date 2020/05/09
 **/
public class CompositePropertySourceFactory extends DefaultPropertySourceFactory {

    private List<String> yamlFiles = Arrays.asList(".yml", ".yaml");

    /**
     * 判断 yml 格式文件
     *
     * @param sourceName 文件名
     * @return 是否是yml文件
     */
    private boolean checkYamlFile(String sourceName) {
        for (String yamlFile : yamlFiles) {
            if (sourceName.endsWith(yamlFile)) {
                return true;
            }
        }
        return false;
    }

    @Override
    public PropertySource<?> createPropertySource(@Nullable String name, EncodedResource resource) throws IOException {
        String sourceName = Optional.ofNullable(name).orElse(resource.getResource().getFilename());
        assert sourceName != null;
        if (!resource.getResource().exists()) {
            return new PropertiesPropertySource(sourceName, new Properties());
        } else if (checkYamlFile(sourceName)) {
            Properties propertiesFromYaml = loadYaml(resource);
            return new PropertiesPropertySource(sourceName, propertiesFromYaml);
        } else {
            return super.createPropertySource(name, resource);
        }
    }

    /**
     * load yaml file to properties
     *
     * @param resource resource
     * @return Properties
     */
    private Properties loadYaml(EncodedResource resource) {
        YamlPropertiesFactoryBean factory = new YamlPropertiesFactoryBean();
        factory.setResources(resource.getResource());
        factory.afterPropertiesSet();
        return factory.getObject();
    }
}
```

### Bean读取
对应Bean读取信息，注意使用自定义的factory

``` java
package cloud.aispring.core.utils;

import cloud.aispring.core.exception.ErrorCode;
import cloud.aispring.core.factory.CompositePropertySourceFactory;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;

import java.util.List;

/**
 * @author spring.li
 * @date 2020/05/09
 **/
@Component
@PropertySource(value = {"classpath:ErrorCodes.yml"}, factory = CompositePropertySourceFactory.class)
@ConfigurationProperties(prefix = "aispring")
public class ErrorCodeUtils {

    List<ErrorCode> errors;

    public List<ErrorCode> getErrors() {
        return errors;
    }

    public void setErrors(List<ErrorCode> errors) {
        this.errors = errors;
    }
}
```

### Test 
Test测试一下，需要注意SpringBootTest(classes = CoreApplication.class)启动整个程序才可以

``` java
package cloud.aispring.core;

import cloud.aispring.core.utils.ErrorCodeUtils;
import cloud.aispring.core.test.IntegrationTest;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;

/**
 * @author spring.li
 * @date 2020/05/09
 **/
@SpringBootTest(classes = CoreApplication.class)
@IntegrationTest
@AutoConfigureMockMvc
class TestErrorCode {

    @Autowired
    ErrorCodeUtils errorCodeUtils;

    private final Log logger = LogFactory.getLog(getClass());

    @Test
    void testErrorCodeUtils() {
        logger.info(errorCodeUtils.getErrors().size());
        assert errorCodeUtils.getErrors().size() == 2;
    }
}
```

### 读取多个文件
```
@Bean
    public static PropertySourcesPlaceholderConfigurer properties() throws IOException {
        PropertySourcesPlaceholderConfigurer configurer = new PropertySourcesPlaceholderConfigurer();
        YamlPropertiesFactoryBean yaml = new YamlPropertiesFactoryBean();
        //class 引入 通配符 匹配文件
        ResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
        Resource[] resources1 = resolver.getResources("classpath*:ErrorCodes/*.yml");
        // 默认
        Resource[] resources2 = resolver.getResources(baseErrorCodesFileName);
        if (resources1.length == 0 && resources2.length == 0) {
            return configurer;
        }
        Resource[] resources = new Resource[resources1.length + resources2.length];
        System.arraycopy(resources1, 0, resources, 0, resources1.length);
        System.arraycopy(resources2, 0, resources, resources1.length, resources2.length);
        Properties[] properties = new Properties[resources.length];
        for (int i = 0; i < resources.length; i++) {
            yaml.setResources(resources[i]);
            properties[i] = yaml.getObject();
        }
        configurer.setPropertiesArray(properties);
        return configurer;
    }
```
