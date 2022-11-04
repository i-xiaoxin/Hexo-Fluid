---
title: Redis缓存雪崩、缓存击穿、缓存穿透
date: 2022-11-3 20:29:02
tags: 
- Redis
- Mysql
index_img: 
banner_img: /img/article4.webp
categories:
comment: waline
---

# 分布式缓存Redis运用场景

### 1. 页面缓存

Redis可将Web页面的内容片段，包括HTML，CSS和图片等静态数据，缓存到Redis实例，提高网站的访问性能。

比如在电商类应用中，热销商品展示、秒杀推荐等数据面临高并发读的压力，分布式缓存Redis的高并发及灵活扩展，可轻松支持此类应用。

### 2. 状态缓存

Redis可将Session会话状态及应用横向扩展时的状态数据等缓存到DCS实例，实现状态数据共享。在应对游戏应用中爆发式增长的玩家数据存储和读写请求时，使用分布式缓存Redis可通过将热点数据放入缓存，加快用户端访问速度，提升用户体验。

### 3. 应用对象缓存

Redis可作为服务层的二级缓存对外提供服务，减轻数据库的负载压力，加速应用访问。

### 4. 事件缓存

Redis可提供针对事件流的连续查询（continuous query）处理技术，满足实时性需求。

# Redis缓存问题

在高并发的业务场景下，数据库大多数情况都是用户并发访问最薄弱的环节。所以，就需要使用redis做一个缓冲操作，让请求先访问到redis，而不是直接访问Mysql等数据库。这样可以大大缓解数据库的压力。缓存问题如下：

- 缓存穿透
- 缓存穿击
- 缓存雪崩
- 缓存污染
- 缓存同步（缓存和数据库一致性）

## 缓存穿透

### 问题描述

**缓存穿透**是指<font color=red>缓存和数据库中都没有数据</font>，而用户不断发起请求则这些<font color=red>请求会穿过缓存直接访问数据库</font>，如发起为id为“-1”的数据或id为特别大不存在的数据。假如有恶意攻击，就可以利用这个漏洞，对数据库造成压力，甚至压垮数据库。

<img src="https://img-blog.csdnimg.cn/img_convert/b7031182f770a7a5b3c82eaf749f53b0.png" style="zoom: 60%;" >

### 解决方案

应对缓存穿透的方案，常见的方案有三种。

- 第一种方案，非法请求的限制；
- 第二种方案，缓存空值或者默认值；
- 第三种方案，使用布隆过滤器快速判断数据是否存在，避免通过查询数据库来判断数据是否存在；

针对后端，此处采用方案二

缓存空对象：当存储层不命中时，<font color=red>创建空对象并将其缓存起来</font>，同时会<font color=red>设置一个过期时间</font>（避免控制占用更多的存储空间），之后再访问这个数据将会从缓存中获取，保护了后端数据源；

```java
/**
	 * 查询商品信息
	 * @param itemId
	 * @return
	 */
	@Override
	public TbItem selectItemInfo(Long itemId) {
		//查询缓存
		TbItem tbItem = (TbItem) redisClient.get(ITEM_INFO + ":" + itemId + ":"+ BASE);
		if(tbItem!=null){
			return tbItem;
		}

		tbItem = tbItemMapper.selectByPrimaryKey(itemId);
		/********************解决缓存穿透************************/
		if(tbItem == null){
			//把空对象保存到缓存
			redisClient.set(ITEM_INFO + ":" + itemId + ":"+ BASE,new TbItem());
			//设置缓存的有效期
			redisClient.expire(ITEM_INFO + ":" + itemId + ":"+ BASE,30);
			return tbItem;
		}
		//把数据保存到缓存
		redisClient.set(ITEM_INFO + ":" + itemId + ":"+ BASE,tbItem);
		//设置缓存的有效期
		redisClient.expire(ITEM_INFO + ":" + itemId + ":"+ BASE,ITEM_INFO_EXPIRE);
		return tbItem;
	}

	/**
	 * 根据商品 ID 查询商品介绍
	 * @param itemId
	 * @return
	 */
	@Override
	public TbItemDesc selectItemDescByItemId(Long itemId) {
		//查询缓存
		TbItemDesc tbItemDesc = (TbItemDesc) redisClient.get(
            	ITEM_INFO + ":" + itemId + ":"+ DESC);
		if(tbItemDesc!=null){
			return tbItemDesc;
		}

		TbItemDescExample example = new TbItemDescExample();
		TbItemDescExample.Criteria criteria = example.createCriteria();
		criteria.andItemIdEqualTo(itemId);
		List<TbItemDesc> itemDescList =
				this.tbItemDescMapper.selectByExampleWithBLOBs(example);
		if(itemDescList!=null && itemDescList.size()>0){
			//把数据保存到缓存
			redisClient.set(ITEM_INFO + ":" + itemId + ":"+ DESC,itemDescList.get(0));
			//设置缓存的有效期
			redisClient.expire(ITEM_INFO + ":" + itemId + ":"+ DESC,ITEM_INFO_EXPIRE);
			return itemDescList.get(0);
		}
		/********************解决缓存穿透************************/
		//把空对象保存到缓存
		redisClient.set(ITEM_INFO + ":" + itemId + ":"+ DESC,new TbItemDesc());
		//设置缓存的有效期
		redisClient.expire(ITEM_INFO + ":" + itemId + ":"+ DESC,30);
		return null;
	}
```

## 缓存击穿

### 问题描述

缓存击穿，是指一个key非常热点，在不停的扛着大并发，大并发集中<font color=red>对一个key不停进行访问，当这个key在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库</font>，就像在一个屏障上凿开了一个洞。

<img src="https://img-blog.csdnimg.cn/img_convert/acb5f4e7ef24a524a53c39eb016f63d4.png" style="zoom:60%;" >

### 解决方案

**分布式锁：**

- 设置热点数据永远不过期，由后台异步更新缓存，或者在热点数据准备要过期前，提前通知后台线程更新缓存以及重新设置过期时间

- 加分布式锁，保证同一时间只有一个业务线程更新缓存，未能获取互斥锁的请求，要么等待锁释放后重新读取缓存，要么就返回空值或者默认值。
  - 释放锁 del(lockKey)
  - 业务处理失败 expire(loclKey)

#### application.yml

```yaml
#分布式锁
SETNX_LOCK_BASC: SETNX_LOCK_BASC
SETNX_LOCK_DESC: SETNX_LOCK_DESC
SETNX_LOCK_PARAM: SETNX_LOCK_PARAM
```

#### service-ItemServiceImpl


```java
	/**
	 * 查询商品信息
	 * @param itemId
	 * @return
	 */
    @Override
    public TbItem selectItemInfo(Long itemId){
        //1、先查询redis,如果有直接返回
        TbItem tbItem = (TbItem) redisClient.get(ITEM_INFO+":"+itemId+":"+BASE);
        if(tbItem!=null){
            return tbItem;
        }
        /*****************解决缓存击穿***************/
        if(redisClient.setnx(SETNX_LOCK_BASC+":"+itemId,itemId,5L)){
            //2、再查询mysql,并把查询结果缓存到redis,并设置失效时间
            tbItem = tbItemMapper.selectByPrimaryKey(itemId);

            /*****************解决缓存穿透*****************/
            if(tbItem!=null){
                redisClient.set(ITEM_INFO+":"+itemId+":"+BASE,tbItem);
                redisClient.expire(ITEM_INFO+":"+itemId+":"+BASE,ITEM_INFO_EXPIRE);
            }else{
                redisClient.set(ITEM_INFO+":"+itemId+":"+BASE,new TbItem());
                redisClient.expire(ITEM_INFO+":"+itemId+":"+BASE,30L);
            }
            redisClient.del(SETNX_LOCK_BASC+":"+itemId);
            return tbItem;
        }else{
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
           return selectItemInfo(itemId);
        }
    }
	/**
	 * 根据商品 ID 查询商品介绍
	 * @param itemId
	 * @return
	 */
    @Override
    public TbItemDesc selectItemDescByItemId(Long itemId) {
        //1、先查询redis,如果有直接返回
        TbItemDesc tbItemDesc = (TbItemDesc) redisClient.get(ITEM_INFO + ":" + itemId + 
                                                             ":" + DESC);
        if(tbItemDesc!=null){
            return tbItemDesc;
        }
        if(redisClient.setnx(SETNX_DESC_LOCK_KEY+":"+itemId,itemId,30L)){
            //2、再查询mysql,并把查询结果缓存到redis,并设置失效时间
            tbItemDesc = tbItemDescMapper.selectByPrimaryKey(itemId);

            if(tbItemDesc!=null){
                redisClient.set(ITEM_INFO + ":" + itemId + ":" + DESC,tbItemDesc);
                redisClient.expire(ITEM_INFO + ":" + itemId + ":" + 
                                   DESC,ITEM_INFO_EXPIRE);

            }else{
                redisClient.set(ITEM_INFO + ":" + itemId + ":" + DESC,new TbItemDesc());
                redisClient.expire(ITEM_INFO + ":" + itemId + ":" + DESC,30L);
            }
            redisClient.del(SETNX_DESC_LOCK_KEY+":"+itemId);
            return tbItemDesc;
        }else{
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return selectItemDescByItemId(itemId);
        }
    }
```

## 缓存雪崩

### 问题描述

缓存雪崩是指缓存中**数据大批量到过期时间，而查询数据量巨大，引起数据库压力过大甚至宕机**。

<img src="https://img-blog.csdnimg.cn/img_convert/717343a0da7a1b05edab1d1cdf8f28e5.png">

### 解决方案

- 缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。

- 如果缓存数据库是分布式部署，将热点数据均匀分布在不同的缓存数据库中。

- 设置热点数据永远不过期。

# 参考文章

[Redis进阶 - 缓存问题：一致性, 穿击, 穿透, 雪崩, 污染等](https://pdai.tech/md/db/nosql-redis/db-redis-x-cache.html#%E6%95%B0%E6%8D%AE%E5%BA%93%E5%92%8C%E7%BC%93%E5%AD%98%E4%B8%80%E8%87%B4%E6%80%A7)

[什么是缓存雪崩、击穿、穿透？](https://xiaolincoding.com/redis/cluster/cache_problem.html#%E7%BC%93%E5%AD%98%E9%9B%AA%E5%B4%A9)

[图解Redis体系](https://xiaolincoding.com/redis/)

[布隆过滤器原理](https://github.com/daydreamdev/MeetingFilm/blob/master/note/%E5%B8%83%E9%9A%86%E8%BF%87%E6%BB%A4%E5%99%A8%E8%A7%A3%E5%86%B3%E7%BC%93%E5%AD%98%E7%A9%BF%E9%80%8F.md)

<div>
<hr>
<script src="https://unpkg.com/@waline/client@v2/dist/waline.js"></script> 
<link
  rel="stylesheet"
  href="https://unpkg.com/@waline/client@v2/dist/waline.css"
/>
<div id="waline"></div>
  <script>
    Waline({
      el: '#waline',
      serverURL: 'https://vercel-project-4d7haxk1c-i-xiaoxin.vercel.app',
    });
  </script>