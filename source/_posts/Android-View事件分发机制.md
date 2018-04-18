---
title: Android View事件分发机制
date: 2017-10-11 17:27:08
tags: [View事件分发机制]
---

### 一、点击事件的传递规则

   首先，要明白点击事件的分发就是MotionEvent事件的分发过程。
   
#### 1.MotionEvent
   
  常见的动作：
  
  常见的动作常量：
  
  * public static final int ACTION_DOWN = 0;单点触摸动作
    
  * public static final int ACTION_UP = 1;单点触摸离开动作
  * public static final int ACTION_MOVE = 2;触摸点移动动作
  * public static final int ACTION_CANCEL = 3;触摸动作取消
  * public static final int ACTION_OUTSIDE = 4;触摸动作超出边界
  * public static final int ACTION_POINTER_DOWN = 5;多点触摸动作
  * public static final int ACTION_POINTER_UP       = 6;多点离开动作


	主要就是：
	
	* ACTION_DOWN--手指刚接触屏幕
	* ACTION_MOVE--手指在屏幕上滑动
	* ACTION_UP--手指离开屏幕

		
#### 2.点击事件的分发中最重要的三个方法

* dispatchTouchEvent(MotionEvent event)--事件分发

	* 用来进行事件分发，如果事件能够传递给当前View,那么这个方法一定会被调用，返回结果受当前View的onTouchEvent和下级View的dispatchTouchEvent方法影响，表示是否消耗了当前事件。
		
* onInterceptTouchEvent(MotionEvent event)--事件拦截

	* 用来判断是否拦截某个事件，如果当前View拦截了某个事件，那么在同一个事件序列中，此方法不会再次调用，返回结果表示是否拦截当前事件。
		
* onTouchEvent(MotionEvent event)--事件消费

	* 在dispatchTouchEvent方法中调用，用来处理点击事件，返回结果表示是否消耗当前事件，如果不消耗，则在同一事件序列中，当前View无法再次接收到事件

#### 3. 伪代码

		public boolean dispatchTouchEvent(MotionEvent ev){
			boolean touch = false;
			if(onInterceptTouchEvent(ev)){
				touch = onTouchEvent(ev);
			}else{
				touch = child.dispatchTouchEvent(ev);
			}
			return touch;
		}
     
   
* 点击事件的传递规则：

  * 对于根ViewGroup来说，点击事件产生后，首先传递给它，这个时候它的dispatchTouchEvent就会被调用	 
  
  * 首先判断自身是否处理，处理的话就会调用onInterceptTouchEvent(MotionEvent event),来判断是否处理，自身处理的话，就会调用onTouchEvent(MotionEvent event)，自己就把这次点击事件处理了
 
  * 如果自身不处理，就不拦截，交给子View去处理。子View处理同样按照ViewGroup自身处理一致。子View不处理的话，最后则调用ViewGroup自身的onTouchEvent(MotionEvent event)消费掉此次点击事件

    经典的伪代码如上。
      
      
#### 4.同一次点击事件只能被一个 View 消费

  安卓为了保证所有的事件都是被一个 View 消费的，对第一次的事件( ACTION_DOWN )进行了特殊判断，View 只有消费了 ACTION_DOWN 事件，才能接收到后续的事件(可点击控件会默认消费所有事件)，并且会将后续所有事件传递过来，不会再传递给其他 View，除非上层 View 进行了拦截。
如果上层 View 拦截了当前正在处理的事件，会收到一个 ACTION_CANCEL，表示当前事件已经结束，后续事件不会再传递过来。


借用GcsSloop源码分析：

	
	public boolean dispatchTouchEvent(MotionEvent ev) {
	  	// 调试用
	    if (mInputEventConsistencyVerifier != null) {
	        mInputEventConsistencyVerifier.onTouchEvent(ev, 1);
	    }
	
	  	// 判断事件是否是针对可访问的焦点视图(很晚才添加的内容，个人猜测和屏幕辅助相关，方便盲人等使用设备)
	    if (ev.isTargetAccessibilityFocus() && isAccessibilityFocusedViewOrHost()) {
	        ev.setTargetAccessibilityFocus(false);
	    }
	
	    boolean handled = false;
	    if (onFilterTouchEventForSecurity(ev)) {
	        final int action = ev.getAction();
	        final int actionMasked = action & MotionEvent.ACTION_MASK;
	
	        // 处理第一次ACTION_DOWN.
	        if (actionMasked == MotionEvent.ACTION_DOWN) {
	            // 清除之前所有的状态
	            cancelAndClearTouchTargets(ev);
	            resetTouchState();
	        }
	
	        // 检查是否需要拦截.
	        final boolean intercepted;
	        if (actionMasked == MotionEvent.ACTION_DOWN || mFirstTouchTarget != null) {
	            final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
	            if (!disallowIntercept) {
	                intercepted = onInterceptTouchEvent(ev);	// 询问是否拦截
	                ev.setAction(action); 						// 恢复操作，防止被更改
	            } else {
	                intercepted = false;
	            }
	        } else {
	          	// 没有目标来处理该事件，而且也不是一个新的事件事件(ACTION_DOWN), 进行拦截。
	            intercepted = true;
	        }
	
	      	// 判断事件是否是针对可访问的焦点视图
	        if (intercepted || mFirstTouchTarget != null) {
	            ev.setTargetAccessibilityFocus(false);
	        }
	
	        // 检查事件是否被取消(ACTION_CANCEL).
	        final boolean canceled = resetCancelNextUpFlag(this)
	                || actionMasked == MotionEvent.ACTION_CANCEL;
	
	        final boolean split = (mGroupFlags & FLAG_SPLIT_MOTION_EVENTS) != 0;
	        TouchTarget newTouchTarget = null;
	        boolean alreadyDispatchedToNewTouchTarget = false;
	      	
	      	// 如果没有取消也没有被拦截	(进入事件分发)
	        if (!canceled && !intercepted) {
	
	            // 如果事件是针对可访问性焦点视图，我们将其提供给具有可访问性焦点的视图。
	          	// 如果它不处理它，我们清除该标志并像往常一样将事件分派给所有的 ChildView。 
	            // 我们检测并避免保持这种状态，因为这些事非常罕见。
	            View childWithAccessibilityFocus = ev.isTargetAccessibilityFocus()
	                    ? findChildWithAccessibilityFocus() : null;
	
	            if (actionMasked == MotionEvent.ACTION_DOWN
	                    || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
	                    || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
	                final int actionIndex = ev.getActionIndex();
	                final int idBitsToAssign = split ? 1 << ev.getPointerId(actionIndex)
	                        : TouchTarget.ALL_POINTER_IDS;
	
	                // 清除此指针ID的早期触摸目标，防止不同步。
	                removePointersFromTouchTargets(idBitsToAssign);
	
	                final int childrenCount = mChildrenCount;
	                if (newTouchTarget == null && childrenCount != 0) {
	                    final float x = ev.getX(actionIndex);	// 获取触摸位置坐标
	                    final float y = ev.getY(actionIndex);
	                    // 查找可以接受事件的 ChildView
	                    final ArrayList<View> preorderedList = buildOrderedChildList();
	                    final boolean customOrder = preorderedList == null
	                            && isChildrenDrawingOrderEnabled();
	                    final View[] children = mChildren;
	                  	// ▼注意，从最后向前扫描
	                    for (int i = childrenCount - 1; i >= 0; i--) {
	                        final int childIndex = customOrder
	                                ? getChildDrawingOrder(childrenCount, i) : i;
	                        final View child = (preorderedList == null)
	                                ? children[childIndex] : preorderedList.get(childIndex);
	
	                        // 如果有一个视图具有可访问性焦点，我们希望它首先获取事件，
	                      	// 如果不处理，我们将执行正常的分派。 
	                      	// 尽管这可能会分发两次，但它能保证在给定的时间内更安全的执行。
	                        if (childWithAccessibilityFocus != null) {
	                            if (childWithAccessibilityFocus != child) {
	                                continue;
	                            }
	                            childWithAccessibilityFocus = null;
	                            i = childrenCount - 1;
	                        }
	
	                      	// 检查View是否允许接受事件(即处于显示状态(VISIBLE)或者正在播放动画)
	                      	// 检查触摸位置是否在View区域内
	                        if (!canViewReceivePointerEvents(child)
	                                || !isTransformedTouchPointInView(x, y, child, null)) {
	                            ev.setTargetAccessibilityFocus(false);
	                            continue;
	                        }
	
	                      	// getTouchTarget 中判断了 child 是否包含在 mFirstTouchTarget 中
	                      	// 如果有返回 target，如果没有返回 null 
	                        newTouchTarget = getTouchTarget(child);
	                        if (newTouchTarget != null) {
	                            // ChildView 已经准备好接受在其区域内的事件。
	                            newTouchTarget.pointerIdBits |= idBitsToAssign;
	                            break;	// ◀︎已经找到目标View，跳出循环
	                        }
	
	                        resetCancelNextUpFlag(child);
	                        if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
	                            mLastTouchDownTime = ev.getDownTime();
	                            if (preorderedList != null) {
	                                for (int j = 0; j < childrenCount; j++) {
	                                    if (children[childIndex] == mChildren[j]) {
	                                        mLastTouchDownIndex = j;
	                                        break;
	                                    }
	                                }
	                            } else {
	                                mLastTouchDownIndex = childIndex;
	                            }
	                            mLastTouchDownX = ev.getX();
	                            mLastTouchDownY = ev.getY();
	                            newTouchTarget = addTouchTarget(child, idBitsToAssign);
	                            alreadyDispatchedToNewTouchTarget = true;
	                            break;
	                        }
	                      
	                        ev.setTargetAccessibilityFocus(false);
	                    }
	                    if (preorderedList != null) preorderedList.clear();
	                }
	
	                if (newTouchTarget == null && mFirstTouchTarget != null) {
	                    // 没有找到 ChildView 接收事件
	                    newTouchTarget = mFirstTouchTarget;
	                    while (newTouchTarget.next != null) {
	                        newTouchTarget = newTouchTarget.next;
	                    }
	                    newTouchTarget.pointerIdBits |= idBitsToAssign;
	                }
	            }
	        }
	
	        // 分发 TouchTarget
	        if (mFirstTouchTarget == null) {
	            // 没有 TouchTarget，将当前 ViewGroup 当作普通的 View 处理。
	            handled = dispatchTransformedTouchEvent(ev, canceled, null,
	                    TouchTarget.ALL_POINTER_IDS);
	        } else {
	            // 分发TouchTarget，如果我们已经分发过，则避免分配给新的目标。 
	          	// 如有必要，取消分发。
	            TouchTarget predecessor = null;
	            TouchTarget target = mFirstTouchTarget;
	            while (target != null) {
	                final TouchTarget next = target.next;
	                if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
	                    handled = true;
	                } else {
	                    final boolean cancelChild = resetCancelNextUpFlag(target.child)
	                            || intercepted;
	                    if (dispatchTransformedTouchEvent(ev, cancelChild,
	                            target.child, target.pointerIdBits)) {
	                        handled = true;
	                    }
	                    if (cancelChild) {
	                        if (predecessor == null) {
	                            mFirstTouchTarget = next;
	                        } else {
	                            predecessor.next = next;
	                        }
	                        target.recycle();
	                        target = next;
	                        continue;
	                    }
	                }
	                predecessor = target;
	                target = next;
	            }
	        }
	
	        // 如果需要，更新指针的触摸目标列表或取消。
	        if (canceled
	                || actionMasked == MotionEvent.ACTION_UP
	                || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
	            resetTouchState();
	        } else if (split && actionMasked == MotionEvent.ACTION_POINTER_UP) {
	            final int actionIndex = ev.getActionIndex();
	            final int idBitsToRemove = 1 << ev.getPointerId(actionIndex);
	            removePointersFromTouchTargets(idBitsToRemove);
	        }
	    }
	
	    if (!handled && mInputEventConsistencyVerifier != null) {
	        mInputEventConsistencyVerifier.onUnhandledEvent(ev, 1);
	    }
	    return handled;
	}
	
	
源码注释已经很详细了。





### 二、小结

#### 1. 同一个事件是从DOWN开始到UP结束的
#### 2. 一个事件最终只能由一个View拦截和消费掉
#### 3. 某个View一旦拦截了，那么久都由它来处理，并且它的onInterceptTouchEvent不会再被调用。因为事件交给要拦截的View处理，就不需要再调用onInterceptTouchEvent去询问是否要拦截事件了
#### 4. 如果一个View拦截了事件，但是如果它不消耗ACTION_DOWN事件，那么同一事件序列就不再会交由该View处理了。
#### 5. ViewGroup默认是不拦截任何事件的，源码中ViewGroup的onInterceptTouchEvent默认返回false
#### 6. 事件是否被消费由返回值决定，true 表示消费，false 表示不消费，与是否使用了事件无关。
#### 7. View没有onInterceptTouchEvent方法，一旦事件分发到View，那么它的onTouchEven就会被调用
#### 8. View的onTouchEvent方法默认会消耗事件即返回true，除非是不可点击的。
#### 9. 事件的传递过程是由外向内的，即先传给父元素，然后再由父元素分发给子View
#### 10. 如果当前正在处理的事件被上层 View 拦截，会收到一个 ACTION_CANCEL，后续事件不会再传递过来。