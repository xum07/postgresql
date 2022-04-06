事务提交的主流程：

```c
FinishPreparedTransaction
   |--- RecordTransactionAbortPrepared
   |       |--- TransactionIdAbortTree
   |               |--- TransactionIdSetTreeStatus
   |--- RecordTransactionCommitPrepared
           |--- TransactionIdCommitTree
                   |--- TransactionIdSetTreeStatus
                   
AbortTransaction
AbortSubTransaction
   |--- RecordTransactionAbort
           |--- nchildren = xactGetCommittedChildren(&children);   // 获取当前事务下的子事务xid，及其数量
           |--- TransactionIdAbortTree
                   |--- TransactionIdSetTreeStatus

CommitTransaction
   |--- RecordTransactionCommit
           |--- nchildren = xactGetCommittedChildren(&children);   // 获取当前事务下的子事务xid，及其数量
           |--- TransactionIdCommitTree
                   |--- TransactionIdSetTreeStatus
```

子事务的xid获取：

```c
RecordTransactionAbort
RecordTransactionCommit
   |--- nchildren = xactGetCommittedChildren(&children);
   
FinishPreparedTransaction
   |--- /*
         * Read and validate 2PC state data. State data will typically be stored
         * in WAL files if the LSN is after the last checkpoint record, or moved
         * to disk if for some reason they have lived for a long time.
         */
    char	   *buf;
	if (gxact->ondisk)
		buf = ReadTwoPhaseFile(xid, false);
	else
		XlogReadTwoPhaseData(gxact->prepare_start_lsn, &buf, NULL);

    TwoPhaseFileHeader* hdr = (TwoPhaseFileHeader *) buf;
	char* bufptr = buf + MAXALIGN(sizeof(TwoPhaseFileHeader));
	bufptr += MAXALIGN(hdr->gidlen);
	children = (TransactionId *) bufptr;
```





# 参考资料

1. [子事务示例](https://zhuanlan.zhihu.com/p/147605189)
