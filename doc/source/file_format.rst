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

In some files, such as ``adjustments.propcol``, there seem to be not one but
*two* #FT2 sections, with #Fil sections that appear *after* the first #FT2 
section.

Header
======

This header is common to both Affinity container files and the files nested inside.

+---------+-------------+---------------------+-------------------------------+
| Count   | Type        | name                | Description                   |
+=========+=============+=====================+===============================+
|    4    |  uint8_t    | signature           | Signature. Set to 00 FF 4B 41 |
+---------+-------------+---------------------+-------------------------------+
+    1    |  uint32\_t  | version             | See Notes                     |
+---------+-------------+---------------------+-------------------------------+
|    1    |  uint32\_t  | file type           | kind of document contained    |
+---------+-------------+---------------------+-------------------------------+

The last byte of the signature is 0x41 for the main container documents and 
apparently 0x53 for most of the documents it contains (persona documents, 
swatches etc.).


Notes
-----

Signature
~~~~~~~~~

In palette files, the Swatches.doc starts out with the first 3 bytes identical
to the container signature, with only the last byte differing, so we can assume
that this is not just one large uint32_t.

The last byte seems to indicate the file type, with 0x41 indicating the main 
container format.

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

+-----------+------------------------+--------------------------+--------------------------------------------------------------------------------+
| File Type | Extensions / Name      | Main document file name  | Description                                                                    |
+===========|========================+==========================+================================================================================+
| Prsn      | .afdesign, .afphoto    | doc.dat                  | Persona Document                                                               |
+-----------+------------------------+--------------------------+--------------------------------------------------------------------------------+
| BrAr      | .afbrushes, .afmacros  | brushes.dat / Macros.dat | Brush Archive or Macro Archive                                                 |
+-----------+------------------------+--------------------------+--------------------------------------------------------------------------------+
| AsAr      | .afassets              | Assets.dat               | Assets Archive (1.5 and up)                                                    |
+-----------+------------------------+--------------------------+--------------------------------------------------------------------------------+
| Swth      | .afpalette             | Swatches.dat             | Swatches                                                                       |
+-----------+------------------------+--------------------------+--------------------------------------------------------------------------------+
| Pref      | preferences.dat        | preferences              |                                                                                |
+-----------+------------------------+--------------------------+--------------------------------------------------------------------------------+
| Adjm      | adjustments.propcol    | structure                |                                                                                |
+-----------+------------------------+--------------------------+--------------------------------------------------------------------------------+
| AstP      | assets.propcol         | structure                |                                                                                |
+-----------+------------------------+--------------------------+--------------------------------------------------------------------------------+
| CroP      | croppresets.propcol    |                          | No presets, so file empty in my case, but filename would likely be "structure" |
+-----------+------------------------+--------------------------+--------------------------------------------------------------------------------+
| DevP      | develop.propcol        | structure                |                                                                                |
+-----------+------------------------+--------------------------+--------------------------------------------------------------------------------+
| Fils      | fills.propcol          | structure                |                                                                                |
+-----------+------------------------+--------------------------+--------------------------------------------------------------------------------+
| FoMa      | font_map.dat           | n/a                      | See notes - not really an Affinity Container file                              |
+-----------+------------------------+--------------------------+--------------------------------------------------------------------------------+
| Macs      | macros.propcol         | structure                |                                                                                |
+-----------+------------------------+--------------------------+--------------------------------------------------------------------------------+
| Objs      | objects.propcol        | structure                |                                                                                |
+-----------+------------------------+--------------------------+--------------------------------------------------------------------------------+
| OSty      | objectstyles.propcol   | (none)                   |                                                                                |
+-----------+------------------------+--------------------------+--------------------------------------------------------------------------------+
| RBru      | raster_brushes.propcol | (none?)                  |                                                                                |
+-----------+------------------------+--------------------------+--------------------------------------------------------------------------------+
| Shps      | shapes.propcol         | structure                |                                                                                |
+-----------+------------------------+--------------------------+--------------------------------------------------------------------------------+
| TonP      | tone_map.propcol       | structure                |                                                                                |
+-----------+------------------------+--------------------------+--------------------------------------------------------------------------------+
| VBru      | vector_brushes.propcol | structure                |                                                                                |
+-----------+------------------------+--------------------------+--------------------------------------------------------------------------------+

preferences.dat is found in ~/Library/Containers/com.seriflabs.affinitydesigner 
(for Designer) or ~/Library/Containers/com.seriflabs.affinityphoto (for Photo) 
on macOS, in the subdirectory /Data/Library/Application Support.

This folder also has a "user" subfolder that contains additional files with a
.propcol extension that use the Affinity container format as well.

The font_map.dat file is interesting in that the last byte of the signature of 
the main file is 0x53, just like for the Swatches.dat files. Hence this is not
an Affinity Container file, but the font map data file is stored without a 
container. The .dat extension is a further indication, as is the fact, that 
font_map.dat does not have the complete header of the container format. 
The two-byte version field that follows the signature is set to 1 (unlike for
Swatches.dat, where it is set to 2).

It is also worth noting that some of the .propcol files seem to use an earlier
version of the container format, with some like the raster_brushes.propcol using
version 8 instead of 10, even for Affinity 1.5. This may be due to them being
created by older versions and never having been re-written since.


#Inf section
============

In container files, this is followed by the '#Inf' section.

+---------+-------------+---------------------+-------------------------------+
| Count   | Type        | name                | Description                   |
+=========+=============+=====================+===============================+
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
|    4    |     char    | Prot marker         | "Prot"                        |
+---------+-------------+---------------------+-------------------------------+
|    1    | uint32\_t   | unknown             | usually non-zero              |    
+---------+-------------+---------------------+-------------------------------+

Notes
-----

Offsets
~~~~~~~

It the first number encountered in the offsets is the location of the #FAT 
section.

The second number is the absolute offset of the end of #FAT (or #FT2) section.
Since Designer files contain an additional PNG thumbnail after the FAT, this can
not be assumed to be the total size of the container file.

If there are multiple #FAT/#FT2 sections (such as in the adjustments.propcol 
file), the offset field in the main header points to the last #FT2 section in 
the file.

If the file is empty (i.e. no #Fil sections), the #FAT/#FT2 offset is set to
zero and no #FAT/#FT2 section is written into the file (can be observed in
an empty croppresets.propcol file).

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

If there are no files in the container (i.e. no #Fil sections), the #FT2 section
is not written at all and the corresponding offset field in the main header is
set to zero.

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

CrvD (Curve Description?)
-------------------------

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
individual chunks and starts with a similar signature and version.

Unfortunately, there is no size or version field in the individual chunks, 
meaning that it is not possible to skip unknown or unsupported chunks when 
reading. It also makes it difficult to spot potential nested chunks.

Some of the chunks in a swatches document also seem to pop up in a doc.dat file.
At this point, it is not clear if this is because of embedded swatches, or if
some of these chunks also have meaning in context of the main document.

Swatches.dat starts out with a signature that's curiously similar to the 
signature of the container file format. This signature is "00 FF 4B 53" hex. 
Only the last byte is different (0x53 for swatch files, 0x41 for the container 
file).

This is followed by a two-byte field that might be a version (set to 2 in my 
test file that was generated by Affinity 1.5).

After this, the chunk data follows.

Tags are again human-readable four character constants that appear flipped in a 
hex editor due to little endian storage.

Here are the ones that are known so far. Note that since there are no size 
fields, some of these might be nested in others or only happen to be four 
characters in length but not tags.

An empty application palette consists of PalV, PlCN, PalV, and a PanV chunk in
that order.

The pattern 'FilS' - 'Colr' - 'RGBA' - 'colD' seems to be repeated for each 
swatch in the file, though 'FilS' - 'Fill' - 'Colr' - 'RGBA' - 'colD' also seems
to be a possibility.

Hence we can surmise that 'FilS' is a "Fill Setting" and the other tags are 
possibly nested and that the file format is not based on basic chunks like
TIFF or PNG, which also explains the absence of version or size fields.

One possible explanation is that the serializer for each C++ class starts the 
writing process with a four-byte signature tag, followed by whatever data it
wants to save. So if the class saves multiple data fields, some of which are 
classes that know how to serialize themselves, these would also save their tags,
hence leading to nesting. If the tag was connected to a factory, this would also
allow the system to instantiate polymorphic objects when reading the files.

This would imply that there is some kind of root object that starts the process.
By this logic, this would be the first one in the file, 'PalV'. For main 
documents (doc.dat) it would be 'DocR'.

One question this raises is how data from base classes are saved. If the 
serializer saves base class data first, this would result in the derived class
writing its signature, followed immediately by the base class signature. This
type of combination theoretically stand out in files.

So far, this is just speculation, though.

PalV
----

Palette V(alue? ersion?)

Payload is variable length. So far, 5, 7, and 10 bytes have been observed. 
Occured once in one test file, but twice in another (empty application palette).

Based on the 5-byte version, this is the data layout that can be reconstructed.

+-------+-----------+-------------+---------------------------------------------+
| count | type      | name        | description                                 |
+=======+===========+=============+=============================================+
| 1     | uint16\_t | unknown1    |                                             |
+-------+-----------+-------------+---------------------------------------------+
| 1     | uint16\_t | unknown2    |                                             |
+-------+-----------+-------------+---------------------------------------------+
| ...   | ...       | ...         | If first field is non-zero, more data here. |
|       |           |             | If zero, there seems to be one byte of      |
|       |           |             | non-zero data here.                         |
+-------+-----------+-------------+---------------------------------------------+


For an occurence with the first two fields being 0, the size was 5 bytes.
For an occurence with the first two fields being 1 and 3, the total size was 7
bytes. For 8 and 0, 10 bytes.

PlCN
----

Palette C(ontainer?) Name

Contains the name of the swatch palette as a string. Layout is as follows

+----------+-----------+-------------+-------------------------------------+
| count    | type      | name        | description                         |
+==========+===========+=============+=====================================+
| 1        | uint32\_t | tag         | 'PlCN'                              |
+----------+-----------+-------------+-------------------------------------+
| 1        | uint32\_t | name length | number of characters in name        |
+----------+-----------+-------------+-------------------------------------+
| variable | char      | name        | name of the palette as 8-bit string |
+----------+-----------+-------------+-------------------------------------+
| 1        | byte      | unknown     | set to 0xB1                         |
+----------+-----------+-------------+-------------------------------------+

FilS
----

Fill Setting (?)

Constant length of 4 bytes, usually set to 1 (boolean stored as uint32\_t?)

Fill
----

Only came up once, with 5 bytes of payload

Colr
----

Multiple instance. Payload length is 6 bytes (unless RGBA and HSLA are nested 
tags, which we don't know).

Layout seems to be UInt8, UInt8, then 00 00 00 01 hex. Since the format is 
little endian, it is unlikely that it is a UInt32. If it is two shorts, it might
be minimum and maximum intensity (0 and 255 decimal respectively). Or it might 
be four times independent UInt8, one each of an RGBA value for instance.

This tag does not come up in an empty swatches file.

RGBA, HSLA
----------

No payload or four bytes observed so far. Possibly the last part of the 'Colr'
chunk. Usually followed by a ColD chunk.

colD
----

Multiple occurences, length is varying. Lengths of 20 and 25 bytes have been 
observed.

This seems to contain the actual color values for the swatch.

+----------+-----------+--------------+-------------------------------------+
| count    | type      | name         | description                         |
+==========+===========+==============+=====================================+
| 1        | uint32\_t | tag          | 'colD'                              |
+----------+-----------+--------------+-------------------------------------+
| 1        | uint8\_t  | unknown      | usually 0x5F                        |
+----------+-----------+--------------+-------------------------------------+
| 4        | float32   | color values | R-G-B-A                             |
+----------+-----------+--------------+-------------------------------------+
| variable | byte      | unknown      | unknown - may not be present        |
+----------+-----------+--------------+-------------------------------------+

Note that Affinity also saves a noise component for all colors (accessible in
the UI by clicking the opacity dot in the Color panel). This is presumably saved
in one of the unknown fields.

TBD: How does this look for HSL, CMYK, or Spot color swatches?

The curious changing length at the end of the data indicates that this chunk is
either actually part of another chunk, or that at least its length is determined
by data in one of the preceding chunks.

PaNV
----

Palette Name Values?

This chunk contains a list of the names of all swatches. 

NOTE: While these tend to contain actual color values, this is due to the fact
that Affinity names swatches based on their numeric values by default.

The actual color data is stored in other chunks (see ColD).

Layout of the 'PaNV' chunk is as follows

+--------+-----------+--------------+--------------------------------+
| count  | type      | name         | description                    |
+========+===========+==============+================================+
| 1      | uint32\_t | tag          | 'PaNV'                         |
+--------+-----------+--------------+--------------------------------+
| 1      | uint32\_t | data size    | number of bytes of swatch data |
+--------+-----------+--------------+--------------------------------+
| 1      | uint32\_t | num swatches | swatch count. See notes.       |
+--------+-----------+--------------+--------------------------------+

If the data size is 0, num swatches appears to only be one single zero byte.

What follows is the name data. The names are stored as 8-bit pascal strings, 
presumably UTF-8.

Each entry is a uint32\_t length field, followed by as many characters as 
specified by that field. There is no zero terminator.