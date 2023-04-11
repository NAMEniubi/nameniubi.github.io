---
title: MapStruct对象转换用法
description: 这是一个副标题
slug: mapstruct
date: 2021-08-30 19:47:34 +0800
image: cover.jpg
categories:
    - 代码
tags:
    - java
---

![image.png](https://file.yfniubi.club/blog/image_1630319882433.png)
官网地址：http://mapstruct.org/

MapStruct是一个代码生成器，简化了不同的Java Bean之间映射的处理，所以映射指的就是从一个实体变化成一个实体。例如我们在实际开发中，DAO层的实体和一些数据传输对象(DTO)，大部分属性都是相同的，只有少部分的不同，通过mapStruct，可以让不同实体之间的转换变的简单。我们只需要按照约定的方式进行配置即可。

MapStruct是一个可以处理注解的Java编译器插件，可以在命令行中使用，也可以在IDE中使用。MapStruct有一些默认配置，但是也为用户提供了自己进行配置的途径。

### 1 开发环境搭建--基于Maven

MapStruct主要由两部分组成：

org.mapstruct:mapstruct：包含了一些必要的注解，例如@Mapping。我们使用的JDK版本高于1.8，当我们在pom里面导入依赖时候，建议使用坐标是：org.mapstruct:mapstruct-jdk8，这可以帮助我们利用一些Java8的新特性。  

org.mapstruct:mapstruct-processor：注解处理器，根据注解自动生成mapper的实现。
```java
		<org.mapstruct.version>1.3.0.Final</org.mapstruct.version>

		<!--mapStruct依赖-->
		<dependency>
			<groupId>org.mapstruct</groupId>
			<artifactId>mapstruct-jdk8</artifactId>
			<version>${org.mapstruct.version}</version>
		</dependency>
		<dependency>
			<groupId>org.mapstruct</groupId>
			<artifactId>mapstruct-processor</artifactId>
			<version>${org.mapstruct.version}</version>
			<scope>provided</scope>
		</dependency>

```

### 2 MapStruct2分钟入门

下面的代码演示了如何使用Map Struct实现Java  Bean之间的映射。假设我们有一个表示汽车的类Car，并且还有一个数据传输对象(DTO)CarDTO。

这两个类非常相似，只是表示作为数量的属性名称是不同的并且，在Car对象中，表示汽车类型的字段是一个枚举，而在CarDTO中，直接使用字符串表示。

Car.java
```java
public class Car {
    private String make;
    private int numberOfSeats;
    private CarType type;
    //constructor, getters, setters etc.
    }
    
  static enum CarType {
    SEDAN
}
```
CarDTO.java
```java
public class CarDto {
 
    private String make;
    private int seatCount;
    private String type;
 
    //constructor, getters, setters etc.
}
```
#### 2.1 Mapper接口

要生成一个CarDTO与Car对象相互转换的映射器，我们需要定义一个mapper接口。

CarMapper.java
```java
@Mapper
public interface CarMapper {
 
    CarMapper INSTANCE = Mappers.getMapper( CarMapper.class );
 
    @Mapping(source = "numberOfSeats", target = "seatCount")
    CarDto carToCarDto(Car car);
}
```
1、@Mapper注解标记这个接口作为一个映射接口，并且是编译时MapStruct处理器的入口。

2、真正实现映射的方法需要源对象作为参数，并返回目标对象。映射方法的名字是随意的。对于在源对象和目标对象中，属性名字不同的情况，可以通过@Mapping注解来配置这些名字。我们也可以将源类型与目标类型中类型不同的参数进行转换，在这里就是通过type属性将枚举类型转换为一个字符串。当然在一个接口里可以定义多个映射方法。MapStruct都会为其生成一个实现。

3、自动生成的接口的实现可以通过Mapper的class对象获取。按照惯例，接口中会声明一个成员变量INSTANCE，从而让客户端可以访问Mapper接口的实现。
#### 2.2 @Mapper注解的componentModel属性
componentModel属性用于指定自动生成的接口实现类的组件类型。这个属性支持四个值：

default: 这是默认的情况，mapstruct不使用任何组件类型, 可以通过Mappers.getMapper(Class)方式获取自动生成的实例对象。

cdi: the generated mapper is an application-scoped CDI bean and can be retrieved via @Inject

spring: 生成的实现类上面会自动添加一个@Component注解，可以通过Spring的 @Autowired方式进行注入

jsr330: 生成的实现类上会添加@javax.inject.Named 和@Singleton注解，可以通过 @Inject注解获取。
```java
//设置组件模型为Spring
@Mapper(componentModel = "spring")
public interface CarMapper {
 
    CarMapper INSTANCE = Mappers.getMapper( CarMapper.class );
 
    @Mapping(source = "numberOfSeats", target = "seatCount")
    CarDto carToCarDto(Car car);
}
```
#### 2.3 编译

因为MapStruct是以Java编译器插件的形式来处理注解，生成mapper接口的实现。因此在使用之前我们必须手工的编译(IDE的自动编译功能不会使用到MapStruct这个插件功能)。

执行maven命令：mvn compile

可以看到在target目录来多个一个类CarMapperImpl.class，这个类实际上就是map struct插件自动帮助我们根据CarMapper接口生成的实现类。我们可以通过IDE的反编译功能查看自动生成的实现类的源码。通过反编译的源码，我们可以看出，对于属性名称不同的情况(seatCount与numberOfSeats)、以及属性类型不同(枚举类型的type与字符串类型的type)都自动帮助我们转换了。对于属性名称不同的转换，我们是通过在@Mapping注解指定的，而不同属性类型的转换，这是MapStruct的默认配置。

### 3 多个属性的映射

直接引用源参数的映射方法
```java
    @Mapping(source = "person.description", target = "description")
    @Mapping(source = "address.houseNo", target = "houseNumber")
    DeliveryAddressDto personAndAddressToDeliveryAddressDto(Person person, Address address);
```
### 4 更新现有的Bean实例
场景：将一个Bean的属性更新到另外一个bean的同名属性
```java
   void updateCarFromDto(CarDto carDto, @MappingTarget Car car);
```
### 5 继承反转配置
场景：已经存在一个Car映射成CarDTO的方法，现在需要将CarDTO映射成Car。
```java
	@Mapping(source = "make", target = "manufacturer")
	@Mapping(source = "numberOfSeats", target = "seatCount")
	CarDto carToCarDto(Car car);

	@InheritInverseConfiguration(name = "carToCarDto")
	Car carDTOTocar(CarDto carDto);
```
### 6 隐式类型转换
在许多情况下，MapStruct会自动处理类型转换。例如，如果一个属性int在源Bean中是类型但String在目标Bean中是类型，则生成的代码将分别通过分别调用String#valueOf(int)和来透明地执行转换Integer#parseInt(String)。
当前，以下转换将自动应用：
之间的所有Java基本数据类型及其相应的包装类型，例如之间int和Integer，boolean和Boolean等。之间的所有Java基本类型和包装类之间，例如int和long或byte和Integer。
从较大的数据类型转换为较小的数据类型（例如从long到int）可能会导致值或精度损失。在Mapper和MapperConfig注释有一个方法typeConversionPolicy来控制警告/错误。由于向后兼容的原因，默认值为“ ReportingPolicy.IGNORE”。
所有Java基本类型之间（包括其包装）和String之间，例如int和String或Boolean和String。java.text.DecimalFormat可以指定理解的格式字符串。
#### 6.1 从int到String的转换
```java
@Mapper
public interface CarMapper {

    @Mapping(source = "price", numberFormat = "$#.00")
    CarDto carToCarDto(Car car);

    @IterableMapping(numberFormat = "$#.00")
    List<String> prices(List<Integer> prices);
}
```
#### 6.2 从BigDecimal到String的转换

```java
@Mapper
public interface CarMapper {

    @Mapping(source = "power", numberFormat = "#.##E0")
    CarDto carToCarDto(Car car);

}
```
#### 6.3 从日期到字符串的转换
```java
@Mapper
public interface CarMapper {

    @Mapping(source = "manufacturingDate", dateFormat = "dd.MM.yyyy")
    CarDto carToCarDto(Car car);

    @IterableMapping(dateFormat = "dd.MM.yyyy")
    List<String> stringListToDateList(List<Date> dates);
}
```
### 7 默认值和常量
分别可以通过@Mapping的defaultValue和constant属性指定，当source对象的属性值为null时，如果有指定defaultValue将注入defaultValue的设定的值。constant属性通用用于给target属性注入常量值。

```java
    @Mapping(target = "stringProperty", source = "stringProp", defaultValue = "undefined")
    @Mapping(target = "longProperty", source = "longProp", defaultValue = "-1")
    @Mapping(target = "stringConstant", constant = "Constant Value")
    @Mapping(target = "integerConstant", constant = "14")
    @Mapping(target = "longWrapperConstant", constant = "3001")
    @Mapping(target = "dateConstant", dateFormat = "dd-MM-yyyy", constant = "09-01-2014")
    @Mapping(target = "stringListConstants", constant = "jack-jill-tom")
    Target sourceToTarget(Source s);
```
如果为s.getStringProp() == null，则将target属性stringProperty设置为"undefined"而不是应用来自s.getStringProp()的值。如果为s.getLongProperty() == null，则目标属性longProperty将设置为-1。将String "Constant Value"设置为目标属性stringConstant。该值"3001"被类型转换为 targe类的longWrapperConstant属性。该常量"jack-jill-tom"将破折号分隔的列表映射到List。


### 8 踩坑记录
#### 8.1 于lombook同时使用时冲突
该问题导致编译后生成的实体类中没有set属性
在pom文件中添加以下配置即可

``` maven
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <encoding>UTF-8</encoding>
                    <annotationProcessorPaths>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                            <version>${lombok.version}</version>
                        </path>
                        <path>
                            <groupId>org.mapstruct</groupId>
                            <artifactId>mapstruct-processor</artifactId>
                            <version>${org.mapstruct.version}</version>
                        </path>
                    </annotationProcessorPaths>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.22.2</version>
                <configuration>
                    <useSystemClassLoader>false</useSystemClassLoader>
                    <skipTests>true</skipTests>
                </configuration>
            </plugin>
        </plugins>
    </build>
```