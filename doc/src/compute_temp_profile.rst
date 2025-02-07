.. index:: compute temp/profile

compute temp/profile command
============================

Syntax
""""""

.. code-block:: LAMMPS

   compute ID group-ID temp/profile xflag yflag zflag binstyle args

* ID, group-ID are documented in :doc:`compute <compute>` command
* temp/profile = style name of this compute command
* xflag,yflag,zflag = 0/1 for whether to exclude/include this dimension
* binstyle = *x* or *y* or *z* or *xy* or *yz* or *xz* or *xyz*

  .. parsed-literal::

       *x* arg = Nx
       *y* arg = Ny
       *z* arg = Nz
       *xy* args = Nx Ny
       *yz* args = Ny Nz
       *xz* args = Nx Nz
       *xyz* args = Nx Ny Nz
         Nx, Ny, Nz = number of velocity bins in *x*, *y*, *z* dimensions

* zero or more keyword/value pairs may be appended
* keyword = *out*

  .. parsed-literal::

       *out* value = *tensor* or *bin*

Examples
""""""""

.. code-block:: LAMMPS

   compute myTemp flow temp/profile 1 1 1 x 10
   compute myTemp flow temp/profile 1 1 1 x 10 out bin
   compute myTemp flow temp/profile 0 1 1 xyz 20 20 20

Description
"""""""""""

Define a computation that calculates the temperature of a group of
atoms, after subtracting out a spatially-averaged center-of-mass
velocity field, before computing the kinetic energy.  This can be
useful for thermostatting a collection of atoms undergoing a complex
flow (e.g. via a profile-unbiased thermostat (PUT) as described in
:ref:`(Evans) <Evans1>`).  A compute of this style can be used by any command
that computes a temperature (e.g. :doc:`thermo_modify <thermo_modify>`,
:doc:`fix temp/rescale <fix_temp_rescale>`, :doc:`fix npt <fix_nh>`).

The *xflag*, *yflag*, *zflag* settings determine which components of
average velocity are subtracted out.

The *binstyle* setting and its *Nx*, *Ny*, *Nz* arguments determine how bins
are setup to perform spatial averaging.  "Bins" can be 1d slabs, 2d pencils,
or 3d bricks depending on which *binstyle* is used.  The simulation box is
partitioned conceptually into *Nx* :math:`\times` *Ny* :math:`\times` *Nz*
bins.  Depending on the *binstyle*, you may only specify one or two of these
values; the others are effectively set to 1 (no binning in that dimension).
For non-orthogonal (triclinic) simulation boxes, the bins are "tilted" slabs or
pencils or bricks that are parallel to the tilted faces of the box.  See the
:doc:`region prism <region>` command for a discussion of the geometry of tilted
boxes in LAMMPS.

When a temperature is computed, the center-of-mass velocity for the
set of atoms that are both in the compute group and in the same
spatial bin is calculated.  This bias velocity is then subtracted from
the velocities of individual atoms in the bin to yield a thermal
velocity for each atom.  Note that if there is only one atom in the
bin, its thermal velocity will thus be 0.0.

After the spatially-averaged velocity field has been subtracted from
each atom, the temperature is calculated by the formula

.. math::

  \text{KE} = \left( \frac{\text{dim}}{N} - N_s N_x N_y N_z
                        - \text{extra} \right) \frac{k_B T}{2},

where KE is the total kinetic energy of the group of atoms (sum of
:math:`\frac12 m v^2`; dim = 2 or 3 is the dimensionality of the simulation;
:math:`N_s =` 0, 1, 2, or 3 for streaming velocity subtracted in 0, 1, 2, or 3
dimensions, respectively; *extra* is the number of  extra degrees of freedom;
*N* is the number of atoms in the group; :math:`k_B` is the Boltzmann constant,
and :math:`T` is the absolute temperature.  The :math:`N_s N_x N_y N_z` term is
the number of degrees of freedom subtracted to adjust for the removal of the
center-of-mass velocity in each direction of the *Nx\*Ny\*Nz* bins, as
discussed in the :ref:`(Evans) <Evans1>` paper.  The extra term defaults to
:math:`\text{dim} - N_s` and accounts for overall conservation of
center-of-mass velocity across the group in directions where streaming velocity
is *not* subtracted. This can be altered using the *extra* option of the
:doc:`compute_modify <compute_modify>` command.

If the *out* keyword is used with a *tensor* value, which is the default,
a kinetic energy tensor, stored as a six-element vector, is also calculated by
this compute for use in the computation of a pressure tensor.  The formula for
the components of the tensor is the same as the above formula, except that
:math:`v^2` is replaced by :math:`v_x v_y` for the :math:`xy` component, and
so on.  The six components of the vector are ordered :math:`xx`, :math:`yy`,
:math:`zz`, :math:`xy`, :math:`xz`, :math:`yz`.

If the *out* keyword is used with a *bin* value, the count of atoms and
computed temperature for each bin are stored for output, as an array of values,
as described below.  The temperature of each bin is calculated as described
above, where the bias velocity is subtracted and only the remaining thermal
velocity of atoms in the bin contributes to the temperature.  See the note
below for how the temperature is normalized by the degrees-of-freedom of atoms
in the bin.

The number of atoms contributing to the temperature is assumed to be
constant for the duration of the run; use the *dynamic* option of the
:doc:`compute_modify <compute_modify>` command if this is not the case.

The removal of the spatially-averaged velocity field by this fix is
essentially computing the temperature after a "bias" has been removed
from the velocity of the atoms.  If this compute is used with a fix
command that performs thermostatting then this bias will be subtracted
from each atom, thermostatting of the remaining thermal velocity will
be performed, and the bias will be added back in.  Thermostatting
fixes that work in this way include :doc:`fix nvt <fix_nh>`,
:doc:`fix temp/rescale <fix_temp_rescale>`,
:doc:`fix temp/berendsen <fix_temp_berendsen>`,
and :doc:`fix langevin <fix_langevin>`.

This compute subtracts out degrees-of-freedom due to fixes that constrain
molecular motion, such as :doc:`fix shake <fix_shake>` and
:doc:`fix rigid <fix_rigid>`.  This means the temperature of groups of atoms
that include these constraints will be computed correctly.  If needed, the
subtracted degrees-of-freedom can be altered using the *extra* option of the
:doc:`compute_modify <compute_modify>` command.

.. note::

   When using the *out* keyword with a value of *bin*, the
   calculated temperature for each bin includes the degrees-of-freedom
   adjustment described in the preceding paragraph for fixes that
   constrain molecular motion, as well as the adjustment due to
   the *extra* option (which defaults to *dim* - *Ns* as described above),
   by fractionally applying them based on the fraction of atoms in each
   bin. As a result, the bin degrees-of-freedom summed over all bins exactly
   equals the degrees-of-freedom used in the scalar temperature calculation,
   :math:`\Sigma N_{\text{DOF}_i} = N_\text{DOF}` and the corresponding
   relation for temperature is also satisfied
   (:math:`\Sigma N_{\text{DOF}_i} T_i = N_\text{DOF} T`).
   These relations will break down in cases for which the adjustment
   exceeds the actual number of degrees of freedom in a bin. This could happen
   if a bin is empty or in situations in which rigid molecules
   are non-uniformly distributed, in which case the reported
   temperature within a bin may not be accurate.

See the :doc:`Howto thermostat <Howto_thermostat>` page for a
discussion of different ways to compute temperature and perform
thermostatting.  Using this compute in conjunction with a
thermostatting fix, as explained there, will effectively implement a
profile-unbiased thermostat (PUT), as described in :ref:`(Evans) <Evans1>`.

Output info
"""""""""""

This compute calculates a global scalar (the temperature).  Depending
on the setting of the *out* keyword, it also calculates a global
vector or array.  For *out* = *tensor*, it calculates a vector of
length 6 (KE tensor), which can be accessed by indices 1--6.  For *out*
= *bin* it calculates a global array which has 2 columns and :math:`N` rows,
where :math:`N` is the number of bins.  The first column contains the number
of atoms in that bin.  The second contains the temperature of that
bin, calculated as described above.  The ordering of rows in the array
is as follows.  Bins in :math:`x` vary fastest, then :math:`y`, then
:math:`z`.  Thus for a :math:`10\times 10\times 10` 3d array of bins, there
will be 1000 rows.  The bin with indices :math:`(i_x,i_y,i_z) = (2,3,4)` would
map to row :math:`M = 10^2(i_z-1)  + 10(i_y-1) + i_x = 322`, where the rows are
numbered from 1 to 1000 and the bin indices are numbered from 1 to 10 in each
dimension.

These values can be used by any command that uses global scalar or
vector or array values from a compute as input.  See the
:doc:`Howto output <Howto_output>` page for an overview of LAMMPS output
options.

The scalar value calculated by this compute is "intensive."  The
vector values are "extensive."  The array values are "intensive."

The scalar value will be in temperature :doc:`units <units>`.  The
vector values will be in energy :doc:`units <units>`.  The first column
of array values are counts; the values in the second column will be in
temperature :doc:`units <units>`.

Restrictions
""""""""""""

You should not use too large a velocity-binning grid, especially in
3d.  In the current implementation, the binned velocity averages are
summed across all processors, so this will be inefficient if the grid
is too large, and the operation is performed every timestep, as it
will be for most thermostats.

Related commands
""""""""""""""""

:doc:`compute temp <compute_temp>`, :doc:`compute temp/ramp <compute_temp_ramp>`, :doc:`compute temp/deform <compute_temp_deform>`, :doc:`compute pressure <compute_pressure>`

Default
"""""""

The option default is out = tensor.

----------

.. _Evans1:

**(Evans)** Evans and Morriss, Phys Rev Lett, 56, 2172-2175 (1986).
