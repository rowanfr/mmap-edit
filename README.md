# mmap-edit
**`mmap-edit` is in development and not for production use**. 

It is designed to be a rust crate that provides functionality to enable a memory map based file editor. This is designed to be an editor for large files, a small subsection of which is buffered into memory for editing.
The restrictions it will have are:
- It focuses primarily on in memory edits, not hard disk edits. While functionality for that may be enabled later for now this crate will focus on keeping any edits in memory through action chains of basic actions associated with byte positions.
- Referencing the previous section "markers or positions" I should clarify that this crate relies on working with random location accessing within the file and thus will only ever work on serialized files that use/are one of:
    - Fixed-size Records
    - Offset Tables or Lookup Tables
    - Basic Delimiters (Only works if delimiter is impossible in data (like null byte or newline in some formats). This can be seen in some styles of JSON and CSV files where `\n` serves as a newline)

  These options are due to necessary restriction that come from reading anywhere in the memory map. Fixed-size records allow for the calculation of the nearest starting point after the one requested, offset tables allow for the lookup of the nearest point after the one requested, and delimiters allow for the continued parsing of the memory map until it reaches a delimiter from which it orients further parsing. To be clear when stating that I want this crate to be able to start "reading anywhere", I want the user to be able to request a number of records starting from any byte but during deserialization one necessarily can't read from the middle of a given serialized record. As a result of that this crate will determine what is the next record in line after the requested "byte" and continue to return records sequentially from that point. This may be altered later to where it can also look backward to try to return as many records up until it reaches the number passed in.

- Very simple core set of core actions from which more complex actions can be created:
    - Create
    - Delete
    - Unchecked Modify (Some formats have rules about how to modify records, this as a result is not guaranteed to be safe and can make some records unreadable or error prone if used improperly)
- There must be some mechanism of ordering records so that chains can be ordered and searched in memory. This is typically a unique timestamps or some time step but it must be some unique order that can be used in chains. Further this too might eventually become either altered in some manner, such as having a given order passed in with records that can be used to orient action locations or having the location of records be an ordered enumerator that distinguishes between record and in memory action chain. My preference is for record order within the record itself in something akin to timestamps above, determination of given order passed in and enumerator distinguishing record and in memory action. Between the 2 remaining I would likely go with the enumerator as while it is more wasteful in the in memory data returned to a given number of requested records the determination of global order is much more complex and likely to result in similar global costs with a far harder to reason about implementation.

