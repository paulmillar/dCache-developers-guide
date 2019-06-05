# A Developer's Guide to dCache

The dCache team develops and maintains software for storing large amounts of data, making use of using heteogenous hardware.  There are dCache instances that store many tens of petabytes of data, transfer petabytes of data per day, instances that span multiple countries, and instances that fit on a Raspberry Pi.

The dCache software is freely available and open-source.

## What this document is

This document is meant for people who want to modify how dCache behaves: either fixing a bug or adding some exciting new feature.  We hope that you will contribute your changes so others can benefit from them, so a large fraction of this document describes how to interact with the rest of the dCache team to facilitate this.  The underlying idea is that this document helps new contributors become productive quickly.

Additionally, documentation on dCache's release process is included which is aimed at members of DESY's development team.

## What this document isn't

This document is not a manual on how to use dCache software.  That topic is covered by "The dCache Book".  It is also not a design document that describes how dCache works.  That document, as a single document, is currently missing; however, important aspects of dCache design are describe in seperate documents.

This guide is not authorative on how to get your code into dCache.  There may be some aspect that this document has missed and that, in the process of becoming an active dCache developer, you have to find out for yourself. If so, please let the document's editor \(Paul Millar\) know, so helpfully future developers will have an easier time.

It also is not a guarantee: following the advice here does not mean that your work will be accepted.  There may be many factors that affect whether code is accepted.  This document cannot cover them all.



