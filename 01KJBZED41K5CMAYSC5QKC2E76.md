---
title: 20240818 - 对MySQL的mutex锁进行分析
confluence_page_id: 3146182
created_at: 2024-08-18T05:41:29+00:00
updated_at: 2024-08-18T05:41:29+00:00
---

# 背景

分析锁中signal_count的机制

# 新建锁

获取cell (锁的实体化), cell中会复用os的event. 所以需要重置event:

```
os_event::os_event() {
	signal_count = 1
}
 
获取一个空的cell, 复用cell中的event
sync_array_reserve_cell {
	cell->signal_count = os_event_reset(event)
}
``` 

重置event, 会返回event的当前的signal_count. (后面等待时会用到)

# 锁等待

event wait时, 需要传入signal_count. 如果event的signal_count与传入的不一样, 则不进行wait

signal_count机制, 为了防止 在复用event的时候, 在"前世的event"进行等待.

```
TTASEventMutex<GenericPolicy>::wait {
	sync_array_wait_event {
		os_event_wait_low(sync_cell_get_event(cell), cell->signal_count)

		sync_array_free_cell(arr, cell) {
			signal_count = 0
		}
	}
}
``` 

# 解锁

解锁时, 递增signal_count, 表示这个锁的"锁生"结束

```
rw_lock_x_unlock_func {
	os_event::set() {
		broadcast()
			{
				++signal_count
			}
	}
}
```
