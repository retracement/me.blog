+++
date = '2010-02-25T14:36:00+01:00'
draft = false
title = 'Good practices for manual SQL script promotion'
+++

An often overlooked part of promoting DML scripts is knowing what the intention is going to be before one is run. For instance if you work in an environment with a high throughput of DML script promotions then there is a good chance that those scripts will be ran via change control and then the results fed back. However what happens when the result counts end up being incorrect (i.e different to what was anticipated)? A rollback occurs and we are then talking about possible availability issues.

This is why it is very important to ensure that any DML script clearly has expected change counts listed and that there is always a `WHERE` clause specified in the DML statement. If the DML operation is on all rows of a table then the `WHERE` clause should still be included specifying `1=1` to demonstrate that this is the intent.

Finally executing the script in an open ended `BEGIN TRAN` would be preferable so that a simple `ROLLBACK` can be issued if the counts are incorrect. However be careful to not leave the transaction open otherwise you may cause lots of blocking activity on your server.

Another safety precautionary measure you can take in addition to backups could be creating a snapshot on the database prior to any `COMMIT` which would provide you other recovery options beyond restoring the database such as using the snapshot data to help revert the changed data back. Or worst case a very quick revert of the entire snapshot. In the latter scenario, the consideration would be that any transaction that occurred after the snapshot was taken would be lost - so be very careful to understand what you are doing.

Hope this post gives you some food for thought.