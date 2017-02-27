=====================
.afdesign file format
=====================
.. include:: <isonum.txt>

Values are stored in little-endian format

Affinity files are containers that contain one or more nested files. The 
container file format is not only used for Designer/Photo/Publisher documents,
but also for secondary file types like swatch and brush libraries, macros and 
so on.

The file starts out with a header, followed by an #Inf section that points to
the location of the FAT in the file, then the individual files follow in #Fil 
sections, and the files end with a table of contents called FAT (presumably
an acronym for something like "File Allocation Table"). If there is an embedded
thumbnail, it follows at the end in PNG format, but is not part of the FAT.

These nested files can have file names, though it looks like only one file will
have a reliable name ("doc.dat"), and other files will be in a directory 
structure (d/a, d/c, b/a...). This may or may not be related to incremental 
saving.

Files can be uncompressed or compressed via zlib.

You will find the term "Persona" creeping up from time to time. This is Serif's
internal code name for the Affinity series and is not to be confused with the UI
concept of a Persona (essentially, a Workspace within the application).

The file format is designed for really fast loads and saves. As such, there 
seems to be a mechanism for incremental saves that does not re-write the entire
document. Doing a "Save As" in the main application appears to re-write and
thus consolidate all the information in the document.

I haven't seen anything that actually looks like a flags field, usually
there are a few boolean feilds so idk what some of these fields are for

It looks the the ``#Fil`` section is ended with ``0xFFFFFFFF``. The #FAT section
also ends with this marker if a thumbnail follows, otherwise this marker is 
omitted (eg. for swatch files).

Header
======

Size: 68 bytes? Variable length?

+---------+-------------+---------------------+-------------------------------+
| Count   | Type        | name                | Description                   |
+=========+=============+=====================+===============================+
|    4    |  uint8_t    | signature           | Signature. Set to 00 FF 4B 41 |
+---------+-------------+---------------------+-------------------------------+
+    1    |  uint32\_t  | version             | See Notes                     |
+---------+-------------+---------------------+-------------------------------+
|    1    |  uint32\_t  | file type           | kind of document contained    |
+---------+-------------+---------------------+-------------------------------+
+    4    +     char    | info section marker | "#Inf"                        |
+---------+-------------+---------------------+-------------------------------+
|         |             | FAT\_offset         | See Notes on offset           |
+         +             +---------------------+-------------------------------+
|         |             | FAT\_end\_offset    | See Notes on offset           |
+         + uint64\_t   +---------------------+-------------------------------+
|         |             | data\_length        | See Notes on offset           |
+         +             +---------------------+-------------------------------+
|         |             | unused / unknown    | apparently always zero        |
+    1    +-------------+---------------------+-------------------------------+
|         |             | creation            | Date (unix timestamp)         |
+         +             +---------------------+-------------------------------+
|         |             | unknown / unused    | apparently always zero        |
+         + uint32\_t   +---------------------+-------------------------------+
|         |             | unknown             | unknown                       |
+         +             +---------------------+-------------------------------+
|         |             | unknown             | unknown                       |
+---------+-------------+---------------------+-------------------------------+
|    4    |     char    | prot marker         | "Prot"                        |
+---------+-------------+---------------------+-------------------------------+
|    1    | uint32\_t   | unknown             | usually non-zero              |    
+---------+-------------+---------------------+-------------------------------+


Notes
-----

Signature
~~~~~~~~~

In palette files, the Swatches.doc starts out with the first 3 bytes identical
to the container signature, with only the last byte differing, so we can assume
that this is not just one large uint32_t.

Version
~~~~~~~

The version field indicates the version of the file format overall. The header
seems to have remained at least mostly identical between these versions. It is
likely that this refers to the features required in the Affinity application to 
interpret all the data and not necessarily implies changes in the binary 
structure aside from new chunks for additional features.

It is likely that this version is incremented whenever Serif breaks 
compatibility with earlier versions.

Here are the known file format versions and the versions of the Affinity 
software that are known to write them:

+--------------+---------------------------+
| File Version | Affinity software version |
+==============+===========================+
|      8       | TBD                       |
+--------------+---------------------------+
|      9       | TBD                       |
+--------------+---------------------------+
|     10       | 1.5                       |
+--------------+---------------------------+

Note that in version 10, as saved by Affinity 1.5, the '#FAT' table is replaced
with a newer version with the marker '#FT2'.

File Type
~~~~~~~~~

The file type is a four character code packed into a uint32_t. This code is used
to indicate whether the file contains a regular document, brush archive, 
swatches palette or similar.

Since Affinity files are stored in little endian byte order, the characters are
essentially reversed from how they appear in a C++ literal. For instance, 
something originally defined as const uint32_t SIGNATURE = 'Prsn'; will end up
appearing as 'nsrP' in a hex editor. The following values are byte swapped for 
your convenience.

Here are the file types known so far with the corresponding extensions.

+-----------+-----------------------+--------------------------+--------------------------------+
| File Type | Extensions            | Main document file name  | Description                    |
+===========|=======================+==========================+================================+
| Prsn      | .afdesign, .afphoto   | doc.dat                  | Persona Document               |
+-----------+-----------------------+--------------------------+--------------------------------+
| BrAr      | .afbrushes, .afmacros | brushes.dat / Macros.dat | Brush Archive or Macro Archive |
+-----------+-----------------------+--------------------------+--------------------------------+
| AsAr      | .afassets             | Assets.dat               | Assets Archive (1.5 and up)    |
+-----------+-----------------------+--------------------------+--------------------------------+
| Swth      | .afpalette            | Swatches.dat             | Swatches                       |
+-----------+-----------------------+--------------------------+--------------------------------+

Offsets
~~~~~~~

It the first number encountered in the offsets is the location of the #FAT 
section.

The second number is the absolute offset of the end of #FAT (or #FT2) section.
Since Designer files contain an additional PNG thumbnail after the FAT, this can
not be assumed to be the total size of the container file.

If there is only one nested file in the container, the data\_length field seems
to be the compressed size of said file, excluding the #Fil marker that strats a
file section and the FFFFFFFF marker that ends it. Not sure what this value 
means if there are multiple files in the container. TBD.

#FAT section
============

The FAT is the table that indicates the offsets, compression info and so on for
all files in the container. Unless there is an embedded thumbnail, this is the
last thing in the file.

This allows the application to append data to the file and then just rewrite the
FAT without having to shift any of the information before it.

The #FAT section is used in file format version prior to 10. In version 10, it 
was replaced with the #FT2 table (see below).

Ok so, the data ends in the file name or something? a short will
indicate the length of the filename?

It looks like this section will define the offset of the #Fil sections
and their length

Looks like a entry starts with one byte, a 64 bit nubmer and a timestap

It looks like each section starts with 1 byte for flags, then a 8 byte
timestamp

|darr| This might be the FAT header? |darr|

+---------------+---------+-------------+---------------------------------+
| name          | count   | Type        | description                     |
+===============+=========+=============+=================================+
| flags         |         |  uint64\_t  | Flags?                          |
+---------------+         +-------------+---------------------------------+
| date          | 1       |             | UNIX timestamp                  |
+---------------+         +  uint32\_t  +---------------------------------+
| flags         |         |             | more flags                      |
+---------------+---------+-------------+---------------------------------+
| offsets       | 4       |  uint64\_t  | offsets                         |
+---------------+---------+-------------+---------------------------------+
| fat\_length   | 1       |  uint16\_t  | The length of the fat section   |
+---------------+---------+-------------+---------------------------------+
| idk           | 3       | char        |                                 |
+---------------+---------+             +---------------------------------+
| idk           | 2       |             |                                 |
+---------------+---------+-------------+---------------------------------+

^ I am pretty sure this is misaligned, there are more pad/flag bytes
than I want, so I'm probably missing something. Also doc.dat occurs at
the end of the file so this might be half a entry?

Also, the last offset could very well be the number of entries

Sometimes the timestamp seems to be wildly off, but was the file
creation date for the doc.dat

+----------------+------------+-------------+----------------------------------------+
| name           |  count     | type        | description                            |
+================+============+=============+========================================+
| idk            |            | uint32\_t   | idk. It's always zero?                 |
+----------------+            +-------------+----------------------------------------+
| idk\_Again     |            | bool        | looks like a bitfield or a number...   |
+----------------+            +-------------+----------------------------------------+
| data\_offset   |            |             | The offset of the data                 |
+----------------+            +             +----------------------------------------+
| real\_len      |            |  uint64\_t  | uncompressed length                    |
+----------------+    1       +             +----------------------------------------+
| data\_len      |            |             | The lengh of the chunk                 |
+----------------+            +-------------+----------------------------------------+
| date           |            | uint32\_t   | the date                               |
+----------------+            +-------------+----------------------------------------+
| compressed     |            | bool        | if the chunk is compressed             |
+----------------+            +-------------+----------------------------------------+
| fname\_len     |            | uint16\_t   | The filename length                    |
+----------------+------------+-------------+----------------------------------------+
| filename       |*fname\_len*| char        | The filename                           |
+----------------+------------+-------------+----------------------------------------+

#FT2 Section
============

NOTE: Peter here - I have not looked into the accuracy of the #FAT description, so 
some of my findings here might be relevant to the FAT section as well and vice
versa. These should ideally be compared and amended at some point.

The #FT2 section replaces the older #FAT section in Affinity versions from 1.5
on (file format version 10).

This is the header for the version 2 FAT:

+---------+-----------+----------------+-------------------------------------------------------------------------+
| count   | type      | name           | description                                                             |
+=========+===========+================+=========================================================================+
| 4       | char      | signature      | "#FT2"                                                                  |
+---------+-----------+----------------+-------------------------------------------------------------------------+
| 8       | byte      | unknown        | zero in test files. Might be a uint64\_t.                               |
+---------+-----------+----------------+-------------------------------------------------------------------------+
| 1       | uint32\_t | timestamp      | unix timestamp                                                          |
+---------+-----------+----------------+-------------------------------------------------------------------------+
| 4       | byte      | unknown        | apparently always 0. Maybe the timestamp is stored in a 64-bit integer? |
+---------+-----------+----------------+-------------------------------------------------------------------------+
| 1       | uint64\_t | fat end offset |                                                                         |
+---------+-----------+----------------+-------------------------------------------------------------------------+

This is followed by the individual entries. What's interesting is that there 
seems to be no "numEntries" field. It can be assumed that the Affinity software
reads the FAT with sort of a "while offset < fatEndOffset" type of loop.

FAT version 2 entries have the following structure:

+----------+-----------+------------------+----------------------------------------------------------------------------+
| count    | type      | name             | description                                                                |
+==========+===========+==================+============================================================================+
| 1        | uint64\_t | data length      | length of the nested file in bytes                                         |
+----------+-----------+------------------+----------------------------------------------------------------------------+
| 8        | bytes     | unknown          | usually zero. Might be 64-bit (u)int.                                      |
+----------+-----------+------------------+----------------------------------------------------------------------------+
| 1        | uint64\_t | unknown          | set to 1 in my swatches file -- possibly a bool?                           |
+----------+-----------+------------------+----------------------------------------------------------------------------+
| 12       | bytes     | unknown          | unknown data, not all of which is zero, first 4 bytes might be a UInt32    |
+----------+-----------+------------------+----------------------------------------------------------------------------+
| 1        | uint64\_t | data offset      | start offset of the corresponding #Fil section                             |
+----------+-----------+------------------+----------------------------------------------------------------------------+
| 1        | uint64\_t | data length      | again length of nested file in bytes                                       |
+----------+-----------+------------------+----------------------------------------------------------------------------+
| 1        | uint64\_t | data length      | again length of nested file in bytes                                       |
+----------+-----------+------------------+----------------------------------------------------------------------------+
| 5        | bytes     | unknown          | unknown partially non-zero data. First four bytes might be a timestamp?    |
+----------+-----------+------------------+----------------------------------------------------------------------------+
| 1        | uint32\_t | unknown          | set to 3 for my Swatches.dat                                               |
+----------+-----------+------------------+----------------------------------------------------------------------------+
| 1        | uint16\_t | file name length | number of characters in the file name                                      |
+----------+-----------+------------------+----------------------------------------------------------------------------+
| variable | char      | file name        | file name as 8-bit string without null terminator                          |
+----------+-----------+------------------+----------------------------------------------------------------------------+

This has been deduced from a swatches file where the nested Swatches.dat file
is not compressed, hence information about which fields are compressed size and
which ones are the inflated size is still TBD.

TBD - Not sure yet if there is a flag somewhere that indicates whether a file is
compressed or not, or if this is deduced by comparing compressed and 
uncompressed size.

The version 2 FAT is ended with a section end marker (FFFFFFFF) if more data
(i.e. a thumbnail image) follows, otherwise this marker is omitted.

Thumbnail
=========

The thubnail, if present, follows after the #FAT/#FT2. It is only stored for
documents, not for swatches etc.

Note that the thumbnail is not a nested file in the container and thus is not
listet in the FAT.

The Section looks like this:

+-----------+-----------------------+-------------------------------------+
| Type      | Name                  | Description                         |
+===========+=======================+=====================================+
| char[4]   | Marker                | "Thmb"                              |
+-----------+-----------------------+-------------------------------------+
| uint32\_t | unknown               | unknown                             |
+-----------+-----------------------+-------------------------------------+
| uint32\_t | unknown               | unknown                             |
+-----------+-----------------------+-------------------------------------+
| uint64\_t | unknown               | unknown                             |
+-----------+-----------------------+-------------------------------------+
| uint32\_t | thumbnail data length | length of the PNG thumnail in bytes |
+-----------+-----------------------+-------------------------------------+
| 2 bytes   | unknown               | non-zero                            |
+-----------+-----------------------+-------------------------------------+
| variable  | thumbnail data        | length as per field above           |
+-----------+-----------------------+-------------------------------------+

Some of the unknown fields might be thumbnail width and height or similar.

If you want to read the embedded thumbnail, the best strategy is to look in the
header for the FAT end offset, then seek to it. If it is EOF, there is no 
thumbnail. If not, check for the "Thmb" marker, and if it is there, read the
thumbnail data as specified here.


doc.dat
=======

This is the main file contained in .afdesign or .afphoto files.

It looks like the tags are strings, but where stored in 32-bit integers
when saved. The names of each chunk are reversed and are each characters long.

They were probably stored something like this:

.. code-block:: c
  :linenos:

  #include <stdio.h>
  #include <stdint.h>

  int main(){
    union {
      uint32_t tag;
      char name[4];
    };
    strncpy(name, "Prsn", 4);
    printf("0x%X\n", tag);
    // On a little endian system, this should output
    // 0x6E737250
    return 0;
  }

It looks like the capitalization of the chunks do not matter to indicate flags,
like the PNG format does.. So that's cool.

NOTE: Peter here -- most C++ compilers allow you to write these like basic char
literals. For example: uint32_t my_const = 'Prsn';


SprB (Document properties?)
---------------------------


Total size: 36 bytes

+-------+-------+-----------+-------------------------------+
| name  | count |  type     | description                   |
+=======+=======+===========+===============================+
| BrpS  |   1   | uint32\_t | Dimension info                |
+-------+-------+-----------+-------------------------------+
|  ?    |   2   |           | Idk? looks to be zero usually |
|       |       |           | Maybe some offset info        |
|       |       |           | like x and y offset           |
|       |       |  double   | maybe w\ |times|\ h+x+y       |
+-------+-------+           +-------------------------------+
| width |   1   |           | Width                         |
+-------+       +           +-------------------------------+
| height|       |           | Height                        |
+-------+-------+-----------+-------------------------------+

Opac
----

Total size: 8 bytes

+-----------+---------+-------------+------------------------------+
| name      | count   | type        | Descrpition                  |
+===========+=========+=============+==============================+
| Opac      | 1       | uint32\_t   | Tag                          |
+-----------+         +-------------+------------------------------+
| opacity   |         | float       | The opacity of the element   |
+-----------+---------+-------------+------------------------------+

Visi
----

Total Size: 5 Bytes

+-----------+---------+-------------+-----------------------------+
| name      | count   | type        | Descrpition                 |
+===========+=========+=============+=============================+
| Visi      | 1       | uint32\_t   | The tag                     |
+-----------+         +-------------+-----------------------------+
| visible   |         | bool        | visibility of the element   |
+-----------+---------+-------------+-----------------------------+

Desc
----

Total Size: variable (smallest is 6 bytes)

+--------+---------+-------------+------------------------+
| name   | count   | type        | Descrpition            |
+========+=========+=============+========================+
| Desc   | 1       | uint32\_t   | The tag                |
+--------+         +-------------+------------------------+
| size   |         | uint16\_t   | Length of the name     |
+--------+---------+-------------+------------------------+
| name   | *size*  | char        | The name of the Desc   |
+--------+---------+-------------+------------------------+

Mrgn
----

This field is speculation, I haven't gotten time to look at it yet But,
based on the size of the data I am assuming.

Total size: 36bytes

+----------+---------+----------+-----------------+
| name     | count   | type     | Descrpition     |
+==========+=========+==========+=================+
| Mrgn     |         | char     | The Tag         |
+----------+         +----------+-----------------+
| left     |         |          | left margin     |
+----------+         +          +-----------------+
| top      |    1    |          | top margin      |
+----------+         +  double  +-----------------+
| right    |         |          | right margin    |
+----------+         +          +-----------------+
| bottom   |         |          | bottom margin   |
+----------+---------+----------+-----------------+

Data
----

Total Size: variable (smallest is 6 bytes)

+--------+---------+-------------+----------------------+
| name   |  count  | type        | Descrpition          |
+========+=========+=============+======================+
| Data   |         | uint32\_t   | The tag              |
+--------+    1    +-------------+----------------------+
| size   |         | uint16\_t   | Length of the data   |
+--------+---------+-------------+----------------------+
| data   | *size*  | byte        | The data             |
+--------+---------+-------------+----------------------+

Root
----

Speculation, again. Presumably the offset of the root node Total Size: 8
bytes

+----------+---------+-------------+-----------------------+
| name     |  count  | type        | Descrpition           |
+==========+=========+=============+=======================+
| Root     |         |             | The tag               |
+----------+    1    +  uint32\_t  +-----------------------+
| offset   |         |             | Offset to Root node   |
+----------+---------+-------------+-----------------------+

NgoL (Logarithm in base N?)
---------------------------
Ok, to be honest I have no clue what this means. I don't know how long it is, 
as I can't figure out where the length is specified

UOrg (???)
-------------
User organization? maybe the layers?

BmpW (Bitmap width)
-------------------
Width of a bitmap

+----------+---------+-------------+-----------------------+
| name     |  count  | type        | Descrpition           |
+==========+=========+=============+=======================+
| tag      |    1    | uint32_t    | WpmB                  |
+----------+---------+-------------+-----------------------+
| width    |    1    | uint32_t    | Width                 |
+----------+---------+-------------+-----------------------+


BmpH (Bitmap Height)
--------------------
Height of a bitmap

+----------+---------+-------------+-----------------------+
| name     |  count  | type        | Descrpition           |
+==========+=========+=============+=======================+
| tag      |    1    | uint32_t    | HpmB                  |
+----------+---------+-------------+-----------------------+
| height   |    1    | uint32_t    | Height                |
+----------+---------+-------------+-----------------------+

Bitm (Bitmap)
-------------

+----------+---------+-------------+-----------------------+
| name     |  count  | type        | Descrpition           |
+==========+=========+=============+=======================+
| tag      |         | uint32_t    | Bitm                  |
+----------+         +-------------+-----------------------+
| flag     |    1    |   bool      | boolean               |
+----------+         +-------------+-----------------------+
| size?    |         | uint32_t    |                       |
+----------+---------+-------------+-----------------------+


Frmt (Format)
-------------

+----------+---------+-------------+-----------------------+
| name     |  count  | type        | Descrpition           |
+==========+=========+=============+=======================+
| tag      |    1    | uint32_t    | Frmt                  |
+----------+---------+-------------+-----------------------+
| flag     |    1    | unit32_t    | Format?               |
+----------+---------+-------------+-----------------------+

Shap (Shape)
------------
Shape info? I tried creating a test document with several shapes. But there were only one of these....

Stri (String)
-------------

+----------+---------+-------------+-----------------------+
| name     |  count  | type        | Descrpition           |
+==========+=========+=============+=======================+
| tag      |    1    | uint32_t    | Stri                  |
+----------+---------+-------------+-----------------------+

FOpc (F? Opacity)
-----------------

+----------+---------+-------------+-----------------------+
| name     |  count  | type        | Descrpition           |
+==========+=========+=============+=======================+
| tag      |    1    |  uint32_t   | FOpc                  |
+----------+---------+-------------+-----------------------+
| opacity  |    1    |  float      |                       |
+----------+---------+-------------+-----------------------+

Blnd (Blend)
------------

Layer mode? Blending mode? Blendy blend blend.


XMPD (XMP Data)
---------------

The XMP Data

+----------+---------+-------------+-----------------------+
| name     |  count  | type        | Descrpition           |
+==========+=========+=============+=======================+
| tag      |    1    |  uint32_t   | XMPD                  |
+----------+---------+-------------+-----------------------+
| length   |    1    |  uint32_t   | Length of the XMP     |
|          |         |             | data                  |
+----------+---------+-------------+-----------------------+
| data     |*length* | char        | XML String            |
+----------+---------+-------------+-----------------------+

Symb (Symbol)
-------------


Post (Postscript)
-----------------
Post script name of a font

+----------+---------+-------------+-----------------------+
| name     |  count  | type        | Descrpition           |
+==========+=========+=============+=======================+
| tag      |    1    |  uint32_t   | Post                  |
+----------+---------+-------------+-----------------------+
| length   |    1    |  uint32_t   | Length of the font    |
|          |         |             | name                  |
+----------+---------+-------------+-----------------------+
| name     |*length* | char        |                       |
+----------+---------+-------------+-----------------------+

Famy (Font Family)
------------------
Font family of a font

+----------+---------+-------------+-----------------------+
| name     |  count  | type        | Descrpition           |
+==========+=========+=============+=======================+
| tag      |    1    |  uint32_t   | Famy                  |
+----------+---------+-------------+-----------------------+
| length   |    1    |  uint32_t   | Length of the font    |
|          |         |             | family name           |
+----------+---------+-------------+-----------------------+
| Family   |*length* |  char       |                       |
+----------+---------+-------------+-----------------------+


Crvs (Curves)
-------------
Curves

+----------+---------+-------------+-----------------------+
| name     |  count  | type        | Descrpition           |
+==========+=========+=============+=======================+
| tag      |    1    |  uint32_t   | Crvs                  |
+----------+---------+-------------+-----------------------+

Rect (Rect)
-----------
Rectangle, 4 32-bit integers, order x, y, w, h


Chld (Children)
---------------
Looks like it indicates how many children a field has


Swatches.dat
============

This file is the main (and usually only) file in an .afpalette file that 
contains color swatches. It is usually stored uncompressed.

The file format is similar to the doc.dat format in that it is made of 
individual chunks.

Unfortunately, there is no size or version field, meaning that it is not 
possible to skip unknown or unsupported chunks when reading.

At this point, it is unknown if the chunks in a swatches document are shared 
with the doc.dat format or if they can only appear in a Swatches.dat file.

Swatches.dat starts out with a signature that's curiously similar to the 
signature of the container file format. This signature is "00 FF 4B 53" hex. 
Only the last byte is different (0x53 for swatch files, 0x41 for the container 
file).

This is followed by a two-byte field that might be a version (set to 2 in my 
test file that was generated by Affinity 1.5).

After this, the chunk data follows.

Tags are again human-readable four character constants that appear flipped in a hex editor due
to little endian storage.

Here are the ones that are known so far. Note that since there are no size 
fields, some of these might be nested in others or only happen to be four 
characters in length but not tags.

PalV
----

Palette V(alue? ersion?)

Comes up first and only once in my sample file and the payload without the tag
is 7 bytes in length.

PlCn
----

Palette C(ount?)

Only one instance, payload probably one single uint32_t. There was one more 
swatch in the file than the value of this int, so it might be the index of the
last item in the palette (i.e. count - 1).

FilS
----

Constant length of 1 byte, usually set to 1 (boolean?)

Fill
----

Only came up once, with 5 bytes of payload

Colr
----

Multiple instance. Payload length is 6 bytes, unless RGBA and HSLA are nested 
tags.

RGBA, HSLA
----------

No payload or four bytes. Possibly the last part of the 'Colr' chunk. Usually 
followed by a ColD chunk.

colD
----

Multiple occurences, length is constant 25 bytes


PaNV
----

Palette N... Values?

This contains the actual color swatch definitions as ASCII strings in the vein
of "R:230 G:23 B:118" or "H:108 S:89 L:37".

Layout of the chunk is as follows

+--------+-----------+--------------+--------------------------------+
| count  | type      | name         | description                    |
+========+===========+==============+================================+
| 1      | uint32\_t | data size    | number of bytes of swatch data |
+--------+-----------+--------------+--------------------------------+
| 1      | uint32\_t | num swatches | swatch count. See notes.       |
+--------+-----------+--------------+--------------------------------+

Note: The swatch count field might also be the bit depth of the color 
specifications (I had 8 swatches in my test file so I can't say for sure until
I check more files), but I think swatch count is a lot more likely.

What follows is the swatch data. Curiously, the color values are stored as 
ASCII pascal strings.

Each entry is a uint32\_t length field, followed by as many characters as 
specified by that field. There is no zero terminator.

The strings have the form "R:230 G:23 B:118" or "H:108 S:89 L:37", i.e. the 
color model is implicit, values are stored as 8-bit ASCII text with no zero 
padding, but a single whitespace to separate the values (probably for easy
parsing with sscanf).