.. SPDX-License-Identifier: GPL-2.0+

Blob Lists - bloblist
=====================

Introduction
------------

A bloblist provides a way to store collections of binary information (blobs) in
a central structure. Each record of information is assigned a tag so that its
owner can find it and update it. Each record is generally described by a C
structure defined by the code that owns it.


Passing state through the boot process
--------------------------------------

The bloblist is created when the first U-Boot component runs (often SPL,
sometimes TPL). It is passed through to each successive part of the boot and
can be accessed as needed. This provides a way to transfer state from one part
to the next. For example, TPL may determine that a watchdog reset occurred by
reading an SoC register. Reading the register may reset the value, so that it
cannot be read a second time. So TPL can store that in a bloblist record which
can be passed through to SPL and U-Boot proper, which can print a message
indicating that something went wrong and the watchdog fired.


Blobs
-----

While each blob in the bloblist can be of any length, bloblists are designed to
hold small amounts of data, typically a few KB at most. It is not possible to
change the length of a blob once it has been written. Each blob is normally
created from a C structure which can beused to access its fields.


Blob tags
---------

Each blob has a tag which is a 32-bit number. This uniquely identifies the
owner of the blob. Blob tags are listed in enum blob_tag_t and are named
with a `BLOBT_` prefix.


Single structure
----------------

There is normally only one bloblist in U-Boot. Since a bloblist can store
multiple blobs it does not seem useful to allow multiple bloblists. Of course
there could be reasons for this, such as needing to spread the blobs around in
different memory areas due to fragmented memory, but it is simpler to just have
a single bloblist.


API
---

Bloblist provides a fairly simple API which allows blobs to be created and
found. All access is via the blob's tag. Blob records are zeroed when added.


Placing the bloblist
--------------------

The bloblist is typically positioned at a fixed address by TPL, or SPL. This
is controlled by `CONFIG_BLOBLIST_ADDR`. But in some cases it is preferable to
allocate the bloblist in the malloc() space. Use the `CONFIG_BLOBLIST_ALLOC`
option to enable this.

The bloblist is automatically relocated as part of U-Boot relocation. Sometimes
it is useful to expand the bloblist in U-Boot proper, since it may want to add
information for use by Linux. Note that this does not mean that Linux needs to
know anything about the bloblist format, just that it is convenient to use
bloblist to place things contiguously in memory. Set
`CONFIG_BLOBLIST_SIZE_RELOC` to define the expanded size, if needed.


Finishing the bloblist
----------------------

When a part of U-Boot is about to jump to the next part, it can 'finish' the
bloblist in preparation for the next stage. This involves adding a checksum so
that the next stage can make sure that the data arrived safely. While the
bloblist is in use, changes can be made which will affect the checksum, so it
is easier to calculate the checksum at the end after all changes are made.


Future work
-----------

Bootstage has a mechanism to 'stash' its records for passing to the next part.
This should move to using bloblist, to avoid having its own mechanism for
passing information between U-Boot parts.


Simon Glass
sjg@chromium.org
12-Aug-2018
