# MyBatis

MyBatis最初是Apache的一个开源项目**iBatis**, 2010年6月这个项目由Apache Software Foundation迁移到了Google Code。随着开发团队转投Google Code旗下， iBatis3.x正式更名为MyBatis。代码于2013年11月迁移到Github。

iBatis一词来源于"internet"和"abatis"的组合，是一个基于Java的持久层框架。 iBatis提供的持久层框架包括`SQL Maps`和`Data Access Objects`(DAO)。

![./images](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303010949392.png)

- [MyBatis介绍](Basis/mybatis_introduce.md)

> MyBatis介绍：MyBatis历史、MyBatis下载地址、MyBatis特性、MyBatis优缺点、和其他持久化层技术对比

- [MyBatis构建入门程序](Basis/basic_program.md)

> MyBatis构建入门程序：创建数据、创建项目、搭建框架开发环境、测试代码、添加日志

- [MyBatis核心配置和映射器](Basis/core_mapping.md)

> 核心配置：configuration标签(配置properties标签、配置settings标签、配置typeAliases标签、配置typeHandlers标签、配置objectFactory标签、配置plugins标签、配置environments标签、配置mappers标签)
>
> 映射器：XML实现映射器、注解实现映射器、映射文件的命名规则

- [MyBatis基本操作](Basis/basic_operation.md)

> MyBatis基本操作：执行SQL的两种方式(SqlSession发送SQL、Mapper接口发送 SQL)、获取参数的两种方式、插入标签、更新标签、删除标签、查询标签、resultType和resultMap

- [MyBatis处理特殊操作](Advanced/special_operation.md)

> MyBatis处理特殊操作：模糊查询、批量删除、动态设置表名、获取自增的主键、处理字段和属性的映射关系、各类查询功能、批量插入大量数据

- [MyBatis注解](Advanced/mybatis_annotation.md)

> MyBatis注解：SQL语句映射、结果集映射、关系映射

- [MyBatis关联查询及延迟加载](Advanced/conjunctive_query.md)

> 关联查询(一对一关系、一对多关系、多对多关系)、延迟加载
>

- [MyBatis动态SQL](Advanced/dynamic_sql.md)

> MyBatis动态SQL：if标签、where标签、set标签、trim标签、choose/when/otherwise标签、foreach标签、bind标签、sql标签、script标签、多数据库支持、动态SQL中的插入脚本语言

- [MyBatis缓存](Advanced/mybatis_cache.md)

> MyBatis缓存：缓存机制介绍、缓存的基本原理、一级缓存和二级缓存、自定义缓存、整合EHCache

- [MyBatis的逆向工程](Advanced/reverse_engineering.md)

> MyBatis的逆向工程：相关概念、操作步骤(MyBatis3Simple、MyBatis3)

- [MyBatis分页操作](Advanced/mybatis_paging.md)

> MyBatis分页操作：手动实现分页操作、使用第三方实现分页操作

**拓展**：

- [ORM对象关系映射](../../../Other/Concept/orm.md)
- [MyBatis内置类型别名](Expand/default_alias.md)
- [IDEA配置模板](Expand/idea_template.md)
- [封装SqlSessionUtils工具类](Expand/sqlsessionutils.md)
- [MyBatis源码解析](Expand/source_principle.md)

