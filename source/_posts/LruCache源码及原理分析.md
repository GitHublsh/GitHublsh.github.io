---
title: LruCache源码及原理分析
date: 2018-04-19 17:02:25
tags: [Android进阶]
---

### 一、源码及解析

因为源码不多，就直接全部贴出来分析。源码及分析如下：

	/*
	 * Copyright (C) 2011 The Android Open Source Project
	 *
	 * Licensed under the Apache License, Version 2.0 (the "License");
	 * you may not use this file except in compliance with the License.
	 * You may obtain a copy of the License at
	 *
	 *      http://www.apache.org/licenses/LICENSE-2.0
	 *
	 * Unless required by applicable law or agreed to in writing, software
	 * distributed under the License is distributed on an "AS IS" BASIS,
	 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
	 * See the License for the specific language governing permissions and
	 * limitations under the License.
	 */
	
	package android.util;
	
	import java.util.LinkedHashMap;
	import java.util.Map;
	
	/**
	 * BEGIN LAYOUTLIB CHANGE
	 * This is a custom version that doesn't use the non standard LinkedHashMap#eldest.
	 * END LAYOUTLIB CHANGE
	 *
	 * A cache that holds strong references to a limited number of values. Each time
	 * a value is accessed, it is moved to the head of a queue. When a value is
	 * added to a full cache, the value at the end of that queue is evicted and may
	 * become eligible for garbage collection.
	 *
	 * <p>If your cached values hold resources that need to be explicitly released,
	 * override {@link #entryRemoved}.
	 *
	 * <p>If a cache miss should be computed on demand for the corresponding keys,
	 * override {@link #create}. This simplifies the calling code, allowing it to
	 * assume a value will always be returned, even when there's a cache miss.
	 *
	 * <p>By default, the cache size is measured in the number of entries. Override
	 * {@link #sizeOf} to size the cache in different units. For example, this cache
	 * is limited to 4MiB of bitmaps:
	 * <pre>   {@code
	 *   int cacheSize = 4 * 1024 * 1024; // 4MiB
	 *   LruCache<String, Bitmap> bitmapCache = new LruCache<String, Bitmap>(cacheSize) {
	 *       protected int sizeOf(String key, Bitmap value) {
	 *           return value.getByteCount();
	 *       }
	 *   }}</pre>
	 *
	 * <p>This class is thread-safe. Perform multiple cache operations atomically by
	 * synchronizing on the cache: <pre>   {@code
	 *   synchronized (cache) {
	 *     if (cache.get(key) == null) {
	 *         cache.put(key, value);
	 *     }
	 *   }}</pre>
	 *
	 * <p>This class does not allow null to be used as a key or value. A return
	 * value of null from {@link #get}, {@link #put} or {@link #remove} is
	 * unambiguous: the key was not in the cache.
	 *
	 * <p>This class appeared in Android 3.1 (Honeycomb MR1); it's available as part
	 * of <a href="http://developer.android.com/sdk/compatibility-library.html">Android's
	 * Support Package</a> for earlier releases.
	 */
	// note: ssh 
	//date: 20170418
	public class LruCache<K, V> {
	    private final LinkedHashMap<K, V> map;//存放数据的集合
	
	    /** Size of this cache in units. Not necessarily the number of elements. */
	    private int size;//当前LruCache的内存占用的大小
	    private int maxSize;//Lrucache的最大容量
	
	    private int putCount;//put的次数
	    private int createCount;//create的次数
	    private int evictionCount;//回收的次数
	    private int hitCount;//命中的次数
	    private int missCount;//丢失的次数
	
	    /**
	     * @param maxSize for caches that do not override {@link #sizeOf}, this is
	     *     the maximum number of entries in the cache. For all other caches,
	     *     this is the maximum sum of the sizes of the entries in this cache.
	     */
	    //构造函数，在这里设置maxsize,并且实例化一个LinkedHashMap.LinkedHashMap是LruCache的核心。
	    public LruCache(int maxSize) {
	        if (maxSize <= 0) {
	            throw new IllegalArgumentException("maxSize <= 0");
	        }
	        this.maxSize = maxSize;
	        //初始容量为0，加载因子是0.75f即当容量达到最大容量的0.75时会把内存增加一半，accessOrder 为true：访问顺序，false:插入顺序
	        this.map = new LinkedHashMap<K, V>(0, 0.75f, true);
	    }
	
	    /**
	     * Sets the size of the cache.
	     * @param maxSize The new maximum size.
	     *
	     * @hide
	     */
	    //重置最大容量，调用trimToSize来实现
	    public void resize(int maxSize) {
	        if (maxSize <= 0) {
	            throw new IllegalArgumentException("maxSize <= 0");
	        }
	
	        synchronized (this) {
	            this.maxSize = maxSize;
	        }
	        trimToSize(maxSize);
	    }
	
	    /**
	     * Returns the value for {@code key} if it exists in the cache or can be
	     * created by {@code #create}. If a value was returned, it is moved to the
	     * head of the queue. This returns null if a value is not cached and cannot
	     * be created.
	     */
	    //通过key获取元素值，如果缓存中存在的话
	    public final V get(K key) {
	        if (key == null) {
	            throw new NullPointerException("key == null");
	        }
	
	        V mapValue;
	        synchronized (this) {
	            mapValue = map.get(key);
	            if (mapValue != null) {
	                hitCount++;
	                return mapValue;
	            }
	            missCount++;
	        }
	
	        /*
	         * Attempt to create a value. This may take a long time, and the map
	         * may be different when create() returns. If a conflicting value was
	         * added to the map while create() was working, we leave that value in
	         * the map and release the created value.
	         */
	        //如果值不存在，那么就通过create（key）来创建一个，create(key)默认是返回null.如果需要自定义可重写这个方法
	        V createdValue = create(key);
	        if (createdValue == null) {
	            return null;
	        }
	        //如果重写了create(key)，返回并不为null,即创建了新的数据，那么就会将数据放进缓存中
	        synchronized (this) {
	            createCount++;
	            mapValue = map.put(key, createdValue);
	
	            if (mapValue != null) {
	                // There was a conflict so undo that last put
	                map.put(key, mapValue);
	            } else {
	                size += safeSizeOf(key, createdValue);
	            }
	        }
	
	        if (mapValue != null) {
	            entryRemoved(false, key, createdValue, mapValue);
	            return mapValue;
	        } else {
	            trimToSize(maxSize);
	            return createdValue;
	        }
	    }
	
	    /**
	     * Caches {@code value} for {@code key}. The value is moved to the head of
	     * the queue.
	     *
	     * @return the previous value mapped by {@code key}.
	     */
	    //向缓存中添加数据
	    public final V put(K key, V value) {
	        if (key == null || value == null) {
	            throw new NullPointerException("key == null || value == null");
	        }
	
	        V previous;
	        synchronized (this) {
	            putCount++;
	            size += safeSizeOf(key, value);//safeSizeOf(key, value)会调用SizeOf(key, value)，返回的值为1
	            previous = map.put(key, value);//Hashmap.put()
	            if (previous != null) {//如果不为null,即添加失败，需要在缓存中减去这个元素，重置大小
	                size -= safeSizeOf(key, previous);
	            }
	        }
	
	        if (previous != null) {
	            entryRemoved(false, key, previous, value);
	        }
	
	        trimToSize(maxSize);
	        return previous;
	    }
	
	    /**
	     * @param maxSize the maximum size of the cache before returning. May be -1
	     *     to evict even 0-sized elements.
	     */
	    private void trimToSize(int maxSize) {
	        while (true) {//开启死循环
	            K key;
	            V value;
	            synchronized (this) {
	                if (size < 0 || (map.isEmpty() && size != 0)) {
	                    throw new IllegalStateException(getClass().getName()
	                            + ".sizeOf() is reporting inconsistent results!");
	                }
	
	                if (size <= maxSize) {
	                    break;//当已用的缓存小于最大缓存时，推出循环
	                }
	
	                // BEGIN LAYOUTLIB CHANGE
	                // get the last item in the linked list.
	                // This is not efficient, the goal here is to minimize the changes
	                // compared to the platform version.
	                //否则就在缓存中找到最近最少使用的元素
	                Map.Entry<K, V> toEvict = null;
	                for (Map.Entry<K, V> entry : map.entrySet()) {
	                    toEvict = entry;
	                }
	                // END LAYOUTLIB CHANGE
	
	                if (toEvict == null) {
	                    break;
	                }
	
	                key = toEvict.getKey();
	                value = toEvict.getValue();
	                map.remove(key);//然后删掉找到的最近最少使用的元素
	                size -= safeSizeOf(key, value);//减少了已使用的缓存空间
	                evictionCount++;
	            }
	
	            entryRemoved(true, key, value, null);
	        }
	    }
	
	    /**
	     * Removes the entry for {@code key} if it exists.
	     *
	     * @return the previous value mapped by {@code key}.
	     */
	    //删除元素
	    //删去元素，缩减已使用缓存值
	    public final V remove(K key) {
	        if (key == null) {
	            throw new NullPointerException("key == null");
	        }
	
	        V previous;
	        synchronized (this) {
	            previous = map.remove(key);
	            if (previous != null) {
	                size -= safeSizeOf(key, previous);
	            }
	        }
	
	        if (previous != null) {
	            entryRemoved(false, key, previous, null);
	        }
	
	        return previous;
	    }
	
	    /**
	     * Called for entries that have been evicted or removed. This method is
	     * invoked when a value is evicted to make space, removed by a call to
	     * {@link #remove}, or replaced by a call to {@link #put}. The default
	     * implementation does nothing.
	     *
	     * <p>The method is called without synchronization: other threads may
	     * access the cache while this method is executing.
	     *
	     * @param evicted true if the entry is being removed to make space, false
	     *     if the removal was caused by a {@link #put} or {@link #remove}.
	     * @param newValue the new value for {@code key}, if it exists. If non-null,
	     *     this removal was caused by a {@link #put}. Otherwise it was caused by
	     *     an eviction or a {@link #remove}.
	     */
	    protected void entryRemoved(boolean evicted, K key, V oldValue, V newValue) {}
	
	    /**
	     * Called after a cache miss to compute a value for the corresponding key.
	     * Returns the computed value or null if no value can be computed. The
	     * default implementation returns null.
	     *
	     * <p>The method is called without synchronization: other threads may
	     * access the cache while this method is executing.
	     *
	     * <p>If a value for {@code key} exists in the cache when this method
	     * returns, the created value will be released with {@link #entryRemoved}
	     * and discarded. This can occur when multiple threads request the same key
	     * at the same time (causing multiple values to be created), or when one
	     * thread calls {@link #put} while another is creating a value for the same
	     * key.
	     */
	    protected V create(K key) {
	        return null;
	    }
	
	    private int safeSizeOf(K key, V value) {
	        int result = sizeOf(key, value);
	        if (result < 0) {
	            throw new IllegalStateException("Negative size: " + key + "=" + value);
	        }
	        return result;
	    }
	
	    /**
	     * Returns the size of the entry for {@code key} and {@code value} in
	     * user-defined units.  The default implementation returns 1 so that size
	     * is the number of entries and max size is the maximum number of entries.
	     *
	     * <p>An entry's size must not change while it is in the cache.
	     */
	    protected int sizeOf(K key, V value) {
	        return 1;
	    }
	
	    /**
	     * Clear the cache, calling {@link #entryRemoved} on each removed entry.
	     */
	    public final void evictAll() {
	        trimToSize(-1); // -1 will evict 0-sized elements
	    }
	
	    /**
	     * For caches that do not override {@link #sizeOf}, this returns the number
	     * of entries in the cache. For all other caches, this returns the sum of
	     * the sizes of the entries in this cache.
	     */
	    public synchronized final int size() {
	        return size;
	    }
	
	    /**
	     * For caches that do not override {@link #sizeOf}, this returns the maximum
	     * number of entries in the cache. For all other caches, this returns the
	     * maximum sum of the sizes of the entries in this cache.
	     */
	    public synchronized final int maxSize() {
	        return maxSize;
	    }
	
	    /**
	     * Returns the number of times {@link #get} returned a value that was
	     * already present in the cache.
	     */
	    public synchronized final int hitCount() {
	        return hitCount;
	    }
	
	    /**
	     * Returns the number of times {@link #get} returned null or required a new
	     * value to be created.
	     */
	    public synchronized final int missCount() {
	        return missCount;
	    }
	
	    /**
	     * Returns the number of times {@link #create(Object)} returned a value.
	     */
	    public synchronized final int createCount() {
	        return createCount;
	    }
	
	    /**
	     * Returns the number of times {@link #put} was called.
	     */
	    public synchronized final int putCount() {
	        return putCount;
	    }
	
	    /**
	     * Returns the number of values that have been evicted.
	     */
	    public synchronized final int evictionCount() {
	        return evictionCount;
	    }
	
	    /**
	     * Returns a copy of the current contents of the cache, ordered from least
	     * recently accessed to most recently accessed.
	     */
	    public synchronized final Map<K, V> snapshot() {
	        return new LinkedHashMap<K, V>(map);
	    }
	
	    @Override public synchronized final String toString() {
	        int accesses = hitCount + missCount;
	        int hitPercent = accesses != 0 ? (100 * hitCount / accesses) : 0;
	        return String.format("LruCache[maxSize=%d,hits=%d,misses=%d,hitRate=%d%%]",
	                maxSize, hitCount, missCount, hitPercent);
	    }
	}

以上就是全部源码和分析注释。


### 二、简单使用

举个例子，图片缓存：

	private static class BitmapLruCache extends LruCache<String, Bitmap> {
	        public BitmapLruCache() {
	            // 构造方法传入当前应用可用最大内存的八分之一
	            super((int) (Runtime.getRuntime().maxMemory() / 1024 / 8));
	        }
	
	        @Override
	        // 重写sizeOf方法，并计算返回每个Bitmap对象占用的内存，必须重写
	        protected int sizeOf(String key, Bitmap value) {
	            return value.getByteCount() / 1024;
	        }
	
	        @Override
	        // 当缓存被移除时调用，第一个参数是表明缓存移除的原因，true表示被LruCache移除，false表示被主动remove移除，可不重写
	        protected void entryRemoved(boolean evicted, String key, Bitmap oldValue, Bitmap
	                newValue) {
	            super.entryRemoved(evicted, key, oldValue, newValue);
	        }
	
	        @Override
	        // 当get方法获取不到缓存的时候调用，如果需要创建自定义默认缓存，可以在这里添加逻辑，可不重写。可在取不到缓存图片的时候自定义操作
	        protected Bitmap create(String key) {
	            return super.create(key);
	        }
	    }


* 1.初始化

		LruCache<String, Bitmap> mLruCache = new BitmapLruCache();
	
* 2.将图片写入缓存中

		mLruCache.put(key, bitmap);
		
* 3.从缓存中读取图片

		mLruCache.get(key, bitmap);
		
* 4.将图片从缓存中删除

		mLruCache.remove(key);
		
		
		
使用还是很简单的。


### 三、小结

LruCache实现的核心是LinkedHashMap，为什么会用LinkedHashMap?
因为LinkedHashMap是由数组+双向链表的数据结构来实现的。其中双向链表的结构可以实现访问顺序和插入顺序.那么利用这个特性就可以实现LRU了。

看一下LinkedHashMap的构造方法，

	/**
	     * Constructs a new {@code LinkedHashMap} instance with the specified
	     * capacity, load factor and a flag specifying the ordering behavior.
	     *
	     * @param initialCapacity
	     *            the initial capacity of this hash map.
	     * @param loadFactor
	     *            the initial load factor.
	     * @param accessOrder
	     *            {@code true} if the ordering should be done based on the last
	     *            access (from least-recently accessed to most-recently
	     *            accessed), and {@code false} if the ordering should be the
	     *            order in which the entries were inserted.
	     * @throws IllegalArgumentException
	     *             when the capacity is less than zero or the load factor is
	     *             less or equal to zero.
	     */
	    public LinkedHashMap(
	            int initialCapacity, float loadFactor, boolean accessOrder) {
	        super(initialCapacity, loadFactor);
	        init();
	        this.accessOrder = accessOrder;
	    }
	    
参数在LruCache中已经分析过了，再说一遍具体含义。
    
initialCapacity：这个哈希表的初始容量

loadFactor：加载因子

accessOrder：true,访问顺序；false,插入顺序。


其实很简单。
