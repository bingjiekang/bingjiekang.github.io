---
title: "Golang的Redis数据库方法"
layout: post
date: 2023-01-17
image: /assets/images/markdown.jpg
headerImage: false
tag:
- golang
category: 笔记
---

##	Golang的Redis数据库用法 


1. Golang安装Redis数据库依赖包

		go get -u github.com/go-redis/redis
		
2. Golang引入依赖包

		import "github.com/go-redis/redis"
		
3. Golang连接数据库

		// 根据redis配置初始化一个客户端
		client := redis.NewClient(&redis.Options{
			Addr:     "localhost:6379", // redis地址
			Password: "", // redis密码，没有则留空
			DB:       0,  // 默认数据库，默认是0
		})
		
4. 验证是否连接成功

		_, err := client.Ping().Result() // 心跳验证
		if err != nil {
			log.Fatal(err) // 打印错误
		}
		
5. Tip：go-redis包自带了连接池，会自动维护redis连接，因此创建一次client即可，不要查询一次redis就关闭client。

6. options参数展示

		type Options struct {
			// 网络类型 tcp 或者 unix.
			// 默认是 tcp.
			Network string
			
			// redis地址，格式 host:port
			Addr string
			​
			// 新建一个redis连接的时候，会回调这个函数
			OnConnect func(*Conn) error
			​
			// redis密码，redis server没有设置可以为空。
			Password string
			    
			// redis数据库，序号从0开始，默认是0，可以不用设置
			DB int
			​
			// redis操作失败最大重试次数，默认不重试。
			MaxRetries int
			    
			// 最小重试时间间隔.
			// 默认是 8ms ; -1 表示关闭.
			MinRetryBackoff time.Duration
			    
			// 最大重试时间间隔
			// 默认是 512ms; -1 表示关闭.
			MaxRetryBackoff time.Duration
			​
			// redis连接超时时间.
			// 默认是 5 秒.
			DialTimeout time.Duration
			    
			// socket读取超时时间
			// 默认 3 秒.
			ReadTimeout time.Duration
			    
			// socket写超时时间
			WriteTimeout time.Duration
			​
			// redis连接池的最大连接数.
			// 默认连接池大小等于 cpu个数 * 10
			PoolSize int
			    
			// redis连接池最小空闲连接数.
			MinIdleConns int
			
			// redis连接最大的存活时间，默认不会关闭过时的连接.
			MaxConnAge time.Duration
			    
			// 当你从redis连接池获取一个连接之后，连接池最多等待这个拿出去的连接多长时间。
			// 默认是等待 ReadTimeout + 1 秒.
			PoolTimeout time.Duration
			
			// redis连接池多久会关闭一个空闲连接.
			// 默认是 5 分钟. -1 则表示关闭这个配置项
			IdleTimeout time.Duration
			
			// 多长时间检测一下，空闲连接
			// 默认是 1 分钟. -1 表示关闭空闲连接检测
			IdleCheckFrequency time.Duration
			
			// 只读设置，如果设置为true， redis只能查询缓存不能更新。
			readOnly bool
		}
		
7. Golang的Redis基本操作

		golang redis常用函数列表：
		Set - 设置一个key的值
		Get - 查询key的值
		GetSet - 设置一个key的值，并返回这个key的旧值
		SetNX - 如果key不存在，则设置这个key的值
		MGet - 批量查询key的值
		MSet - 批量设置key的值
		Incr,IncrBy,IncrByFloat - 针对一个key的数值进行递增操作
		Decr,DecrBy - 针对一个key的数值进行递减操作
		Del - 删除key操作，可以批量删除
		Expire - 设置key的过期时间
		
8. Golang基本键值对操作
		
		// 1、Set-设置值，若存在则直接更新
		// 第三个参数代表key的过期时间，0代表不会过期。
		// Second 是秒 180*Second意思是 3分钟后消息过期
		err := client.Set("key", "value", 180*time.Second).Err()
		if err != nil {
		    panic(err)
		}
		
		// 2、Get-查询值
		// Result函数返回两个值，第一个是key的值，第二个是错误信息
		val, err := client.Get("key").Result()
		// 判断查询是否出错
		if err != nil {
			panic(err)
		}
		fmt.Println("key", val)
		
		// 3、GetSet-设置新值，取出旧值
		// Result函数返回两个值，第一个是key的值，第二个是错误信息
		oldVal, err := client.GetSet("key", "new value").Result()
		if err != nil {
			panic(err)
		}
		// 打印key的旧值
		fmt.Println("key", oldVal)
		
		// 4、SetNX-当key不存在时，设置这个key的值，若存在则不设置
		// 第三个参数代表key的过期时间，0代表不会过期。
		err := client.SetNX("key", "value", 0).Err()
		if err != nil {
		    panic(err)
		}
		
		// 5、MGet-批量查询key的值
		// MGet函数可以传入任意个key，一次性返回多个值。
		// 这里Result返回两个值，第一个值是一个数组，第二个值是错误信息
		vals, err := client.MGet("key1", "key2", "key3").Result()
		if err != nil {
		    panic(err)
		}
		fmt.Println(vals)
		
		// 6、MSet-批量设置Key的值
		err := client.MSet("key1", "value1", "key2", "value2", "key3", "value3").Err()
		if err != nil {
		  panic(err)
		}
		
		// 7、Incr,IncrBy-针对一个key的数值进行递增操作
		// Incr函数每次加一
		val, err := client.Incr("key").Result()
		if err != nil {
		    panic(err)
		}
		fmt.Println("最新值", val)
		// IncrBy函数，可以指定每次递增多少
		val, err := client.IncrBy("key", 2).Result()
		if err != nil {
		    panic(err)
		}
		fmt.Println("最新值", val)
		// IncrByFloat函数，可以指定每次递增多少，跟IncrBy的区别是累加的是浮点数
		val, err := client.IncrByFloat("key", 2).Result()
		if err != nil {
		    panic(err)
		}
		fmt.Println("最新值", val)
		
		// 8、Decr,DecrBy-针对一个key的数值进行递减操作
		// Decr函数每次减一
		val, err := client.Decr("key").Result()
		if err != nil {
		    panic(err)
		}
		fmt.Println("最新值", val)
		// DecrBy函数，可以指定每次递减多少
		val, err := client.DecrBy("key", 2).Result()
		if err != nil {
		    panic(err)
		}
		fmt.Println("最新值", val)
		
		// 9、Del-删除key操作，支持批量删除
		// 删除key
		client.Del("key")
		// 删除多个key, Del函数支持删除多个key
		err := client.Del("key1", "key2", "key3").Err()
		if err != nil {
		    panic(err)
		}
		
		// 10、Expire-设置key的过期时间,单位秒
		client.Expire("key", 3)
		
9. Golang Redis hash教程

		redis hash操作主要有2-3个元素组成：
			key - redis key 唯一标识
			field - hash数据的字段名
			value - 值，有些操作不需要值
			
10. Go Redis hash数据常用函数

		HSet - 根据key和field字段设置，field字段的值
		HGet - 根据key和field字段，查询field字段的值
		HGetAll - 根据key查询所有字段和值
		HIncrBy - 根据key和field字段，累加数值。
		HKeys - 根据key返回所有字段名
		HLen - 根据key，查询hash的字段数量
		HMGet - 根据key和多个字段名，批量查询多个hash字段值
		HMSet - 根据key和多个字段名和字段值，批量设置hash字段值
		HSetNX - 如果field字段不存在，则设置hash字段值
		HDel - 根据key和字段名，删除hash字段，支持批量删除hash字段
		HExists - 检测hash字段名是否存在。
		
11. Go Redis hash函数用法

		// 1、HSet-根据key和field字段（keys）设置，field字段的值（value）
		// user_1 是hash key，username 是字段名, tizi365是字段值
		err := client.HSet("user_1", "username", "tizi365").Err()
		if err != nil {
		    panic(err)
		}

		// 2、HGet-根据key和field字段，查询field字段的值
		// user_1 是hash key，username是字段名
		val, err := client.HGet("user_1", "username").Result()
		if err != nil {
		    panic(err)
		}
		fmt.Println(val)
		
		// 3、HGetAll-根据key查询所有字段和值
		// 一次性返回key=user_1的所有hash字段和值
		data, err := client.HGetAll("user_1").Result()
		if err != nil {
		    panic(err)
		}
		// data是一个map类型，这里使用使用循环迭代输出
		for field, val := range data {
		    fmt.Println(field,val)
		}

		// 4、HIncrBy-根据key和field字段，累加字段的数值
		// 累加count字段的值，一次性累加2， user_1为hash key
		count, err := client.HIncrBy("user_1", "count", 2).Result()
		if err != nil {
		    panic(err)
		}
		fmt.Println(count)

		// 5、HKeys-根据key返回所有字段名
		// keys是一个string数组
		keys, err := client.HKeys("user_1").Result()
		if err != nil {
		    panic(err)
		}
		fmt.Println(keys)

		// 6、HLen-根据key，查询hash的字段数量
		// hashkey 是这个hashkey是哈希表的命名
		size, err := client.HLen("hashkey").Result()
		if err != nil {
		    panic(err)
		}
		fmt.Println(size)
		
		// 7、HMGet-根据key和多个字段名，批量查询多个hash字段值
		// HMGet支持多个field字段名，意思是一次返回多个字段值
		vals, err := client.HMGet("key","field1", "field2").Result()
		if err != nil {
		    panic(err)
		}
		// vals是一个数组
		fmt.Println(vals)
		
		// 8、HMSet-根据key和多个字段名和字段值，批量设置hash字段值
		// 初始化hash数据的多个字段值
		data := make(map[string]interface{})
		data["id"] = 1
		data["username"] = "tizi"
		// 一次性保存多个hash字段值
		err := client.HMSet("key", data).Err()
		if err != nil {
		    panic(err)
		}
		
		// 9、HSetNX-当field字段不存在时，设置hashkey里的字段的值
		err := client.HSetNX("key", "id", 100).Err()
		if err != nil {
		    panic(err)
		}
		
		// 10、HDel-根据key和字段名，删除hash字段，支持批量删除hash字段
		// 删除一个字段id
		client.HDel("key", "username")
		// 删除多个字段
		client.HDel("key", "username1", "username2")
		
		// 11、HExists-检测hash字段名是否存在。
		// 检测id字段是否存在
		err := client.HExists("key", "username").Err()
		if err != nil {
		    panic(err)
		}
		
12. Golang Redis 列表(list)用法:Redis列表是简单的字符串列表，列表是有序的，列表中的元素可以重复。可以添加一个元素到列表的头部（左边）或者尾部（右边）

13. Golang Redis list数据操作常用函数

		LPush - 从列表左边插入数据
		LPushX - 跟LPush的区别是，仅当列表存在的时候才插入数据
		RPop - 从列表的右边删除第一个数据，并返回删除的数据
		RPush - 从列表右边插入数据
		RPushX - 跟RPush的区别是，仅当列表存在的时候才插入数据
		LPop - 从列表左边删除第一个数据，并返回删除的数据
		LLen - 返回列表的大小
		LRange - 返回列表的一个范围内的数据，也可以返回全部数据
		LRem - 删除列表中的数据
		LIndex - 根据索引坐标，查询列表中的数据
		LInsert - 在指定位置插入数据
		
14. Golang Redis list函数用法

		// 1、LPush-从列表左边插入数据
		// 插入一个数据
		client.LPush("key", "data1")
		// LPush支持一次插入任意个数据
		err := client.LPush("key", data1,data2,data3,data4,data5).Err()
		if err != nil {
		    panic(err)
		}
		
		// 2、LPushX-跟LPush的区别是，仅当列表存在的时候才插入数据，用法完全一样。
		// 插入一个数据
		client.LPushX("key", "data1")
		// LPushX支持一次插入任意个数据
		err := client.LPushX("key", data1,data2,data3,data4,data5).Err()
		if err != nil {
		    panic(err)
		}
		
		// 3、RPop-从列表的右边删除第一个数据，并返回删除的数据
		val, err := client.RPop("key").Result()
		if err != nil {
		    panic(err)
		}
		fmt.Println(val)
		
		// 4、RPush-从列表右边插入数据
		// 插入一个数据
		client.RPush("key", "data1")
		// 支持一次插入任意个数据
		err := client.RPush("key", data1,data2,data3,data4,data5).Err()
		if err != nil {
		    panic(err)
		}
		
		// 5、RPushX-跟RPush的区别是，仅当列表存在的时候才插入数据, 他们用法一样
		// 插入一个数据
		client.RPushX("key", "data1")
		// 支持一次插入任意个数据
		err := client.RPushX("key", data1,data2,data3,data4,data5).Err()
		if err != nil {
		    panic(err)
		}
		
		// 6、LPop-从列表左边删除第一个数据，并返回删除的数据
		val, err := client.LPop("key").Result()
		if err != nil {
		    panic(err)
		}
		fmt.Println(val)
		
		// 7、LLen-返回列表的大小
		val, err := client.LLen("key").Result()
		if err != nil {
		    panic(err)
		}
		fmt.Println(val)
		
		// 8、LRange-返回列表的一个范围内的数据，也可以返回全部数据
		// 返回从0开始到-1位置之间的数据，意思就是返回全部数据
		vals, err := client.LRange("key",0,-1).Result()
		if err != nil {
		    panic(err)
		}
		fmt.Println(vals)
		
		// 9、LRem-删除列表中的数据
		// 从列表左边开始，删除100， 如果出现重复元素，仅删除1次，也就是删除第一个
		dels, err := client.LRem("key",1,100).Result()
		if err != nil {
		    panic(err)
		}
		// 如果存在多个100，则从列表左边开始删除2个100
		client.LRem("key",2,100)
		// 如果存在多个100，则从列表右边开始删除2个100
		// 第二个参数负数表示从右边开始删除几个等于100的元素
		client.LRem("key",-2,100)
		// 如果存在多个100，第二个参数为0，表示删除所有元素等于100的数据
		client.LRem("key",0,100)
		
		// 10、LIndex-根据索引坐标，查询列表中的数据
		// 列表索引从0开始计算，这里返回第6个元素
		val, err := client.LIndex("key",5).Result()
		if err != nil {
		    panic(err)
		}
		fmt.Println(val)
		
		// 11、LInsert-在指定位置插入数据
		// 在列表中5的前面插入4
		// before是之前的意思
		err := client.LInsert("key","before", 5, 4).Err()
		if err != nil {
		    panic(err)
		}
		// 在列表中 tizi365 元素的前面插入 欢迎你
		client.LInsert("key","before", "tizi365", "欢迎你")
		// 在列表中 tizi365 元素的后面插入 2019
		client.LInsert("key","after", "tizi365", "2019")
		

15. Golang Redis 集合(set)：redis的set类型（集合）是string类型数值的无序集合，并且集合元素唯一。

16. Golang Redis 集合常用函数：

		SAdd - 添加集合元素
		SCard - 获取集合元素个数
		SIsMember - 判断元素是否在集合中
		SMembers - 获取集合中所有的元素
		SRem - 删除集合元素
		SPop,SPopN - 随机返回集合中的元素，并且删除返回的元素
		
17. Golang Redis集合方法

		// 1、SAdd-添加集合元素
		// 添加100到集合中
		err := client.SAdd("key",100).Err()
		if err != nil {
		    panic(err)
		}
		// 将100,200,300添加到集合中
		client.SAdd("key",100, 200, 300)
		
		// 2、SCard-获取集合元素个数
		size, err := client.SCard("key").Result()
		if err != nil {
		    panic(err)
		}
		fmt.Println(size)
		
		// 3、SIsMember-判断元素是否在集合中
		// 检测100是否包含在集合中
		ok, _ := client.SIsMember("key", 100).Result()
		if ok {
		    fmt.Println("集合包含指定元素")
		}
		
		// 4、SMembers-获取集合中所有的元素
		es, _ := client.SMembers("key").Result()
		// 返回的es是string数组
		fmt.Println(es)
		
		// 5、SRem-删除集合元素
		// 删除集合中的元素100
		client.SRem("key", 100)
		// 删除集合中的元素tizi和2019
		client.SRem("key", "tizi", "2019")
		
		// 6、SPop,SPopN-随机返回集合中的元素，并且删除返回的元素
		// 随机返回集合中的一个元素，并且删除这个元素
		val, _ := client.SPop("key").Result()
		fmt.Println(val)
		// 随机返回集合中的5个元素，并且删除这些元素
		vals, _ := client.SPopN("key", 5).Result()
		fmt.Println(vals)
		
18. Golang Redis 有序集合(sorted set)：Redis 有序集合（sorted set）和集合一样也是string类型元素的集合，且不允许重复的成员，不同的是每个元素都会关联一个double类型的分数，这个分数主要用于集合元素排序。

19. Golang Redis 有序集合常用函数：

		ZAdd - 添加一个或者多个元素到集合，如果元素已经存在则更新分数
		ZCard - 返回集合元素个数
		ZCount - 统计某个分数范围内的元素个数
		ZIncrBy - 增加元素的分数
		ZRange,ZRevRange - 返回集合中某个索引范围的元素，根据分数从小到大排序
		ZRangeByScore,ZRevRangeByScore - 根据分数范围返回集合元素，元素根据分数从小到大排序，支持分页。
		ZRem - 删除集合元素
		ZRemRangeByRank - 根据索引范围删除元素
		ZRemRangeByScore - 根据分数范围删除元素
		ZScore - 查询元素对应的分数
		ZRank, ZRevRank - 查询元素的排名
		
20. Golang Redis有序集合函数用法

		// 1、ZAdd-添加一个或者多个元素到集合，如果元素已经存在则更新分数
		// 添加一个集合元素到集合中， 这个元素的分数是2.5，元素名是tizi
		err := client.ZAdd("key", redis.Z{2.5,"tizi"}).Err()
		if err != nil {
		    panic(err)
		}
		下面是redis.Z结构体说明：
		type Z struct {
		    Score  float64 // 分数
		    Member interface{} // 元素名
		}
		
		// 2、ZCard-返回集合元素个数
		size, err := client.ZCard("key").Result()
		if err != nil {
		    panic(err)
		}
		fmt.Println(size)
		
		// 3、ZCount-统计某个分数范围内的元素个数
		// 返回： 1<=分数<=5 的元素个数, 注意："1", "5"两个参数是字符串
		size, err := client.ZCount("key", "1","5").Result()
		if err != nil {
		    panic(err)
		}
		fmt.Println(size)
		// 返回： 1<分数<5 的元素个数
		// 说明：默认第二，第三个参数是大于等于和小于等于的关系。
		// 如果加上（ 则表示大于或者小于，相当于去掉了等于关系。
		size, err := client.ZCount("key", "(1","(5").Result()
		
		// 4、ZIncrBy-增加元素的分数
		// 给元素5，加上2分
		client.ZIncrBy("key", 2,"5")
		
		// 5、ZRange,ZRevRange-返回集合中某个索引范围的元素，根据分数从小到大排序
		// 返回从0到-1位置的集合元素， 元素按分数从小到大排序
		// 0到-1代表则返回全部数据
		vals, err := client.ZRange("key", 0,-1).Result()
		if err != nil {
		    panic(err)
		}
		for _, val := range vals {
		    fmt.Println(val)
		}
		ZRevRange用法跟ZRange一样，区别是ZRevRange的结果是按分数从大到小排序。
		
		// 6、ZRangeByScore-根据分数范围返回集合元素，元素根据分数从小到大排序，支持分页。
		// 初始化查询条件， Offset和Count用于分页
		op := redis.ZRangeBy{
		    Min:"2", // 最小分数
		    Max:"10", // 最大分数
		    Offset:0, // 类似sql的limit, 表示开始偏移量
		    Count:5, // 一次返回多少数据
		}
		vals, err := client.ZRangeByScore("key", op).Result()
		if err != nil {
		    panic(err)
		}
		for _, val := range vals {
		    fmt.Println(val)
		}
		
		// 7、ZRevRangeByScore-用法类似ZRangeByScore，区别是元素根据分数从大到小排序。
		
		// 8、ZRangeByScoreWithScores-用法跟ZRangeByScore一样，区别是除了返回集合元素，同时也返回元素对应的分数
		// 初始化查询条件， Offset和Count用于分页
		op := redis.ZRangeBy{
		    Min:"2", // 最小分数
		    Max:"10", // 最大分数
		    Offset:0, // 类似sql的limit, 表示开始偏移量
		    Count:5, // 一次返回多少数据
		}
		vals, err := client.ZRangeByScoreWithScores("key", op).Result()
		if err != nil {
		    panic(err)
		}
		for _, val := range vals {
		    fmt.Println(val.Member) // 集合元素
		    fmt.Println(val.Score) // 分数
		}
		
		// 9、ZRem-删除集合元素
		// 删除集合中的元素tizi
		client.ZRem("key", "tizi")
		// 删除集合中的元素tizi和xiaoli
		// 支持一次删除多个元素
		client.ZRem("key", "tizi", "xiaoli")
		
		// 10、ZRemRangeByRank-根据索引范围删除元素
		// 集合元素按分数排序，从最低分到高分，删除第0个元素到第5个元素。
		// 这里相当于删除最低分的几个元素
		client.ZRemRangeByRank("key", 0, 5)
		// 位置参数写成负数，代表从高分开始删除。
		// 这个例子，删除最高分数的两个元素，-1代表最高分数的位置，-2第二高分，以此类推。
		client.ZRemRangeByRank("key", -1, -2)
		
		// 11、ZRemRangeByScore-根据分数范围删除元素
		// 删除范围： 2<=分数<=5 的元素
		client.ZRemRangeByScore("key", "2", "5")
		// 删除范围： 2<=分数<5 的元素，加上（代表去掉等于
		client.ZRemRangeByScore("key", "2", "(5")
		
		// 12、ZScore-查询元素对应的分数
		// 查询集合元素tizi的分数
		score, _ := client.ZScore("key", "tizi").Result()
		fmt.Println(score)
		
		// 13、ZRank-根据元素名，查询集合元素在集合中的排名，从0开始算，集合元素按分数从小到大排序
		rk, _ := client.ZRank("key", "tizi").Result()
		fmt.Println(rk)
		ZRevRank的作用跟ZRank一样，区别是ZRevRank是按分数从大到小排序。
		

21. Golang Redis发布订阅：

			Redis提供了发布订阅功能，可以用于消息的传输，Redis的发布订阅机制包括三个部分，发布者，订阅者和Channel。
			发布订阅架构图：发布者和订阅者都是Redis客户端，Channel则为Redis服务器端，发布者将消息发送到某个的频道，订阅了这个频道的订阅者就能接收到这条消息。
			Redis的这种发布订阅机制与基于主题的发布订阅类似，Channel相当于主题。
			
22. Go Redis发布订阅常用函数：

		Subscribe - 订阅channel
		PSubscribe - 订阅channel支持通配符匹配
		Publish - 将信息发送到指定的channel。
		PubSubChannels - 查询活跃的channel
		PubSubNumSub - 查询指定的channel有多少个订阅者
		
23. Golang Redis发布订阅函数用法：

		// 1、Subscribe-订阅channel
		
		例子1：
		// 订阅channel1这个channel
		sub := client.Subscribe("channel1")
		// 读取channel消息
		iface, err := sub.Receive()
		if err != nil {
		    // handle error
		}
		// 检测收到的消息类型
		switch iface.(type) {
			case *redis.Subscription:
			    // 订阅成功
			case *redis.Message:
			    // 处理收到的消息
			    // 这里需要做一下类型转换
			    m := iface.(redis.Message)
			    // 打印收到的小
			    fmt.Println(m.Payload)
			case *redis.Pong:
			    // 收到Pong消息
			default:
			    // handle error
		}
		
		例子2： 使用golang channel的方式处理消息
		// 订阅channel1这个channel
		sub := client.Subscribe("channel1")
		// sub.Channel() 返回go channel，可以循环读取redis服务器发过来的消息
		for msg := range sub.Channel() {
		    // 打印收到的消息
		    fmt.Println(msg.Channel)
		    fmt.Println(msg.Payload)
		}
		
		例子3： 取消订阅
		// 订阅channel1这个channel
		sub := client.Subscribe("channel1")
		// 忽略其他处理逻辑
		// 取消订阅
		sub.Unsubscribe("channel1")
		
		// 2、PSubscribe-用法跟Subscribe一样，区别是PSubscribe订阅通道(channel)支持模式匹配。
		例子：
		// 订阅channel1这个channel
		sub := client.PSubscribe("ch_user_*")
		// 可以匹配ch_user_开头的任意channel
		
		// 3、Publish-将消息发送到指定的channel
		// 将"message"消息发送到channel1这个通道上
		client.Publish("channel1","message")
		
		// 4、PubSubChannels-查询活跃的channel
		// 没有指定查询channel的匹配模式，则返回所有的channel
		chs, _ := client.PubSubChannels("").Result()
		for _, ch := range chs {
		    fmt.Println(ch)
		}
		// 匹配user_开头的channel
		chs, _ := client.PubSubChannels("user_*").Result()
		
		// 5、PubSubNumSub-查询指定的channel有多少个订阅者
		// 查询channel1，channel2两个通道的订阅者数量
		chs, _ := client.PubSubNumSub("channel1", "channel2").Result()
		for ch, count := range chs {
		    fmt.Println(ch) // channel名字
		    fmt.Println(count) // channel的订阅者数量
		}
		
24. Golang Redis事务

		redis事务可以一次执行多个命令， 并且带有以下两个重要的保证：
			事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。
			事务是一个原子操作：事务中的命令要么全部被执行，要么全部都不执行。
			
25. Go Redis事务常用函数：

		TxPipeline - 以Pipeline的方式操作事务
		Watch - redis乐观锁支持
		
26. Golang Redis事务函数用法

		// 1、TxPipeline-以Pipeline的方式操作事务
		// 开启一个TxPipeline事务
		pipe := client.TxPipeline()
		​
		// 执行事务操作，可以通过pipe读写redis
		incr := pipe.Incr("tx_pipeline_counter")
		pipe.Expire("tx_pipeline_counter", time.Hour)
		​
		// 上面代码等同于执行下面redis命令
		//
		//     MULTI
		//     INCR pipeline_counter
		//     EXPIRE pipeline_counts 3600
		//     EXEC
		​
		// 通过Exec函数提交redis事务
		_, err := pipe.Exec()
		​
		// 提交事务后，我们可以查询事务操作的结果
		// 前面执行Incr函数，在没有执行exec函数之前，实际上还没开始运行。
		fmt.Println(incr.Val(), err)
		
		// 2、watch-redis乐观锁支持，可以通过watch监听一些Key, 如果这些key的值没有被其他人改变的话，才可以提交事务。

		// 定义一个回调函数，用于处理事务逻辑
		fn := func(tx *redis.Tx) error {
			// 先查询下当前watch监听的key的值
			v, err := tx.Get("key").Result()
			if err != nil && err != redis.Nil {
			    return err
			}
		
			// 这里可以处理业务
			fmt.Println(v)
			
			// 如果key的值没有改变的话，Pipelined函数才会调用成功
			_, err = tx.Pipelined(func(pipe redis.Pipeliner) error {
			    // 在这里给key设置最新值
			    pipe.Set("key", "new value", 0)
			    return nil
			})
			return err
		}
		
		// 使用Watch监听一些Key, 同时绑定一个回调函数fn, 监听Key后的逻辑写在fn这个回调函数里面
		// 如果想监听多个key，可以这么写：client.Watch(fn, "key1", "key2", "key3")
		client.Watch(fn, "key")
		

	


				
	




		






	