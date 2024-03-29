  
---  
  
Linear Tape File System  
  
The  **Linear Tape File System**  (**LTFS**) is a  [file system](https://en.wikipedia.org/wiki/File_system "File system")  that allows files stored on  [magnetic tape](https://en.wikipedia.org/wiki/Magnetic_tape_data_storage "Magnetic tape data storage")  to be accessed in a similar fashion to those on disk or removable flash drives. It requires both a specific format of data on the tape media and software to provide a file system interface to the data.  
  
The technology, based around a self-describing tape format developed by  [IBM](https://en.wikipedia.org/wiki/IBM "IBM"), was adopted by the  [LTO Consortium](https://en.wikipedia.org/wiki/Linear_Tape-Open "Linear Tape-Open")  in 2010.  
  
---  
  
| Format | LTO-1 | LTO-2 | LTO-3 | LTO-4 |LTO-5 |LTO-6 |LTO-7 | _Type&nbsp;M&nbsp;(M8)&nbsp;[[Note 1]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-12) | LTO-8 | LTO-9 | LTO-10 | LTO-11 | LTO-12 | LTO-13 | LTO-14 |  
|--|--|--|--|--|--|--|--|--|--|--|--|--|--|--|--|  
| Release date | 2000[[12]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-First_million-13) | 2003 | 2005 | 2007 | 2010[[13]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-LTO-5-14) | Dec. 2012[[14]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-Bechtle-15) | Dec. 2015[[15]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-LTO-7-16)[[16]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-reglto7b-17)[[17]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-lto7lic-18) | Dec. 2017 | <- | Sep. 2021[[18]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-19) | _TBA_ | _TBA_ | _TBA_ | _TBA_ | _TBA_ |  
| Uncompressed/[Native](https://en.wikipedia.org/wiki/Native_capacity "Native capacity")  capacity | 100 GB | 200 GB | 400 GB | 800 GB | 1.5 TB[[19]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-ltfs-20) | 2.5 TB[[20]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-lto6pressrelease-21) | 6.0 TB[[17]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-lto7lic-18)[[21]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-ltogenerations-22) | 9 TB | 12 TB[[22]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-Ts2280-23) | 18 TB[[23]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-lto9rel-24)[[19]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-ltfs-20)[[24]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-lto910-25) | 36 TB[[25]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-roadmap1120-26) | 72 TB[[25]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-roadmap1120-26) | 144 TB[[25]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-roadmap1120-26) | 288 TB[[25]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-roadmap1120-26) | 576 TB[[25]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-roadmap1120-26) |  
| Compressed capacity | 200 GB | 400 GB | 800 GB | 1.6 TB | 3.0 TB | 6.25 TB | 15 TB | 22.5 TB | 30 TB | 45 TB | 90 TB | 180 TB | 360 TB | 720 TB | 1,440 TB |  
| Max uncompressed speed (MB/s)[[21]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-ltogenerations-22)[[Note 2]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-27) | 20 | 40 | 80 | 120 | 140 | 160 | 300[[26]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-reglto7a-28) | <- | 360 | 400 | _1,100_ | _TBA_ | _TBA_ | _TBA_ | _TBA_ |  
| Max compressed speed (MB/s) | 40 | 80 | 160 | 240 | 280 | 400 | 750 | <- | 900 | 1,000 | _2,750_ | _TBA_ | _TBA_ | _TBA_ | _TBA_ |  
| Time to write a full tape at max speed(hh:mm) | 1:23 | <- | <- | 1:51 | 3:10 | 4:20 | 5:33 | 8:20 | 9:16 | 12:30 | _12:07_ | _TBA_ | _TBA_ | _TBA_ | _TBA_ |  
| [Compression](https://en.wikipedia.org/wiki/Data_compression "Data compression")  capable? | Yes, "2:1" | <- | <- | <- | <- | Yes, "2.5:1" | <- | <- | <- | <- | Planned, "2.5:1"[[25]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-roadmap1120-26)[[24]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-lto910-25)[[27]](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_note-lto78-29) | <- | <- | <- | <- |  
| [WORM](https://en.wikipedia.org/wiki/Write_Once_Read_Many "Write Once Read Many")  capable? | No | <- | Yes | <- | <- | <- | <- | No | Yes | <- | Planned | <- | <- | <- | <- |  
| [Encryption](https://en.wikipedia.org/wiki/Encryption "Encryption")  capable? | No | <- | <- | Yes | <- | <- | <- | <- | <- | <- | Planned | <- | <- | <- | <- |  
| [LTFS](https://en.wikipedia.org/wiki/Linear_Tape_File_System "Linear Tape File System")  capable? | No | <- | <- | <- | Yes | <- | <- | <- | <- | <- | Planned | <- | <- | <- | <- |  
| Max. number of partitions | 1 (no partitioning) | <- | <- | <- | 2 | 4 | <- | <- | <- | <- | Planned | <- | <- | <- | <- |  
  
---  
  
1.  **[^](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_ref-12 "Jump up")**  Previously unused LTO-7 tape, not an independent generation, part of LTO-8 generation. See:  [Compatibility](https://en.wikipedia.org/wiki/Linear_Tape-Open#Compatibility)  
2.  **[^](https://en.wikipedia.org/wiki/Linear_Tape-Open#cite_ref-27 "Jump up")**  Maximum uncompressed speeds valid for full height drives. Half height drives may not attain the same speed. Check manufacturer's specifications.