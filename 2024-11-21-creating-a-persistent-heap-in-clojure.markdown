---
layout: post
title:  "Creating a persistent Heap in Clojure"
date:   2024-11-21 20:41:17 -0300
categories: clojure
---

Goal: Develop an application that simulates the storage of data
blocks in a hierarchical storage system using a binary heap.

Each block has a “weight” (or priority) that represents its size or
frequency of access.


# Requirements
- Java 8+
- Clojure 1.11

# Architecture:

- Storing the files in a certain order is called File Organization
- File Structure refers to the format of the label and data blocks and
- of any logical control record.  Use Heap File Organization to
- organize files

A file has r = 20,000 STUDENT records of fixed length, each record has
the following fields: KEY# (7 bytes), VAL MAX(1024 bytes)

An additional byte is used as a deletion marker.
This file is stored on the disk with the following characteristics:
block size B = 512 bytes; inter-block gap; size G = 128 bytes;
number of blocks per track = 20; number of tracks
per surface = 400.

- Records in a file can be physically ordered based on the values of
  one of their fields.
- Locating the next record from the current one in order of the
  ordering field usually requires no additional block accesses,
  because the next record is often stored in the same block (unless
  the current record is the last one in the block).
- Retrieval using a search condition based on the value of the
  ordering field can be efficient when the binary search technique is
  used.

# Heap File Organization

Heap File Organization works with data blocks. In this method, records
are inserted at the end of the file, into the data blocks. No Sorting
or Ordering is required in this method. If a data block is full, the
new record is stored in some other block.

- The records of a file must be allocated to disk blocks a block is
 the unit of data transfer between disk and main memory
- When the record size is smaller than the block size, a block can
  accommodate many such records.
- If a record has too large a size to be fitted in one block, two or
  more blocks will have to be used.
- If we do not want to waste the unused spaces in the blocks, we may
  choose to store part of a record in them and the rest of the record
  in another block.
- A pointer at the end of the first block points to the block
  containing the other part of the record, in case it is not the next
  consecutive block on disk.
- For variable-length records using spanned organisation, each block
  may store a different number of records.

# File headers

- A file normally contains a file header or file descriptor providing
  information which is needed by programs that access the file
  records.
- The contents of a header contain information that can be used to
  determine the disk addresses of the file blocks, as well as to
  record format descriptions, which may include field lengths and
  order of fields within a record for fixed-length unspanned records,
  separator characters, and record type codes for variable-length
  records.
- To search for a record on disk, one or more blocks are transferred
  into main memory buffers. Programs then search for the desired
  record or records within the buffers, using the header information.
- If the address of the block that contains the desired record is not
  known, the programs have to carry out a linear search through the
  blocks. Each block is loaded into a buffer and checked until either
  the record is found or all the blocks have been searched
  unsuccessfully (which means the required record is not in the file).
- The goal of a good file organisation is to locate the block that
  contains a desired record with a minimum number of block transfers.

# Operation on Files

- Operations on files can usually be grouped into retrieval operations
  and update operations.
- To insert a new record, we must first find its correct position
  among existing records in the file, according to its ordering field
  value. Then a space has to be made at that location to store it. This
  involves reorganisation of the file, and for a large file it can be
  very time-consuming. The reason is that on average, half the records
  of the file must be moved to make the space.
- For record deletion, the problem is less severe, if deletion markers
  are used and the file is reorganised periodically.


# Binary Search Algorithm

A binary search for disk files can be performed on the blocks rather
than on the records. Suppose that:

- the file has b blocks numbered 1, 2, ..., b;
- the records are ordered by ascending value of their ordering key;
- we are searching for a record whose ordering field value is K;
- disk addresses of the file blocks are available in the file header.


# Protocol

The system can issue requests to carry out the following operations
(with assistance from the operating-system file/disk managers)

- Find: Searches for the first record satisfying a search condition (a
  condition specifying the criteria that the desired records must
  satisfy). Transfers the block containing that record into a buffer
  (if it is not already in main memory). The record is located in the
  buffer and becomes the current record (ready to be processed).

- Read (or Get): Copies the current record from the buffer to a
  program variable. This command may also advance the current record
  pointer to the next record in the file.

- FindNext: Searches for the next record in the file that satisfies
  the search condition. Transfers the block containing that record
  into a buffer, and the record becomes the current record.

- Delete: Deletes the current record and updates the file on disk to
  reflect the change requested.

- Modify: Modifies some field values for the current record and
  updates the file on disk to reflect the modification.

- Insert: Inserts a new record in the file by locating the block where
  the record is to be inserted, transferring that block into a buffer,
  writing the (new) record into the buffer, and writing the buffer to
  the disk file to reflect the insertion.

- FindAll: Locates all the records in the file that satisfy a search
  condition.

- FindOrdered: Retrieves all the records in the file in a specified
  order.

- Reorganise: Rearranges records in a file according to certain
  criteria. An example is the 'sort' operation, which organises
  records according to the values of specified field(s).

- Open: Prepares a file for access by retrieving the file header and
  preparing buffers for subsequent file operations.

- Close: Signals the end of using a file.