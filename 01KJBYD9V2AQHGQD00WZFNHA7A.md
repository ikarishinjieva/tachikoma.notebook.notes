---
title: 20210409 - iostat 与 sar -b 的区别
confluence_page_id: 753936
created_at: 2021-04-09T03:22:35+00:00
updated_at: 2021-04-09T07:53:10+00:00
---

# sar

sar的数据来源: <https://sourcegraph.com/github.com/sysstat/sysstat@d8a22f2c32f5705319ff64c83c6b4cdccd3fdc27/-/blob/rd_stats.c#L751:18>

数据获取: 

```
if (sscanf(line,
			   "%u %u %s "
			   "%lu %*u %lu %*u "
			   "%lu %*u %lu %*u "
			   "%*u %*u %*u "
			   "%lu %*u %lu",
			   &major, &minor, dev_name,
			   &rd_ios, &rd_sec,
			   &wr_ios, &wr_sec,
			   &dc_ios, &dc_sec) >= 7) {

			if (is_device(SLASH_SYS, dev_name, IGNORE_VIRTUAL_DEVICES)) {
				/*
				 * OK: It's a (real) device and not a partition.
				 * Note: Structure should have been initialized first!
				 */
				st_io->dk_drive      += (unsigned long long) rd_ios +
							(unsigned long long) wr_ios +
							(unsigned long long) dc_ios;
				st_io->dk_drive_rio  += rd_ios;
				st_io->dk_drive_rblk += rd_sec;
				st_io->dk_drive_wio  += wr_ios;
				st_io->dk_drive_wblk += wr_sec;
				st_io->dk_drive_dio  += dc_ios;
				st_io->dk_drive_dblk += dc_sec;
			}
		}
``` 

数据输出: 

```
	cprintf_f(NO_UNIT, 7, 9, 2,
		  sic->dk_drive < sip->dk_drive ? 0.0 :
		  S_VALUE(sip->dk_drive, sic->dk_drive, itv),
		  sic->dk_drive_rio < sip->dk_drive_rio ? 0.0 :
		  S_VALUE(sip->dk_drive_rio, sic->dk_drive_rio, itv),
		  sic->dk_drive_wio < sip->dk_drive_wio ? 0.0 :
		  S_VALUE(sip->dk_drive_wio, sic->dk_drive_wio, itv),
		  sic->dk_drive_dio < sip->dk_drive_dio ? 0.0 :
		  S_VALUE(sip->dk_drive_dio, sic->dk_drive_dio, itv),
		  sic->dk_drive_rblk < sip->dk_drive_rblk ? 0.0 :
		  S_VALUE(sip->dk_drive_rblk, sic->dk_drive_rblk, itv),
		  sic->dk_drive_wblk < sip->dk_drive_wblk ? 0.0 :
		  S_VALUE(sip->dk_drive_wblk, sic->dk_drive_wblk, itv),
		  sic->dk_drive_dblk < sip->dk_drive_dblk ? 0.0 :
		  S_VALUE(sip->dk_drive_dblk, sic->dk_drive_dblk, itv));
	printf("\n");
``` 

数值从/proc/diskstats直接读取和累加

# iostat

iostat的数据来源: <https://github.com/sysstat/sysstat/blob/master/iostat.c>, 函数 read_sysfs_file_stat, 文件: /sys/block/{dev}/stat

数据获取: 

```
	/* Try to read given stat file */
	if ((fp = fopen(filename, "r")) == NULL)
		return -1;

	i = fscanf(fp, "%lu %lu %lu %lu %lu %lu %lu %u %u %u %u %lu %lu %lu %u %lu %u",
		   &rd_ios, &rd_merges_or_rd_sec, &rd_sec_or_wr_ios, &rd_ticks_or_wr_sec,
		   &wr_ios, &wr_merges, &wr_sec, &wr_ticks, &ios_pgr, &tot_ticks, &rq_ticks,
		   &dc_ios, &dc_merges, &dc_sec, &dc_ticks,
		   &fl_ios, &fl_ticks);
``` 

数据输出: 

```
void write_plain_basic_stat(unsigned long long itv, int fctr,
			    struct io_stats *ioi, struct io_stats *ioj,
			    char *devname, unsigned long long rd_sec,
			    unsigned long long wr_sec, unsigned long long dc_sec)
{
	double rsectors, wsectors, dsectors;

	if (!DISPLAY_PRETTY(flags)) {
		cprintf_in(IS_STR, "%-13s", devname, 0);
	}

	rsectors = S_VALUE(ioj->rd_sectors, ioi->rd_sectors, itv);
	wsectors = S_VALUE(ioj->wr_sectors, ioi->wr_sectors, itv);
	dsectors = S_VALUE(ioj->dc_sectors, ioi->dc_sectors, itv);
	if (!DISPLAY_UNIT(flags)) {
		rsectors /= fctr;
		wsectors /= fctr;
		dsectors /= fctr;
	}

	/* tps */
	cprintf_f(NO_UNIT, 1, 8, 2,
		  /* Origin (unmerged) flush operations are counted as writes */
		  S_VALUE(ioj->rd_ios + ioj->wr_ios + ioj->dc_ios,
			  ioi->rd_ios + ioi->wr_ios + ioi->dc_ios, itv));

	if (DISPLAY_SHORT_OUTPUT(flags)) {
		/* kB_read/s kB_w+d/s */
		cprintf_f(DISPLAY_UNIT(flags) ? UNIT_SECTOR : NO_UNIT, 2, 12, 2,
			  rsectors, wsectors + dsectors);
		/* kB_read kB_w+d */
		cprintf_u64(DISPLAY_UNIT(flags) ? UNIT_SECTOR : NO_UNIT, 2, 10,
			    DISPLAY_UNIT(flags) ? (unsigned long long) rd_sec
						: (unsigned long long) rd_sec / fctr,
			    DISPLAY_UNIT(flags) ? (unsigned long long) wr_sec + dc_sec
						: (unsigned long long) (wr_sec + dc_sec) / fctr);
	}
	else {
		/* kB_read/s kB_wrtn/s kB_dscd/s */
		cprintf_f(DISPLAY_UNIT(flags) ? UNIT_SECTOR : NO_UNIT, 3, 12, 2,
			  rsectors, wsectors, dsectors);
		/* kB_read kB_wrtn kB_dscd */
		cprintf_u64(DISPLAY_UNIT(flags) ? UNIT_SECTOR : NO_UNIT, 3, 10,
			    DISPLAY_UNIT(flags) ? (unsigned long long) rd_sec
						: (unsigned long long) rd_sec / fctr,
			    DISPLAY_UNIT(flags) ? (unsigned long long) wr_sec
						: (unsigned long long) wr_sec / fctr,
			    DISPLAY_UNIT(flags) ? (unsigned long long) dc_sec
						: (unsigned long long) dc_sec / fctr);
	}

	if (DISPLAY_PRETTY(flags)) {
		cprintf_in(IS_STR, " %s", devname, 0);
	}
	printf("\n");
``` 

数据输出: 

```
void write_plain_basic_stat(unsigned long long itv, int fctr,
			    struct io_stats *ioi, struct io_stats *ioj,
			    char *devname, unsigned long long rd_sec,
			    unsigned long long wr_sec, unsigned long long dc_sec)
{
	double rsectors, wsectors, dsectors;

	if (!DISPLAY_PRETTY(flags)) {
		cprintf_in(IS_STR, "%-13s", devname, 0);
	}

	rsectors = S_VALUE(ioj->rd_sectors, ioi->rd_sectors, itv);
	wsectors = S_VALUE(ioj->wr_sectors, ioi->wr_sectors, itv);
	dsectors = S_VALUE(ioj->dc_sectors, ioi->dc_sectors, itv);
	if (!DISPLAY_UNIT(flags)) {
		rsectors /= fctr;
		wsectors /= fctr;
		dsectors /= fctr;
	}

	/* tps */
	cprintf_f(NO_UNIT, 1, 8, 2,
		  /* Origin (unmerged) flush operations are counted as writes */
		  S_VALUE(ioj->rd_ios + ioj->wr_ios + ioj->dc_ios,
			  ioi->rd_ios + ioi->wr_ios + ioi->dc_ios, itv));

	if (DISPLAY_SHORT_OUTPUT(flags)) {
		/* kB_read/s kB_w+d/s */
		cprintf_f(DISPLAY_UNIT(flags) ? UNIT_SECTOR : NO_UNIT, 2, 12, 2,
			  rsectors, wsectors + dsectors);
		/* kB_read kB_w+d */
		cprintf_u64(DISPLAY_UNIT(flags) ? UNIT_SECTOR : NO_UNIT, 2, 10,
			    DISPLAY_UNIT(flags) ? (unsigned long long) rd_sec
						: (unsigned long long) rd_sec / fctr,
			    DISPLAY_UNIT(flags) ? (unsigned long long) wr_sec + dc_sec
						: (unsigned long long) (wr_sec + dc_sec) / fctr);
	}
	else {
		/* kB_read/s kB_wrtn/s kB_dscd/s */
		cprintf_f(DISPLAY_UNIT(flags) ? UNIT_SECTOR : NO_UNIT, 3, 12, 2,
			  rsectors, wsectors, dsectors);
		/* kB_read kB_wrtn kB_dscd */
		cprintf_u64(DISPLAY_UNIT(flags) ? UNIT_SECTOR : NO_UNIT, 3, 10,
			    DISPLAY_UNIT(flags) ? (unsigned long long) rd_sec
						: (unsigned long long) rd_sec / fctr,
			    DISPLAY_UNIT(flags) ? (unsigned long long) wr_sec
						: (unsigned long long) wr_sec / fctr,
			    DISPLAY_UNIT(flags) ? (unsigned long long) dc_sec
						: (unsigned long long) dc_sec / fctr);
	}

	if (DISPLAY_PRETTY(flags)) {
		cprintf_in(IS_STR, " %s", devname, 0);
	}
	printf("\n");
```
```
void write_plain_ext_stat(unsigned long long itv, int fctr, int hpart,
			  struct io_device *d, struct io_stats *ioi,
			  struct io_stats *ioj, char *devname, struct ext_disk_stats *xds,
			  struct ext_io_stats *xios)
{
	int n;

	/* If this is a group with no devices, skip it */
	if (d->dev_tp == T_GROUP)
		return;

	if (!DISPLAY_PRETTY(flags)) {
		cprintf_in(IS_STR, "%-13s", devname, 0);
	}

	/* Compute number of devices in group */
	if (d->dev_tp > T_GROUP) {
		n = d->dev_tp - T_GROUP;
	}
	else {
		n = 1;
	}

	if (DISPLAY_SHORT_OUTPUT(flags)) {
		/* tps */
		/* Origin (unmerged) flush operations are counted as writes */
		cprintf_f(NO_UNIT, 1, 8, 2,
			  S_VALUE(ioj->rd_ios + ioj->wr_ios + ioj->dc_ios,
				  ioi->rd_ios + ioi->wr_ios + ioi->dc_ios, itv));
		/* kB/s */
		if (!DISPLAY_UNIT(flags)) {
			xios->sectors /= fctr;
		}
		cprintf_f(DISPLAY_UNIT(flags) ? UNIT_SECTOR : NO_UNIT, 1, 9, 2,
			  xios->sectors);
		/* rqm/s */
		cprintf_f(NO_UNIT, 1, 8, 2,
			  S_VALUE(ioj->rd_merges + ioj->wr_merges + ioj->dc_merges,
				  ioi->rd_merges + ioi->wr_merges + ioi->dc_merges, itv));
		/* await */
		cprintf_f(NO_UNIT, 1, 7, 2,
			  xds->await);
		/* areq-sz (in kB, not sectors) */
		cprintf_f(DISPLAY_UNIT(flags) ? UNIT_KILOBYTE : NO_UNIT, 1, 8, 2,
			  xds->arqsz / 2);
		/* aqu-sz */
		cprintf_f(NO_UNIT, 1, 7, 2,
			  S_VALUE(ioj->rq_ticks, ioi->rq_ticks, itv) / 1000.0);
		/*
		 * %util
		 * Again: Ticks in milliseconds.
		 */
		cprintf_pc(DISPLAY_UNIT(flags), 1, 6, 2, xds->util / 10.0 / (double) n);
	}
	else {
		if ((hpart == 1) || !hpart) {
			/* r/s */
			cprintf_f(NO_UNIT, 1, 7, 2,
				  S_VALUE(ioj->rd_ios, ioi->rd_ios, itv));
			/* rkB/s */
			if (!DISPLAY_UNIT(flags)) {
				xios->rsectors /= fctr;
			}
			cprintf_f(DISPLAY_UNIT(flags) ? UNIT_SECTOR : NO_UNIT, 1, 9, 2,
				  xios->rsectors);
			/* rrqm/s */
			cprintf_f(NO_UNIT, 1, 8, 2,
				  S_VALUE(ioj->rd_merges, ioi->rd_merges, itv));
			/* %rrqm */
			cprintf_pc(DISPLAY_UNIT(flags), 1, 6, 2,
				   xios->rrqm_pc);
			/* r_await */
			cprintf_f(NO_UNIT, 1, 7, 2,
				  xios->r_await);
			/* rareq-sz  (in kB, not sectors) */
			cprintf_f(DISPLAY_UNIT(flags) ? UNIT_KILOBYTE : NO_UNIT, 1, 8, 2,
				  xios->rarqsz / 2);
		}
		if ((hpart == 2) || !hpart) {
			/* w/s */
			cprintf_f(NO_UNIT, 1, 7, 2,
				  S_VALUE(ioj->wr_ios, ioi->wr_ios, itv));
			/* wkB/s */
			if (!DISPLAY_UNIT(flags)) {
				xios->wsectors /= fctr;
			}
			cprintf_f(DISPLAY_UNIT(flags) ? UNIT_SECTOR : NO_UNIT, 1, 9, 2,
				  xios->wsectors);
			/* wrqm/s */
			cprintf_f(NO_UNIT, 1, 8, 2,
				  S_VALUE(ioj->wr_merges, ioi->wr_merges, itv));
			/* %wrqm */
			cprintf_pc(DISPLAY_UNIT(flags), 1, 6, 2,
				   xios->wrqm_pc);
			/* w_await */
			cprintf_f(NO_UNIT, 1, 7, 2,
				  xios->w_await);
			/* wareq-sz (in kB, not sectors) */
			cprintf_f(DISPLAY_UNIT(flags) ? UNIT_KILOBYTE : NO_UNIT, 1, 8, 2,
				  xios->warqsz / 2);
		}
		if ((hpart == 3) || !hpart) {
			/* d/s */
			cprintf_f(NO_UNIT, 1, 7, 2,
				  S_VALUE(ioj->dc_ios, ioi->dc_ios, itv));
			/* dkB/s */
			if (!DISPLAY_UNIT(flags)) {
				xios->dsectors /= fctr;
			}
			cprintf_f(DISPLAY_UNIT(flags) ? UNIT_SECTOR : NO_UNIT, 1, 9, 2,
				  xios->dsectors);
			/* drqm/s */
			cprintf_f(NO_UNIT, 1, 8, 2,
				  S_VALUE(ioj->dc_merges, ioi->dc_merges, itv));
			/* %drqm */
			cprintf_pc(DISPLAY_UNIT(flags), 1, 6, 2,
				   xios->drqm_pc);
			/* d_await */
			cprintf_f(NO_UNIT, 1, 7, 2,
				  xios->d_await);
			/* dareq-sz (in kB, not sectors) */
			cprintf_f(DISPLAY_UNIT(flags) ? UNIT_KILOBYTE : NO_UNIT, 1, 8, 2,
				  xios->darqsz / 2);
		}
		if ((hpart == 4) || !hpart) {
			/* f/s */
			cprintf_f(NO_UNIT, 1, 7, 2,
				  S_VALUE(ioj->fl_ios, ioi->fl_ios, itv));
			/* f_await */
			cprintf_f(NO_UNIT, 1, 7, 2,
				  xios->f_await);
			/* aqu-sz */
			cprintf_f(NO_UNIT, 1, 7, 2,
				  S_VALUE(ioj->rq_ticks, ioi->rq_ticks, itv) / 1000.0);
			/*
			 * %util
			 * Again: Ticks in milliseconds.
			 */
			if (d->dev_tp > T_GROUP) {
				n = d->dev_tp - T_GROUP;
			}
			else {
				n = 1;
			}
			cprintf_pc(DISPLAY_UNIT(flags), 1, 6, 2, xds->util / 10.0 / (double) n);
		}
	}

	if (DISPLAY_PRETTY(flags)) {
		cprintf_in(IS_STR, " %s", devname, 0);
	}
	printf("\n");
}
``` 

# 关联

根据文档 <https://www.kernel.org/doc/Documentation/iostats.txt> : /proc/diskstats 与 /sys/block/{dev}/stat 数据同源
