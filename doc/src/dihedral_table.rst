.. index:: dihedral\_style table

dihedral\_style table command
=============================

dihedral\_style table/omp command
=================================

Syntax
""""""


.. parsed-literal::

   dihedral_style table style Ntable

* style = *linear* or *spline* = method of interpolation
* Ntable = size of the internal lookup table

Examples
""""""""


.. parsed-literal::

   dihedral_style table spline 400
   dihedral_style table linear 1000
   dihedral_coeff 1 file.table DIH_TABLE1
   dihedral_coeff 2 file.table DIH_TABLE2

Description
"""""""""""

The *table* dihedral style creates interpolation tables of length
*Ntable* from dihedral potential and derivative values listed in a
file(s) as a function of the dihedral angle "phi".  The files are read
by the :doc:`dihedral\_coeff <dihedral_coeff>` command.

The interpolation tables are created by fitting cubic splines to the
file values and interpolating energy and derivative values at each of
*Ntable* dihedral angles. During a simulation, these tables are used
to interpolate energy and force values on individual atoms as
needed. The interpolation is done in one of 2 styles: *linear* or
*spline*\ .

For the *linear* style, the dihedral angle (phi) is used to find 2
surrounding table values from which an energy or its derivative is
computed by linear interpolation.

For the *spline* style, cubic spline coefficients are computed and
stored at each of the *Ntable* evenly-spaced values in the
interpolated table.  For a given dihedral angle (phi), the appropriate
coefficients are chosen from this list, and a cubic polynomial is used
to compute the energy and the derivative at this angle.

The following coefficients must be defined for each dihedral type via
the :doc:`dihedral\_coeff <dihedral_coeff>` command as in the example
above.

* filename
* keyword

The filename specifies a file containing tabulated energy and
derivative values. The keyword specifies a section of the file.  The
format of this file is described below.


----------


The format of a tabulated file is as follows (without the
parenthesized comments).  It can begin with one or more comment
or blank lines.


.. parsed-literal::

   # Table of the potential and its negative derivative

   DIH_TABLE1                   (keyword is the first text on line)
   N 30 DEGREES                 (N, NOF, DEGREES, RADIANS, CHECKU/F)
                                (blank line)
   1 -168.0 -1.40351172223 0.0423346818422
   2 -156.0 -1.70447981034 0.00811786522531
   3 -144.0 -1.62956100432 -0.0184129719987
   ...
   30 180.0 -0.707106781187 0.0719306095245

   # Example 2: table of the potential. Forces omitted

   DIH_TABLE2
   N 30 NOF CHECKU testU.dat CHECKF testF.dat

   1 -168.0 -1.40351172223
   2 -156.0 -1.70447981034
   3 -144.0 -1.62956100432
   ...
   30 180.0 -0.707106781187

A section begins with a non-blank line whose 1st character is not a
"#"; blank lines or lines starting with "#" can be used as comments
between sections. The first line begins with a keyword which
identifies the section. The line can contain additional text, but the
initial text must match the argument specified in the
:doc:`dihedral\_coeff <dihedral_coeff>` command. The next line lists (in
any order) one or more parameters for the table. Each parameter is a
keyword followed by one or more numeric values.

Following a blank line, the next N lines list the tabulated values. On
each line, the 1st value is the index from 1 to N, the 2nd value is
the angle value, the 3rd value is the energy (in energy units), and
the 4th is -dE/d(phi) also in energy units). The 3rd term is the
energy of the 4-atom configuration for the specified angle.  The 4th
term (when present) is the negative derivative of the energy with
respect to the angle (in degrees, or radians depending on whether the
user selected DEGREES or RADIANS).  Thus the units of the last term
are still energy, not force. The dihedral angle values must increase
from one line to the next.

Dihedral table splines are cyclic.  There is no discontinuity at 180
degrees (or at any other angle).  Although in the examples above, the
angles range from -180 to 180 degrees, in general, the first angle in
the list can have any value (positive, zero, or negative).  However
the *range* of angles represented in the table must be *strictly* less
than 360 degrees (2pi radians) to avoid angle overlap.  (You may not
supply entries in the table for both 180 and -180, for example.)  If
the user's table covers only a narrow range of dihedral angles,
strange numerical behavior can occur in the large remaining gap.

**Parameters:**

The parameter "N" is required and its value is the number of table
entries that follow. Note that this may be different than the N
specified in the :doc:`dihedral\_style table <dihedral_style>` command.
Let *Ntable* is the number of table entries requested dihedral\_style
command, and let *Nfile* be the parameter following "N" in the
tabulated file ("30" in the sparse example above).  What LAMMPS does
is a preliminary interpolation by creating splines using the *Nfile*
tabulated values as nodal points.  It uses these to interpolate as
needed to generate energy and derivative values at *Ntable* different
points (which are evenly spaced over a 360 degree range, even if the
angles in the file are not).  The resulting tables of length *Ntable*
are then used as described above, when computing energy and force for
individual dihedral angles and their atoms.  This means that if you
want the interpolation tables of length *Ntable* to match exactly what
is in the tabulated file (with effectively nopreliminary
interpolation), you should set *Ntable* = *Nfile*\ .  To insure the
nodal points in the user's file are aligned with the interpolated
table entries, the angles in the table should be integer multiples of
360/\ *Ntable* degrees, or 2\*PI/\ *Ntable* radians (depending on your
choice of angle units).

The optional "NOF" keyword allows the user to omit the forces
(negative energy derivatives) from the table file (normally located in
the 4th column).  In their place, forces will be calculated
automatically by differentiating the potential energy function
indicated by the 3rd column of the table (using either linear or
spline interpolation).

The optional "DEGREES" keyword allows the user to specify angles in
degrees instead of radians (default).

The optional "RADIANS" keyword allows the user to specify angles in
radians instead of degrees.  (Note: This changes the way the forces
are scaled in the 4th column of the data file.)

The optional "CHECKU" keyword is followed by a filename.  This allows
the user to save all of the *Ntable* different entries in the
interpolated energy table to a file to make sure that the interpolated
function agrees with the user's expectations.  (Note: You can
temporarily increase the *Ntable* parameter to a high value for this
purpose.  "\ *Ntable*\ " is explained above.)

The optional "CHECKF" keyword is analogous to the "CHECKU" keyword.
It is followed by a filename, and it allows the user to check the
interpolated force table.  This option is available even if the user
selected the "NOF" option.

Note that one file can contain many sections, each with a tabulated
potential. LAMMPS reads the file section by section until it finds one
that matches the specified keyword.


----------


Styles with a *gpu*\ , *intel*\ , *kk*\ , *omp*\ , or *opt* suffix are
functionally the same as the corresponding style without the suffix.
They have been optimized to run faster, depending on your available
hardware, as discussed on the :doc:`Speed packages <Speed_packages>` doc
page.  The accelerated styles take the same arguments and should
produce the same results, except for round-off and precision issues.

These accelerated styles are part of the GPU, USER-INTEL, KOKKOS,
USER-OMP and OPT packages, respectively.  They are only enabled if
LAMMPS was built with those packages.  See the :doc:`Build package <Build_package>` doc page for more info.

You can specify the accelerated styles explicitly in your input script
by including their suffix, or you can use the :doc:`-suffix command-line switch <Run_options>` when you invoke LAMMPS, or you can use the
:doc:`suffix <suffix>` command in your input script.

See the :doc:`Speed packages <Speed_packages>` doc page for more
instructions on how to use the accelerated styles effectively.

**Restart info:**

This dihedral style writes the settings for the "dihedral\_style table"
command to :doc:`binary restart files <restart>`, so a dihedral\_style
command does not need to specified in an input script that reads a
restart file.  However, the coefficient information is not stored in
the restart file, since it is tabulated in the potential files.  Thus,
dihedral\_coeff commands do need to be specified in the restart input
script.

Restrictions
""""""""""""


This dihedral style can only be used if LAMMPS was built with the
USER-MISC package.  See the :doc:`Build package <Build_package>` doc
page for more info.

Related commands
""""""""""""""""

:doc:`dihedral\_coeff <dihedral_coeff>`

**Default:** none


.. _lws: http://lammps.sandia.gov
.. _ld: Manual.html
.. _lc: Commands_all.html
