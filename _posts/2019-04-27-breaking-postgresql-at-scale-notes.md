---
layout: article
title: Breaking PostgreSQL at Scale notes
date: 2019-04-27 12:28:18
categories: programming
tags:
- programming
- databases
- postgresql
---

I watched a great talk from the CEO of PG Experts about working with PostgreSQL at scale. This is interesting to me because we have at least one half-TB database at work that we're interested in sharding soon, but also because we've hit a few mysterious database issues in the past couple of weeks that require some advanced diagnostics.

The synopsis of the talk is kind of, "Postgres is great out of the box and here are some things you can tune to make it better at different orders of magnitude".

* ~10GB
  * how much memory?
    * db should fit in memory
  * backups
    * pglog
  * failover
    * manual
  * replication
    * maybe a primary and secondary
    * you can use direct streaming or WAL archiving
  * tuning
    * use a couple of special in-memory configs (i haven't done research to figure out what these are)
  * upgrades
    * pg_dump/pg_restore
* ~100GB
  * too much for memory at this point
  * queries start to slow down
  * pg_dump is too slow
  * how much memory?
    * shrugs
    * a good rule of thumb is, "can you fit the largest 1-3 indexes in memory?"
    * if your logging is set up to do so, you can see if your queries are creating large tmp files and maybe set `work_mem` equal to 2-3x the largest file.
      * i'm not completely clear on what `work_mem` does. there seem to be quite a few memory configs and i'm not sure if this is the root config or something separate.
  * backups
    * Point in Time backups
      * copies the filesystem every so often and saves WAL segments that occur while in the midst of copying. Together, these comprise your backup.
    * there are some libs that can help with this
  * load balancing
    * use replicas for reads but be aware that replication lag is a real risk here. Make sure your application isn't making assumptions about strong consistency - ideally, at the application-level you're aware of whether you're working with the master or a replica and can act accordingly.
    * may be time to set up connection pooling. he's seen some setups where people have thousands of open connections to the db but they're mostly idle.
  * monitoring
    * you probably want good monitoring at this scale, not just someone tweeting at you to tell you your site is down - I laughed out loud at this.
    * pg_stat_statements

  * queries
    * pg_badger / pg_stat_statements to show slow queries
    * missing indexes start to become apparent
  * availability
    * start looking at failover tooling vs manual switch
  * upgrades
    * use pg_upgrade
* 1TB
  * starting to get big
  * IO throughput is important
    * In addition to normal query workflows, the vacuum process is also spinning up workers and doing more at this point too.
  * backups
    * use incremental backups
  * restrain yourself
    * keep shared buffers to 16-32GB
    * larger will increase checkpoint activity w/o a corresponding boost in performance.
      * i'm unclear what checkpoints are used for or why more mem != more perf at this point
    * be careful with maintenance_work_mem as too high can stall vacuum
  * load balancing
    * definitely use read replicates
    * you may want a dedicated failover candidate that's very up-to-date with the master and doesn't accept queries
      * there's a tradeoff in streaming replication between the queries the server accepts and how close it stays to the primary
    * you also want good tooling for spinning up replicas (e.g. ops as code or something)
  * offload services
    * don't do analytics against the primary, move that data elsewhere (e.g. warehouse)
    * consider logical replicas for above
    * move job queues into redis or something - don't keep them in the main db as there's probably too much activity
  * vacuum
    * can start to take a long time
    * only increase autovacuum workers if you have a large number of tables. workers are per-table, so scaling workers won't necessarily finish the job faster.
    * let the vacuum jobs complete. he's had clients that have paid him thousands just for him to say, "let your vacuum jobs finish".
    * be careful with two-phase commit (he recommends not using it at all)
      * what is this?
    * don't turn autovacuum off
      * you can use autovacuum_vacuum_cost_delay as a knob to make vacuum more or less aggressive
  * Indexes
    * start getting large
    * consider using partial indexes
    * look at which of your indexes are actually being used
    * you may start to see queries that were doing index scans going to bitmap index/heap scans
    * make sure you're using the right indexes for your column types
  * partitioning
    * you should start partitioning your data into different tables
    * be sure the table has a strong partioning key
      * need to do more research here
  * parallel query execution
    * turn it on
  * statistics targets
    * may need to tune this so the planner can understand your data more thoroughly

I stopped here but there were some finer points in here that I skipped over (note-taking fatigue). Overall, great talk.
