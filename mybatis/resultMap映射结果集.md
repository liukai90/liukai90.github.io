**当我们去写多表关联查询的sql，通常会映射出一个非常复杂的结果，如果我们使用Mybatis框架，那么就得使用resultMap去编写映射，这让我们非常头疼，经过我的总结得出了比较简单的步骤。**

**拿一个真实项目开发中的案例：**

###1.先去编写映射数据库查询结果集的java bean如下

```java
package com.xdja.atecs.pojo;

public class ContactEntry {

    private Integer contactType;

    private String  contactValue;

    public Integer getContactType() {

        return contactType;

    }

    public void setContactType(Integer contactType) {

        this.contactType = contactType;

    }

    public String getContactValue() {

        return contactValue;

    }

    public void setContactValue(String contactValue) {

        this.contactValue = contactValue;

    }

}

```



看到这个不要觉得太简单，下面才是我们真正要映射的结果集：

```java
package com.xdja.atecs.pojo;

import com.xdja.atecs.util.Utils;

import java.util.List;

public class MemberExtend {

    private String userId;

    private List<Integer> deptIds;

    private String userName;

    private Integer operateStatus;

    private long operateSerial;

    private List<Integer> identitysIds;

    private List<ContactEntry> contactEntries;

    public String getUserId() {

        return userId;

    }

    public void setUserId(String userId) {

        this.userId = userId;

    }

    public List<Integer> getDeptIds() {

        return deptIds;

    }

    public void setDeptIds(List<Integer> deptIds) {

        this.deptIds = deptIds;

    }

    public String getUserName() {

        return userName;

    }

    public void setUserName(String userName) {

        this.userName = userName;

    }

    public Integer getOperateStatus() {

        return operateStatus;

    }

    public void setOperateStatus(Integer operateStatus) {

        this.operateStatus = operateStatus;

    }

    public long getOperateSerial() {

        return operateSerial;

    }

    public void setOperateSerial(long operateSerial) {

        this.operateSerial = operateSerial;

    }

    public List<Integer> getIdentitysIds() {

        return identitysIds;

    }

    public void setIdentitysIds(List<Integer> identitysIds) {

        this.identitysIds = identitysIds;

    }

    public List<ContactEntry> getContactEntries() {

        return contactEntries;

    }

    public void setContactEntries(List<ContactEntry> contactEntries) {

        this.contactEntries = contactEntries;

    }

}

```



###2.下面说明resultMap一看就懂

```xml-dtd
<resultMap id="memberExtend" type="com.xdja.atecs.pojo.MemberExtend">

    <!-- column对应数据库的字段，property对应java bean的变量名称-->

    <result column="userId" property="userId"/><!--对应MemberExtend 的userId变量-->

    <result column="userName" property="userName"/><!--对应MemberExtend 的userName变量-->

    <result column="operateStatus" property="operateStatus"/><!--对应MemberExtend 的operateStatus变量-->

    <result column="operateSerial" property="operateSerial"/><!--对应MemberExtend 的operateSerial变量-->

<!--对应MemberExtend 的deptIds变量,collection标签相当于集合，ofType相当于集合泛型的具体类型，表示集合里每个元素的类型-->

    <collection property="deptIds" ofType="java.lang.Integer">

        <result property="deptId" column="deptId"/>

    </collection>

<!--对应MemberExtend 的identitysIds变量-->

    <collection property="identitysIds" ofType="java.lang.Integer">

        <result property="identitysId" column="identitysId"/>

    </collection>

<!--对应MemberExtend 的contactEntries变量，一个java bean类型集合，还是相同的道理-->

    <collection property="contactEntries" ofType="com.xdja.atecs.pojo.ContactEntry">

            <result property="contactType" column="contactType"/>

            <result property="contactValue" column="contactValue"/>

    </collection>

    

</resultMap>

```



**虽然说得很清楚,但还可能有疑问如果MemberExtend结果集包装的不是List而是单个的java bean该如何映射。和collection标签大同小异，使用association标签,用法几乎相同。**