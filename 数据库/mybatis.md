- [基础操作](#基础操作)

- [小进阶](#小进阶)

# 基础操作
## 批量插入数据


批量插入数据总共有三种方式：一种是循环整个sql，一种是利用mysql的case when进行批量更新，还有一种是on duplicate key update进行批量更新

* 1.循环整个sql
```
<update id="updateBatch" parameterType="java.util.List" >
        <foreach collection="list" item="item" index="index" open="" close="" separator=";">
            update standard_relation
            <set >
                <if test="item.standardFromUuid != null" >
                    standard_from_uuid = #{item.standardFromUuid,jdbcType=VARCHAR},
                </if>
                <if test="item.standardToUuid != null" >
                    standard_to_uuid = #{item.standardToUuid,jdbcType=VARCHAR},
                </if>
                <if test="item.gmtModified != null" >
                    gmt_modified = #{item.gmtModified,jdbcType=TIMESTAMP},
                </if>
            </set>
            where id = #{item.id,jdbcType=BIGINT}
        </foreach>
    </update>
```
注意这种方法执行的时候如果传入的是个空的数组 会卡在这里（陷入死循环？）


* 2.case when的方式更新
```
<update id="updateBatch" parameterType="java.util.List" >
        update standard_relation
        <trim prefix="set" suffixOverrides=",">
            <trim prefix="standard_from_uuid =case" suffix="end,">
                <foreach collection="list" item="i" index="index">
                    <if test="i.standardFromUuid!=null">
                        when id=#{i.id} then #{i.standardFromUuid}
                    </if>
                </foreach>
            </trim>
            <trim prefix="standard_to_uuid =case" suffix="end,">
                <foreach collection="list" item="i" index="index">
                    <if test="i.standardToUuid!=null">
                        when id=#{i.id} then #{i.standardToUuid}
                    </if>
                </foreach>
            </trim>
            <trim prefix="gmt_modified =case" suffix="end,">
                <foreach collection="list" item="i" index="index">
                    <if test="i.gmtModified!=null">
                        when id=#{i.id} then #{i.gmtModified}
                    </if>
                </foreach>
            </trim>
        </trim>
        where
        <foreach collection="list" separator="or" item="i" index="index" >
            id=#{i.id}
        </foreach>
    </update>
```


* 3.on duplicate key update方式更新（不支持第三种方式，有可能会造成数据丢失和主从上表的自增id值不一致)
```
<insert id="updateBatch" parameterType="java.util.List">
        insert into standard_relation(id,relation_type, standard_from_uuid,
        standard_to_uuid, relation_score, stat,
        last_process_id, is_deleted, gmt_created,
        gmt_modified,relation_desc)VALUES
        <foreach collection="list" item="item" index="index" separator=",">
            (#{item.id,jdbcType=BIGINT},#{item.relationType,jdbcType=VARCHAR}, #{item.standardFromUuid,jdbcType=VARCHAR},
            #{item.standardToUuid,jdbcType=VARCHAR}, #{item.relationScore,jdbcType=DECIMAL}, #{item.stat,jdbcType=TINYINT},
            #{item.lastProcessId,jdbcType=BIGINT}, #{item.isDeleted,jdbcType=TINYINT}, #{item.gmtCreated,jdbcType=TIMESTAMP},
            #{item.gmtModified,jdbcType=TIMESTAMP},#{item.relationDesc,jdbcType=VARCHAR})
        </foreach>
        ON DUPLICATE KEY UPDATE
        id=VALUES(id),relation_type = VALUES(relation_type),standard_from_uuid = VALUES(standard_from_uuid),standard_to_uuid = VALUES(standard_to_uuid),
        relation_score = VALUES(relation_score),stat = VALUES(stat),last_process_id = VALUES(last_process_id),
        is_deleted = VALUES(is_deleted),gmt_created = VALUES(gmt_created),
        gmt_modified = VALUES(gmt_modified),relation_desc = VALUES(relation_desc)
    </insert>
```


# 小进阶
## 一个Spring项目如何同时使用多个数据源
在数据源的配置类上使用@MapperScan注解，里面声明使用该数据源的Mapper接口的包名和其中一种数据源的SqlSessionFactory，则创建Mapper接口的bean时，
会使用声明的SqlSessionFactory进行创建，此时就明确了使用的数据源。
具体可见：https://www.jianshu.com/p/735852145580
//TODO 熟悉mybatis的执行流程


























