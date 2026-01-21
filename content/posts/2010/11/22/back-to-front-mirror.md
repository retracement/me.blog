+++
date = '2010-11-22T14:13:00+01:00'
draft = false
title = 'Understanding Database Mirroring modes of operation'
+++


SQL Server database mirroring was obviously a great feature first introduced in SQL 2005 and will improve even further in the next big release (Denali) but quite surprising to me is how drastically the order in which the transactions are committed differ between both modes of operation.

To recap, there are two types of mirroring (High Performance and High Safety) so lets look at exactly how these two types work.

# High Performance

In order for database mirroring to be achieved, high performance sends transactions asynchronously and this is the order of events:

1. The client application sends a transaction to the principal server which then writes it to the transaction log and commits it.
1. The principal sends an acknowledgement back to the client application.
1. The principal then sends the transaction to the mirror who writes an entry to the transaction log.
1. The mirror then contacts the principal to acknowledge the transaction and then commits it.

*Note that since these first two steps are identical on a database with or without mirroring, you can appreciate why this operation is fast and has little impact to the speed of transactions on the principal.*

![Objects in the mirror are closer than they appear](/images/2010/hope.jpg)

# High Safety

High Safety mode is considered to be a synchronous level of mirroring and the order of events are as follows:
1. The client application sends a transaction to the principal server and then writes it to the transaction log.
1. The principal then sends the transaction to the mirror who writes and entry to the transaction log.
1. The mirror then contacts the principal to acknowledge the transaction, the principal subsequently sends an acknowledgement back to the client application.
1. The principal commits the transaction and then the mirror commits the transaction.

I think the biggest surprise to me really is that in High Safety mode, because the transaction on the primary is committed only after the mirror initiated handshake you are actually reducing the probability of a successful commit for any given transaction (due to the dependency on the mirror). So although in High Safety the principal and mirror would be transactionally in sync, *I personally believe this is another reason why High Performance should be preferred in nearly all situations*.

# Conclusion
I should also add that for the different modes of operation (especially High Safety), depending upon the information source you refer to, the exact mechanism and in particular (for High Safety step 4) the order each server's transaction is committed is a little ambiguous to say the least. The following url titled [Synchronous Database Mirroring (High-Safety Mode)](https://msdn.microsoft.com/en-us/library/ms179344.aspx) from Books Online also leaves a little too much open to interpretation.

For further information on Database Mirroring best practices, please refer to this rather good white paper titled [Database Mirroring Best Practices and Performance Considerations](https://technet.microsoft.com/en-gb/library/cc917681.aspx) from the SQL CAT team.