.. index:: compute torque/chunk

compute torque/chunk command
============================

Syntax
""""""

.. code-block:: LAMMPS

   compute ID group-ID torque/chunk chunkID

* ID, group-ID are documented in :doc:`compute <compute>` command
* torque/chunk = style name of this compute command
* chunkID = ID of :doc:`compute chunk/atom <compute_chunk_atom>` command

Examples
""""""""

.. code-block:: LAMMPS

   compute 1 fluid torque/chunk molchunk

Description
"""""""""""

Define a computation that calculates the torque on multiple chunks of
atoms.

In LAMMPS, chunks are collections of atoms defined by a
:doc:`compute chunk/atom <compute_chunk_atom>` command, which assigns each atom
to a single chunk (or no chunk).  The ID for this command is specified
as chunkID.  For example, a single chunk could be the atoms in a
molecule or atoms in a spatial bin.  See the
:doc:`compute chunk/atom <compute_chunk_atom>` and
:doc:`Howto chunk <Howto_chunk>`
doc pages for details of how chunks can be defined and examples of how
they can be used to measure properties of a system.

This compute calculates the three components of the torque vector for eqch
chunk, due to the forces on the individual atoms in the chunk around
the center-of-mass of the chunk.  The calculation includes all effects
due to atoms passing through periodic boundaries.

Note that only atoms in the specified group contribute to the
calculation.  The :doc:`compute chunk/atom <compute_chunk_atom>` command
defines its own group; atoms will have a chunk ID = 0 if they are not
in that group, signifying they are not assigned to a chunk, and will
thus also not contribute to this calculation.  You can specify the
"all" group for this command if you simply want to include atoms with
non-zero chunk IDs.

.. note::

   The coordinates of an atom contribute to the chunk's torque in
   "unwrapped" form, by using the image flags associated with each atom.
   See the :doc:`dump custom <dump>` command for a discussion of
   "unwrapped" coordinates.  See the Atoms section of the
   :doc:`read_data <read_data>` command for a discussion of image flags and
   how they are set for each atom.  You can reset the image flags
   (e.g., to 0) before invoking this compute by using the
   :doc:`set image <set>` command.

The simplest way to output the results of the compute torque/chunk
calculation to a file is to use the :doc:`fix ave/time <fix_ave_time>`
command, for example:

.. code-block:: LAMMPS

   compute cc1 all chunk/atom molecule
   compute myChunk all torque/chunk cc1
   fix 1 all ave/time 100 1 100 c_myChunk[*] file tmp.out mode vector

Output info
"""""""""""

This compute calculates a global array where the number of rows is equal to the
number of chunks *Nchunk* as calculated by the specified
:doc:`compute chunk/atom <compute_chunk_atom>` command.  The number of columns
is three for the :math:`x`, :math:`y`, and :math:`z` components of the torque
for each chunk.  These values can be accessed by any command that uses global
array values from a compute as input.
See the :doc:`Howto output <Howto_output>` doc page
for an overview of LAMMPS output options.

The array values are "intensive."  The array values will be in
force-distance :doc:`units <units>`.

Restrictions
""""""""""""
 none

Related commands
""""""""""""""""

:doc:`variable torque() function <variable>`

Default
"""""""

none
