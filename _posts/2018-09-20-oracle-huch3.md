---
layout:     post
title:      "OracleORA-01722: invalid number" 
subtitle:   ""
author:     "neal"
header-img: "img/post-bg-universe.jpg"
catalog: true
tags:
    - OracleORA-01722
---

## OracleORA-01722: invalid number---表连接问题 ##
- 昨天遇到了一个令人头疼的问题，定位了半小时才发现，在此记录一下。


### 报错的sql语句 ###

	select 
	  ext.org_channel_type  as  ORG_CHANNEL_TYPE
	from 
      channel.channel_pub_info     i
	left join 
      channel.sec_organize_ext ext
	on 
      i.channel_id = ext.organize_id
	left join 
      channel.cm_bs_static_data a
	on 
      a.code_value = ext.org_channel_type
	and 
      a.code_type = 'FIRST_ORG_TYPE'
	where 1 = 1
	AND 
      ORG_CHANNEL_TYPE='80002' 


### 原因分析 ###

	昨天下午执行这条语句一直报OracleORA-01722: invalid number，
	以为是ORG_CHANNEL_TYPE='80002'的问题，改成
	ORG_CHANNEL_TYPE=80002后还是报这个错。而改成
	ORG_CHANNEL_TYPE='80009'就不会报错，只是查不出数据，因为
	80009这个数据没有。所以猜测是数据库巡查数据的时候校验数据不同导致的。

### 尝试修改 ###

	channel.cm_bs_static_data的code_value 是varchar2类型，而
	channel.sec_organize_ext.org_channel_type是number类型
	，Oracle会从channel.cm_bs_static_data表中检索所有的
	code_value ，这样里面就有非数字的code_value ，Oracle在比较
	varchar与number时，会采用to_number(code_value )
	=ext.org_channel_type,所以会报OracleORA-01722: invalid 
	number的错误。改成a.code_value = CAST
	(ext.org_channel_type AS varchar2(20))后，对
	ext.org_channel_type进行先转varchar再比较就可以了。

### 疑问 ###

	问题虽然发现并解决了，但是有人就会问了，那改成
	ORG_CHANNEL_TYPE='80009'就不会报错？是为什么？

	因为80009在channel.sec_organize_ext中就不存在，所以不会去
	执行left join channel.cm_bs_static_data a
	on a.code_value = ext.org_channel_type
	and a.code_type = 'FIRST_ORG_TYPE'，也就不会检索
	code_value ，所以不会报错。
