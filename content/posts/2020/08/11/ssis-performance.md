+++
date = '2020-08-11T15:18:01+01:00'
draft = false
title = 'SSIS Data Flow Performance Tuning'
+++
Yes before you say it, I know SQL Server Integration Services is "old technology" but a lot of people are still using it, and in many cases are either still developing against it, or are looking to integrate/ migrate with other burgeoning technologies such as Azure Data Factory. In other words, if you are not currently using SSIS then this post is probably not for you -otherwise read on.

If you are still one of the lucky ones to still be using SSIS, I thought it would be worth publishing these comprehensive notes taken from a session titled "SSIS Data Flow Performance Tuning" delivered at SQLBits 8 (Brighton) by the then "SSIS guru" Jamie Thomson. Notes have timings (in mins and seconds) against them, which correlate directly with the presentation times. The video is still available and can be downloaded from the [SQLBits website](https://sqlbits.com/Sessions/Event8/SSIS_Dataflow_Performance_tuning) so you can watch it (if required) and use the timings to follow along.

---

# Presentation notes
**0m45s.** Buffer memory is an area on memory for SSIS data flow components. It does not move or change shape. Buffer is created with all columns that could be used rather than the ones that are used. Therefore using derived column components does not change the shape of the buffer.

**2m.** Data in a buffer can be changed/overwritten rather than new rows needed.

A buffer is a collection of many rows in memory.

A buffer is populated by the source adapter and has a certain number of rows, and that is what is operated on as a single unit of work, before being passed onto the next buffer in the data flow.

Buffers created by asynchronous components.

Data viewer component shows the data in data buffers.

At the bottom of the data viewer you can actually see the number of rows per buffer and the number of buffers (the latter of which will start at 1 until you start pushing the viewer on or detaching the viewer).

![Data Viewer](/images/2020/data-viewer.png)

**4m37s.** SSIS dataflow made up of components (aka transformations). There are two types of transformations – synchronous and asynchronous.
Synchronous components are ones that consume a data buffer and might even transform/ change the data in some way, but will then output and pass that exact same buffer onto the next component to do its work on. Basically, this is an efficient way of consuming the data. The crucial point is that the output buffer memory address is exactly the same as the input buffer memory address.

Asynchronous components are ones that consume a data buffer on input, do processing from a buffer (might be a sort operation or aggregation operation etc) and create a new buffer on output. This is a more expensive operation and is generally why asynchronous components are thought of as being slower, however, there are a few examples when asynchronous can be quicker.

**6m.** Asynchronous components can also be categorized as partially blocking or fully blocking. e.g sort component is fully blocking and is because all upstream buffers are needed before sending sorted output downstream. An example of a partially blocking component is the merge join component which can send output before it has all upstream buffers.

**7m05s.** Synchronous components output the exact number of rows inputted. They are generally very quick and use the same buffers for IO. Examples are: derived column, conditional split, and multicast. In the case of a conditional split, even if rows don't match the condition, they are still outputted but are excluded from the input of the next components (they are classed as "dangling rows").
Multicast outputs are not duplicating rows for inputs (although logically it does). Instead, pointer magic is used.

**8m33s.** Asynchronous components create new buffers, have differently shaped (row count or column count) IO buffers, and can have different numbers of rows for input and outputs. Are usually slower. Examples are the aggregate and sort components.

**10m.** SSIS has a concept of execution trees. The execution tree is a section of the data flow starting from an asynch output and terminating at input/s of asynchronous components. Basically means that each asynch component starts a new execution tree. The art of performance tuning data flows is really the art of performance tuning execution trees.

![Execution trees](/images/2020/execution-trees.png)

Buffers never actually move, they are either owned by a specific component or a copy is made (depending upon the component type).

**15m.** The Data Flow Task is performant by design without tuning or optimization and settings generally deliver great performance. For example, you are usually not going to get any performance improvement by tweaking the number of rows per buffer. The best way to speed up data flows is to design them properly in the first place.

For instance:

- Remove unrequired columns (heed ssis warnings)
- For fixed width files, only parse the columns that you need
- Always use a SQL statement (data access mode) rather than use the OLEDB source editor data access mode of Table or View (see below) -so that you can filter out columns not needed. This can significantly reduce time

   ![OLE DB Source Editor](/images/2020/oledb-source-editor.webp)

- Parse only when needed (or leave them as strings). The flat file source adapter will more efficiently read strings, so if you really don't need to do it then don't. For instance, converting to datetime is quite an expensive operation (see below). You can potentially do conversion downstream after something like aggregation.

   ![Flat File Connection Manager](/images/2020/flat-file-conn-manager.webp)

**23m.** Does a demo using the Lookup Transformation component for type two SCDs. Uses the Advanced tab to change the query. Basically says that doing lookups like this is very slow since it has to do this every time for every row in the data flow (because it cannot cache every single row in memory).
Instead, he uses a merge join component against the lookup data set. Says you need to sort both the fact data and the lookup data for the merge join component.

![Merge Join Component](/images/2020/merge-join.png)

Because the lookup data is type 2, there will be potentially more records in our merged output he uses a conditional split component to remove those rows that don't meet the predicates (ie where the effective date is between the startdate and enddate).

![Split Component](/images/2020/split-component.png)

**30m.** Shows a demo about pulling in records from csv without parsing the file from the flat file adapter and instead of bringing in each row as a whole. This is undoubtedly quicker (especially if you had mostly duplicates and a very small subset), but you would still need to parse (and derive the columns).

In order to bring in each row as an unparsed row, I believe he used the "Ragged right" method (as below) which you configure in the Flat file connection manager.

![Flat File Connection Manager Editor](/images/2020/flat-file-conn-manager-editor.webp)

See the technique below for parsing the columns manually.

![Derived Transformation Editor](/images/2020/derived-transformation-editor.webp)

**35m.** General Tuning tips

- Use FastParse if possible (is a setting on flat file source adapters under certain conditions).
- Turn off OnPipelineRowsSent -this is an event that you can capture in your logfile but it outputs data for every single buffer going through every single component – is useful for debugging but is very bad for performance.
- Let the database do what it is good at – so think about doing things like sorts and aggregations at the database level.
- Consider using Raw files – these are a very good way to pass data across data flows – they are essentially a copy of the buffer data dumped to disk. Could also be a great way to do some form of checkpointing to the workflow. Write to raw is very quick as is read from raw.
- Using 64bit SSIS is better to ensure larger addressable memory space.
- Keep columns narrow. By default the string column sizes will default to 50 – this can be a performance killer. Make them no bigger than the size they need to be – the reason for this is that it will prevent SSIS over-allocating memory.
- Increase Default BufferMaxSize & DefaultBufferMaxRows where necessary to improve performance – see the "SQL 2005 Integration Services: Performance Tuning Techniques" white paper by Microsoft (Elizabeth Vitt et al) and also the "SQL 2012 SSIS Operational and Tuning Guide" white paper by Microsoft looks very useful too!
- Point BufferTempStoragePath/ BLOBTempStoragePath at fast drives.
- Optimise the destination (use fast load, table lock, simple/bulk logged recovery/ disable indexes).
- Identify bottlenecks.
- Never use the SSIS destination (think he meant SQL destination). It was put in SSIS 2005 as a quick way of putting data into SQL Server if the server you were inserting into was the same that you were running a package. There were some enhancements to the OLEDB destination made that effectively makes that always a quicker option.

**45m.** demo showing that sometimes splitting work over multiple components can be quicker than doing so over less.

**50m.** If data is in a trusted format, you can set the FastParse option to true on the output column which will act like a go faster switch particularly for things like dates. If the format of something is wrong it will cause and error when it is hit. This only works on the flat file source adapter.

![Advanced Editor For Flat File](/images/2020/advanced-editor-for-flat-file.png)

**=the end=**