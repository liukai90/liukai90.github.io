# 前言
在做安阳无纸化会议系统经常会涉及到一些增删改查，例如插入操作，插入一张表，也会涉及到其他关联表的数据插入，在插入其他关联表的时候要使用到前一张表的主键id，前一张表的主键id是数据库自动生成，由于多张表的插入操作需要保证事物一致性，则在整个过程操作完之前，是不能查询到前表的id，这个问题是需要解决的一个问题。
还有批量插入，批量查询，以及查询时需要处理的一些细节。

---

# 正文
## 1.参数回填
经常会有这样一种场景在A表里插入了一条数据，操作B表需要用到A表那条数据的id，又由于A表和B表的操作在同一个事物中，通过其他条件来查询A表的id是查询不到的。所以需要mybatis的参数回填属性，示例代码：

```xml-dtd
 <insert id="insertUserInfo" parameterType="com.xdja.conference.model.User" useGeneratedKeys="true" keyProperty="id" keyColumn="id">
    INSERT INTO t_user_info(name,phone,dept_id,position,group_addr,create_time,update_time,operator)
    VALUES(#{name},#{phone},#{deptId},#{position},#{groupAddr},NOW(),NOW(),#{operator})
  </insert>
```
***useGeneratedKeys***：true为启用设置是否使用JDBC的getGenereatedKeys方法获取主键并赋值到keyProperty设置的领域模型属性中。<br/>
***keyProperty***：javabean中的属性名称。<br/>
***keyColumn***：对应数据库的字段名称。<br/>
这条sql执行后传入参数对象user的id属性会拿到数据库的id值。

---

## 2.批量操作
批量操作包括批量增删改查，批量增删改逻辑是一样的，批量查询单独来说。
### 2.1批量写操作
批量插入操作应用场景，多张表同时插入多条数据，分条来插入效率低下且不能保证事物一致性，所有有了批量插入的应用场景，示例代码：

```xml-dtd
 <insert id="insertUserRoleBatch">
    INSERT INTO t_user_role(user_id,role_id)
    VALUES
      <foreach collection="userRoles" index="index" item="userRole" separator=",">
        (#{userRole.userId},#{userRole.roleId})
      </foreach>
  </insert>
```
批量操作必须使用到<foreach>标签类似于编程语言中的for，需要了解的是里面的几个属性:<br/>
***collection***:传入集合或数组的名称。<br/>
***index***:遍历数组或集合的索引一般就写index。<br/>
***item***:取出每一个元素的名称。<br/>
***separator***:上一个元素与下一个元素之间填充的字符，如不需要则不添加此属性。<br>

---
### 2.2批量读操作
批量读操作的应用场景根据多个id查询多条记录，单独查效率上跟不上，所以需要批量去查，原理则是mysql的IN()函数。
示例代码：

```xml-dtd
<select id="queryRoleList" resultType="com.xdja.conference.model.RoleInfo"
          parameterType="com.xdja.conference.model.RoleInfo">
    SELECT t.`id`,t.`name`,t.`code`,t.`description`,t.`operator`,
    t.`create_time`,t.`update_time`,t.clientName,t.clientType
    FROM (SELECT tri.`id`,tri.`name`,tri.`code`,tri.`description`,
    tri.`operator`,tri.`create_time`,tri.`update_time`,
    GROUP_CONCAT(tct.client_name SEPARATOR ',') clientName,
    GROUP_CONCAT(tct.client_type SEPARATOR ',') clientType
    FROM t_role_info tri
      INNER JOIN t_role_client trc ON tri.`id`=trc.`role_id`
      INNER JOIN t_client_type tct ON trc.`client_type`=tct.`client_type`
    <where>
>      <if test="clientTypes != null and clientTypes.size() > 0">
>         AND tct.`client_type` IN
>         <foreach collection="clientTypes" index="index" item="clientTypeInt"
>                  open="(" close=")" separator=",">
>           #{clientTypeInt}
>         </foreach>
>       </if>
      <if test="name != null and name != ''">
        AND locate(#{name,jdbcType=VARCHAR}, tri.name)
      </if>
      AND tri.operating_state != 3
    </where>
       GROUP BY tri.`id`) t
    <where>
      <if test="clientType != null and clientType != ''">
        t.clientType = #{clientType}
      </if>
    </where>
  </select>
```
上面所标记的部分就是批量查询的核心，与批量写操作基本相同：<br>
多余介绍foreach的两个属性：<br>
***open***：表示在每个元素前填充字符。<br>
***close***:表示在每个元素后填充字符。<br>

---
## 3.查询的一些细节
### 3.1 模糊查询
一般进行模糊查询使用***like***关键字，后面的参数使用'%'做前缀或者后缀但是在mybatis mapper.xml文件中写like这种模糊查询的语句会出现很多问题，很难去定位问题。所以建议使用如下的这种方式。代码示例：

```xml-dtd
<select id="queryDeptList" resultType="com.xdja.conference.model.DeptInfo" parameterType="com.xdja.conference.model.DeptInfo" >
    SELECT
    t.id,
    t.`name`,
    t.`level`,
    t.parentId,
    t.update_time AS updateTime,
    t.operator,
    t2.`name` as parentName
    FROM t_dept_info t
    LEFT JOIN t_dept_info t2 ON t.parentId=t2.id
    <where>
>       <if test="name != null and name != ''">
>         and locate(#{name,jdbcType=VARCHAR}, t.name)
>       </if>
      <if test="level != null">
        and t.`level` = #{level,jdbcType=TINYINT}
      </if>
      AND t.operating_state != 3
    </where>
  </select>
```
如上标记部分  locate(#{name,jdbcType=VARCHAR}, t.name) ：t.name数据库字段名，#{name,jdbcType=VARCHAR}传入的参数值。locate函数会帮我们实现模糊查询。

### 3.2 查询结果字符连接
查询的某个字段是多条数据时，但我们需要的是一条数据，这里就可以使用字符串连接函数。示例代码：


```xml-dtd
 SELECT t.`id`,t.`name`,t.`code`,t.`description`,t.`operator`,
    t.`create_time`,t.`update_time`,t.clientName,t.clientType
    FROM (SELECT tri.`id`,tri.`name`,tri.`code`,tri.`description`,
    tri.`operator`,tri.`create_time`,tri.`update_time`,
>     GROUP_CONCAT(tct.client_name SEPARATOR ',') clientName,
>     GROUP_CONCAT(tct.client_type SEPARATOR ',') clientType
    FROM t_role_info tri
      INNER JOIN t_role_client trc ON tri.`id`=trc.`role_id`
      INNER JOIN t_client_type tct ON trc.`client_type`=tct.`client_type`
    <where>
      <if test="clientTypes != null and clientTypes.size() > 0">
        AND tct.`client_type` IN
        <foreach collection="clientTypes" index="index" item="clientTypeInt"
                 open="(" close=")" separator=",">
          #{clientTypeInt}
        </foreach>
      </if>
      <if test="name != null and name != ''">
        AND locate(#{name,jdbcType=VARCHAR}, tri.name)
      </if>
      AND tri.isDel = 0
    </where>
       GROUP BY tri.`id`) t
    <where>
      <if test="clientType != null and clientType != ''">
        t.clientType = #{clientType}
      </if>
    </where>
    ORDER BY t.create_time DESC
```
***GROUP_CONCAT(tct.client_name SEPARATOR ',')*** :连接字符串函数。<br>
***tct.client_name*** :需要连接的字段名。<br>
***SEPARATOR ','*** :中间连接符。<br>

---

# 小结
- 表之间需要关联插入时可使用参数回填
- 批量操作的核心是foreach标签
- 慎用mysql模糊查询关键字**like**，建议使用locate()函数。
- 需要将多条数据拼成一条时使用GROUP_CONCAT(tct.client_type SEPARATOR ',')函数。