---
title: 20210628 - Adaptive hash index 代码解析
confluence_page_id: 1147050
created_at: 2021-06-28T05:18:57+00:00
updated_at: 2021-06-28T05:19:44+00:00
---

# 基本信息

主文件: storage/innobase/btr/[btr0sea.cc](<http://btr0sea.cc>)

功能开关: btr_search_enabled

# 数据结构

  - btr_search_latches[btr_ahi_parts]: 保护全局结构的锁分区列表 (按不同索引进行分区, 一个索引落入一个分区)

  - btr_search_sys->hash_tables[btr_ahi_parts]: 全局分区列表 (按不同索引进行分区, 一个索引落入一个分区)

    - 元素是Hash表, Hash是 fold(检索条件的签名) → rec_t (一个地址, pointer to the data of the first hash table node in chain having the fold number)

  - buf_chunk_map_reg是个全局变量, map类型是 byte * → buf_chunk_t, 每个chunk有blocks[]. buf_chunk_map_reg是地址rec_t到block的二次映射, 用于从地址定位到页, 在初始化buffer pool时生成.

# 编译选项

  - BTR_CUR_ADAPT

  - BTR_CUR_HASH_ADAPT

# 监控项变量

  - btr_cur_n_sea

# 函数列表

可通过 监控项 或者 函数入口文件大致获得:

  - btr_search_sys_create : 初始化

  - btr_search_sys_resize: 调整大小

  - btr_search_enable: 启用

  - btr_search_disable: 禁用+清理

  - btr_search_guess_on_hash: 查找

  - btr_search_build_page_hash_index: 构建页的hash index

    - btr_search_info_update_slow

      - btr_search_info_update

        - btr_cur_search_to_nth_level

    - btr_search_move_or_delete_hash_entries

  - btr_search_update_block_hash_info

    - btr_search_info_update_slow

# 查找函数: btr_search_guess_on_hash
    
    
    //根据搜索条件, 生成hash值fold
    fold = dtuple_fold(…)
    
    btr_search_s_lock(index)
    {
      //在AHI (hash table)中按fold查找rec (一个地址)
      rec = ha_search_and_get_data( btr_get_search_table(index) , fold ) 
      
      //在buf_chunk_map_reg查找rec对应的block
      block = buf_block_from_ahi(rec)
      
      //记录block的访问, 对block上锁 (nowait模式)
      buf_page_get_known_nowait(..., block, ...)
    }
    btr_search_s_unlock(index)
    
    btr_cur_position(index, (rec_t*) rec, block, cursor)
    
    if (buf_page_peek_if_too_old(...))
    {
      buf_page_make_young(&block->page)
    }

# buf_block_from_ahi

输入是地址ptr

buf_chunk_map_reg是个全局变量, map类型是 byte * → buf_chunk_t, 是地址到chunk的映射

每个chunk有blocks[], 通过ptr的offset计算落到那个block (buf_block_t)

# 查找AHI的调用方: btr_cur_search_to_nth_level

主要判断查找AHI的入口条件:
    
    
    rw_lock_get_writer(btr_get_search_latch(index))
    		== RW_LOCK_NOT_LOCKED
    	    && latch_mode <= BTR_MODIFY_LEAF
    	    && info->last_hash_succ
    	    && !index->disable_ahi
    	    && !estimate
    # ifdef PAGE_CUR_LE_OR_EXTENDS
    	    && mode != PAGE_CUR_LE_OR_EXTENDS
    # endif /* PAGE_CUR_LE_OR_EXTENDS */
    	    && !dict_index_is_spatial(index)
    	    /* If !has_search_latch, we do a dirty read of
    	    btr_search_enabled below, and btr_search_guess_on_hash()
    	    will have to check it again. */
    	    && UNIV_LIKELY(btr_search_enabled)
    	    && !modify_external
    	    && btr_search_guess_on_hash(index, info, tuple, mode,
    					latch_mode, cursor,
    					has_search_latch, mtr)

条件:

  1. rw_lock_get_writer(btr_get_search_latch(index)) == RW_LOCK_NOT_LOCKED: AHI上的分区锁没有写锁

  2. latch_mode <= BTR_MODIFY_LEAF: BTR_SEARCH_LEAF/BTR_MODIFY_LEAF, 对btree的操作在叶子节点上

  3. info->last_hash_succ: ??

  4. !index->disable_ahi: AHI没被禁用

  5. !estimate: BTR_ESTIMATE: we do the search in query optimization

  6. mode != PAGE_CUR_LE_OR_EXTENDS: ??
         
         /*      PAGE_CUR_LE_OR_EXTENDS = 5,*/ /* This is a search mode used in
         				 "column LIKE 'abc%' ORDER BY column DESC";
         				 we have to find strings which are <= 'abc' or
         				 which extend it */

  7. !dict_index_is_spatial(index): 非GEO index (RTREE)

  8. btr_search_enabled:
         
         /** Is search system enabled.
         Search system is protected by array of latches. */
         bool		btr_search_enabled	= true;

  9. !modify_external:
         
         /** In the case of BTR_MODIFY_LEAF, the caller intends to allocate or
         free the pages of externally stored fields. */
         #define BTR_MODIFY_EXTERNAL	262144

## latch_mode参考表
    
    
    /** Latching modes for btr_cur_search_to_nth_level(). */
    enum btr_latch_mode {
    	/** Search a record on a leaf page and S-latch it. */
    	BTR_SEARCH_LEAF = RW_S_LATCH,
    	/** (Prepare to) modify a record on a leaf page and X-latch it. */
    	BTR_MODIFY_LEAF	= RW_X_LATCH,
    	/** Obtain no latches. */
    	BTR_NO_LATCHES = RW_NO_LATCH,
    	/** Start modifying the entire B-tree. */
    	BTR_MODIFY_TREE = 33,
    	/** Continue modifying the entire B-tree. */
    	BTR_CONT_MODIFY_TREE = 34,
    	/** Search the previous record. */
    	BTR_SEARCH_PREV = 35,
    	/** Modify the previous record. */
    	BTR_MODIFY_PREV = 36,
    	/** Start searching the entire B-tree. */
    	BTR_SEARCH_TREE = 37,
    	/** Continue searching the entire B-tree. */
    	BTR_CONT_SEARCH_TREE = 38
    };
    

## info->last_hash_succ

设置为TRUE的入口:

  - btr_search_update_block_hash_info

    - btr_search_info_update_slow

      - btr_search_info_update

        - btr_cur_search_to_nth_level
              
              //对于叶子节点, 进行AHI更新
              if (level == 0)
              {
                if (btr_search_enabled && !index->disable_ahi) {
                  btr_search_info_update(index, cursor);
                }
              }

  - btr_search_guess_on_hash

# btr_search_info_update: 更新AHI
    
    
    //对某个索引的AHI请求超过17次(BTR_SEARCH_HASH_ANALYSIS), 才进行AHI更新
    //info的计数保存在何处: 保存在索引对应的内存结构中
    
    info->hash_analysis++;
    if (info->hash_analysis < BTR_SEARCH_HASH_ANALYSIS) {
    	return;
    }
    btr_search_info_update_slow(...)

# btr_search_info_update_slow: 更新AHI
    
    
    //参数为 btr_search_t* info,	btr_cur_t* cursor
    
    block = btr_cur_get_block(cursor);
    btr_search_info_update_hash(info, cursor);
    
    build_index = btr_search_update_block_hash_info(info, block, cursor);
    if (build_index) {
            // 对page构建AHI
    		btr_search_build_page_hash_index(cursor->index, block,
    						 block->n_fields,
    						 block->n_bytes,
    						 block->left_side);
    }

# btr_search_build_page_hash_index: 对page构建AHI
    
    
    //参数包括 index/block/n_fields/n_bytes/left_side (后三项是索引的特性指标)
      //index: 索引
      //block: 索引页
      //n_fields/n_bytes/left_side: 索引的特性指标
    
    table = btr_get_search_table(index);
    page = buf_block_get_frame(block);
    
    rec = page_rec_get_next(page_get_infimum_rec(page))
    fold = rec_fold(rec, n_fields, n_bytes);
    if (left_side) {
    	folds[n_cached] = fold;
    	recs[n_cached] = rec;
    	n_cached++;
    }
    	
    for (;;) {
    		next_rec = page_rec_get_next(rec);
    		if (page_rec_is_supremum(next_rec)) { break; }
    		next_fold = rec_fold(next_rec, n_fields, n_bytes);
    		
    		if (fold != next_fold) {
    			/* Insert an entry into the hash index */
    			if (left_side) {
    				folds[n_cached] = next_fold;
    				recs[n_cached] = next_rec;
    				n_cached++;
    			} else {
    				folds[n_cached] = fold;
    				recs[n_cached] = rec;
    				n_cached++;
    			}
    		}
    
    		rec = next_rec;
    		fold = next_fold;
    }
    
    btr_search_x_lock(index);
    {
      for (i = 0; i < n_cached; i++) {
        //将fold->rec插入hash table, block为调试项
    	ha_insert_for_fold(table, folds[i], block, recs[i]);
      }
    }
    btr_search_x_unlock(index);

  - index/block/n_fields/n_bytes/left_side 用于识别AHI
        
        index
        	volatile ulint	n_bytes;	/*!< recommended prefix length for hash
        					search: number of bytes in
        					an incomplete last field */
        	volatile ulint	n_fields;	/*!< recommended prefix length for hash
        					search: number of full fields */
        					
        					//left_side, 是否记录匹配中最左的项??
        	volatile bool	left_side;	/*!< true or false, depending on
        					whether the leftmost record of several
        					records with the same prefix should be
        					indexed in the hash index */

## btr_search_info_update_hash: 是否推荐新的AHI

btr_search_info_update_hash 检查条件, 判断是否推荐新的AHI
    
    
    n_unique = dict_index_get_n_unique_in_tree(index);
    
    if 命中计数 == 0 {
      推荐新的AHI (set_new_recomm)
    }
    
    // if (推荐的AHI的字段数 >= 索引的唯一字段数, 即AHI可唯一定位到记录) && 
    //    (目标up叶子节点的匹配字段数 >= 索引的唯一字段数, 即 搜索条件足够精确)
    if (info->n_fields >= n_unique && cursor->up_match >= n_unique) {
      增加命中计数并退出 (increment_potential: info->n_hash_potential++; return)
    }
    
    
    // if (推荐的匹配字段长度 <= 目标low叶子节点的匹配字段长度 && left_side) ||
    //    (推荐的匹配字段长度 > 目标low叶子节点的匹配字段长度 && !left_side)
    // then 推荐新的AHI
    cmp = ut_pair_cmp(info->n_fields, info->n_bytes, cursor->low_match,
                      cursor->low_bytes);
    if (info->left_side ? cmp <= 0 : cmp > 0) {
      推荐新的AHI (set_new_recomm)
    }
    
    
    // if (推荐的匹配字段长度 <= 目标up叶子节点的匹配字段长度 && left_side) ||
    //    (推荐的匹配字段长度 > 目标up叶子节点的匹配字段长度 && !left_side)
    // then 推荐新的AHI
    cmp = ut_pair_cmp(info->n_fields, info->n_bytes, cursor->up_match,
                      cursor->up_bytes);
    if (info->left_side ? cmp <= 0 : cmp > 0) {
      增加命中计数并退出
    }
    
    推荐新的AHI:
    {
      if (cursor->up == cursor->low) {
        // ??
        AHI = (1,0,TRUE)
      }
      
      if (cursor->up > cursor->low) {
        if (cursor->up_match >= n_unique) {
          AHI = (n_unique, 0, TRUE)
        } else if (cursor->low_match < cursor->up_match) {
          //建立索引到low_match的下一字段, 就可区分出唯一性
          AHI = (cursor->low_match + 1, 0, TRUE)
        } else {
          //建立索引到low_bytes的下一字节, 就可区分出唯一性
          AHI = (cursor->low_match, cursor->low_bytes + 1, TRUE)
        }
      } else {
        if (cursor->low_match >= n_unique) {
          AHI = (n_unique, 0, FALSE)
        } else if (cursor->low_match > cursor->up_match) {
          AHI = (cursor->up_match + 1, 0, FALSE)
        } else {
          AHI = (cursor->up_match, cursor->up_bytes + 1, FALSE)
        }
      }
    }

对于up_match/low_match的解释:
    
    
    @param[in,out]	iup_matched_fields	already matched fields in the
    upper limit record
    @param[in,out]	iup_matched_bytes	already matched bytes in the
    first partially matched field in the upper limit record
    @param[in,out]	ilow_matched_fields	already matched fields in the
    lower limit record
    @param[in,out]	ilow_matched_bytes	already matched bytes in the
    first partially matched field in the lower limit record

计算up_match/low_match的入口:
    
    
    - btr_cur_search_to_nth_level
      - page_cur_search_with_match_bytes

# 性能相关

UNIV_SEARCH_PERF_STAT

btr_search_n_hash_fail

# TODO

  - n_hash_potential

  - buf_chunk_map_reg 的维护操作?

  - block->n_hash_helps 页上”猜测”的AHI, 命中 (与搜索条件匹配) 的次数

  - info->last_hash_succ 搜索条件 满足已有AHI

  -
