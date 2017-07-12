---
title: Set集合使用注意tips
date: 2017-02-22 19:11:02
tags:
---

### Set中的某些元素，当时使用边遍历，边删除的方法，报了以下异常：

ConcurrentModificationException


	example:
		Set<CheckWork> set = this.getUserDao().getAll(qf).get(0).getActionCheckWorks();
		for(CheckWork checkWork : set){
		    if(checkWork.getState()==1){
		        set.remove(checkWork);
		    }
		}
		
### 解决方案：
1. 遍历删除List

		List<CheckWork> list = this.getUserDao().getAll();
		Iterator<CheckWork> chk_it = list.iterator();
		while(chk_it.hasNext()){
		    CheckWork checkWork = chk_it.next();
		    if(checkWork.getPlanState()==1){
		        chk_it.remove();
		    }
		}
	
2. 遍历删除Set

		Set<CheckWork> set =  this.getUserDao().getAll().get(0).getActionCheckWorks();
				Iterator<CheckWork> it = set.iterator();
				while(it.hasNext()){
					CheckWork checkWork = it.next();
					if(checkWork.getState()==1){
						it.remove();
					}
				}
3. 遍历删除Set

		Set<CheckWork> set = this.getUserDao().getAll(qf).get(0).getActionCheckWorks()；
		Set<CheckWork> delSet=new HashSet<>();
		for(CheckWork checkWork : set){
		    if(checkWork.getState()==1){
		        delSet.add(checkWork);
		    }
		}
		set.removeAll(delSet);	
