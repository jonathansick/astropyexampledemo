*******************************************************
Astronomical Coordinate Systems (`astropy.coordinates`)
*******************************************************

Introduction
============

The `~astropy.coordinates` package provides classes for representing a variety
of celestial/spatial coordinates and their velocity components, as well as tools
for converting between common coordinate systems in a uniform way.

Getting Started
===============

.. example:: Define a coordinate in Right Ascension and Declination
   :tags: units, coordinates

   The best way to start using ``astropy.coordinates`` is to use the
   ``SkyCoord`` class. ``SkyCoord`` objects are instantiated by passing in
   positions (and optional velocities) with specified units and a coordinate
   frame. Sky positions are commonly passed in as ``Quantity`` objects and the
   frame is specified with the string name. As an example of creating
   a ``Skycoord`` to represent an ICRS (Right ascension [RA], Declination
   [Dec]) sky position::
   
       >>> from astropy import units as u
       >>> from astropy.coordinates import SkyCoord
       >>> c = SkyCoord(ra=10.625*u.degree, dec=41.2*u.degree, frame='icrs')
   
   The initializer for ``Skycoord`` is very flexible and supports inputs provided in
   a number of convenient formats. The following ways of initializing a coordinate
   are all equivalent to the above::
   
       >>> c = SkyCoord(10.625, 41.2, frame='icrs', unit='deg')
       >>> c = SkyCoord('00h42m30s', '+41d12m00s', frame='icrs')
       >>> c = SkyCoord('00h42.5m', '+41d12m')
       >>> c = SkyCoord('00 42 30 +41 12 00', unit=(u.hourangle, u.deg))
       >>> c = SkyCoord('00:42.5 +41:12', unit=(u.hourangle, u.deg))
       >>> c  # doctest: +FLOAT_CMP
       <SkyCoord (ICRS): (ra, dec) in deg
           (10.625, 41.2)>

The examples above illustrate a few rules to follow when creating a
coordinate object:

- Coordinate values can be provided either as unnamed positional arguments or
  via keyword arguments like ``ra`` and ``dec``, or  ``l`` and ``b`` (depending
  on the frame).
- The coordinate ``frame`` keyword is optional because it defaults to
  `~astropy.coordinates.ICRS`.
- Angle units must be specified for all components, either by passing in a
  `~astropy.units.Quantity` object (e.g., ``10.5*u.degree``), by including them
  in the value (e.g., ``'+41d12m00s'``), or via the ``unit`` keyword.

|skycoord| and all other `~astropy.coordinates` objects also support
array coordinates. These work in the same way as single-value coordinates, but
they store multiple coordinates in a single object. When you are going
to apply the same operation to many different coordinates (say, from a
catalog), this is a better choice than a list of |skycoord| objects,
because it will be *much* faster than applying the operation to each
|skycoord| in a ``for`` loop. Like the underlying `~numpy.ndarray` instances
that contain the data, |skycoord| objects can be sliced, reshaped, etc.::

    >>> c = SkyCoord(ra=[10, 11, 12, 13]*u.degree, dec=[41, -5, 42, 0]*u.degree)
    >>> c  # doctest: +FLOAT_CMP
    <SkyCoord (ICRS): (ra, dec) in deg
        [(10., 41.), (11., -5.), (12., 42.), (13.,  0.)]>
    >>> c[1]  # doctest: +FLOAT_CMP
    <SkyCoord (ICRS): (ra, dec) in deg
        (11., -5.)>
    >>> c.reshape(2, 2)  # doctest: +FLOAT_CMP
    <SkyCoord (ICRS): (ra, dec) in deg
        [[(10., 41.), (11., -5.)],
         [(12., 42.), (13.,  0.)]]>

Coordinate Access
-----------------

Once you have a coordinate object you can access the components of that
coordinate (e.g., RA, Dec) to get string representations of the full
coordinate.

The component values are accessed using (typically lowercase) named attributes
that depend on the coordinate frame (e.g., ICRS, Galactic, etc.). For the
default, ICRS, the coordinate component names are ``ra`` and ``dec``::

    >>> c = SkyCoord(ra=10.68458*u.degree, dec=41.26917*u.degree)
    >>> c.ra  # doctest: +FLOAT_CMP
    <Longitude 10.68458 deg>
    >>> c.ra.hour  # doctest: +FLOAT_CMP
    0.7123053333333335
    >>> c.ra.hms  # doctest: +FLOAT_CMP
    hms_tuple(h=0.0, m=42.0, s=44.299200000000525)
    >>> c.dec  # doctest: +FLOAT_CMP
    <Latitude 41.26917 deg>
    >>> c.dec.degree  # doctest: +FLOAT_CMP
    41.26917
    >>> c.dec.radian  # doctest: +FLOAT_CMP
    0.7202828960652683

Coordinates can be converted to strings using the
:meth:`~astropy.coordinates.SkyCoord.to_string` method::

    >>> c = SkyCoord(ra=10.68458*u.degree, dec=41.26917*u.degree)
    >>> c.to_string('decimal')
    '10.6846 41.2692'
    >>> c.to_string('dms')
    '10d41m04.488s 41d16m09.012s'
    >>> c.to_string('hmsdms')
    '00h42m44.2992s +41d16m09.012s'

Transformation
--------------

One convenient way to transform to a new coordinate frame is by accessing
the appropriately named attribute. For instance, to get the coordinate in
the `~astropy.coordinates.Galactic` frame use::

    >>> c_icrs = SkyCoord(ra=10.68458*u.degree, dec=41.26917*u.degree, frame='icrs')
    >>> c_icrs.galactic  # doctest: +FLOAT_CMP
    <SkyCoord (Galactic): (l, b) in deg
        (121.17424181, -21.57288557)>

For more control, you can use the `~astropy.coordinates.SkyCoord.transform_to`
method, which accepts a frame name, frame class, or frame instance::

    >>> c_fk5 = c_icrs.transform_to('fk5')  # c_icrs.fk5 does the same thing
    >>> c_fk5  # doctest: +FLOAT_CMP
    <SkyCoord (FK5: equinox=J2000.000): (ra, dec) in deg
        (10.68459154, 41.26917146)>

    >>> from astropy.coordinates import FK5
    >>> c_fk5.transform_to(FK5(equinox='J1975'))  # precess to a different equinox  # doctest: +FLOAT_CMP
    <SkyCoord (FK5: equinox=J1975.000): (ra, dec) in deg
        (10.34209135, 41.13232112)>

This form of `~astropy.coordinates.SkyCoord.transform_to` also makes it
possible to convert from celestial coordinates to
`~astropy.coordinates.AltAz` coordinates, allowing the use of |skycoord|
as a tool for planning observations.

Some coordinate frames such as `~astropy.coordinates.AltAz` require Earth
rotation information (UT1-UTC offset and/or polar motion) when transforming
to/from other frames. These Earth rotation values are automatically downloaded
from the International Earth Rotation and Reference Systems (IERS) service when
required.

Representation
--------------

So far we have been using a spherical coordinate representation in all of our
examples, and this is the default for the built-in frames. Frequently it is
convenient to initialize or work with a coordinate using a different
representation such as Cartesian or Cylindrical. This can be done by setting
the ``representation_type`` for either |skycoord| objects or low-level frame
coordinate objects::

    >>> c = SkyCoord(x=1, y=2, z=3, unit='kpc', representation_type='cartesian')
    >>> c  # doctest: +FLOAT_CMP
    <SkyCoord (ICRS): (x, y, z) in kpc
        (1., 2., 3.)>
    >>> c.x, c.y, c.z  # doctest: +FLOAT_CMP
    (<Quantity 1. kpc>, <Quantity 2. kpc>, <Quantity 3. kpc>)

    >>> c.representation_type = 'cylindrical'
    >>> c  # doctest: +FLOAT_CMP
    <SkyCoord (ICRS): (rho, phi, z) in (kpc, deg, kpc)
        (2.23606798, 63.43494882, 3.)>

Distance
--------

|skycoord| and the individual frame classes also support specifying a distance
from the frame origin. The origin depends on the particular coordinate frame;
this can be, for example, centered on the earth, centered on the solar system
barycenter, etc. Two angles and a distance specify a unique point in 3D space,
which also allows converting the coordinates to a Cartesian representation::

    >>> c = SkyCoord(ra=10.68458*u.degree, dec=41.26917*u.degree, distance=770*u.kpc)
    >>> c.cartesian.x  # doctest: +FLOAT_CMP
    <Quantity 568.71286542 kpc>
    >>> c.cartesian.y  # doctest: +FLOAT_CMP
    <Quantity 107.3008974 kpc>
    >>> c.cartesian.z  # doctest: +FLOAT_CMP
    <Quantity 507.88994292 kpc>

With distances assigned, |skycoord| convenience methods are more powerful, as
they can make use of the 3D information. For example, to compute the physical,
3D separation between two points in space::

    >>> c1 = SkyCoord(ra=10*u.degree, dec=9*u.degree, distance=10*u.pc, frame='icrs')
    >>> c2 = SkyCoord(ra=11*u.degree, dec=10*u.degree, distance=11.5*u.pc, frame='icrs')
    >>> c1.separation_3d(c2)  # doctest: +FLOAT_CMP
    <Distance 1.52286024 pc>

Convenience Methods
-------------------

|skycoord| defines a number of convenience methods that support, for example,
computing on-sky (i.e., angular) and 3D separations between two coordinates::

    >>> c1 = SkyCoord(ra=10*u.degree, dec=9*u.degree, frame='icrs')
    >>> c2 = SkyCoord(ra=11*u.degree, dec=10*u.degree, frame='fk5')
    >>> c1.separation(c2)  # Differing frames handled correctly  # doctest: +FLOAT_CMP
    <Angle 1.40453359 deg>

Or cross-matching catalog coordinates::

    >>> target_c = SkyCoord(ra=10*u.degree, dec=9*u.degree, frame='icrs')
    >>> # read in coordinates from a catalog...
    >>> catalog_c = ... # doctest: +SKIP
    >>> idx, sep, _ = target_c.match_to_catalog_sky(catalog_c) # doctest: +SKIP

The `astropy.coordinates` sub-package also provides a quick way to get
coordinates for named objects, assuming you have an active internet
connection. The `~astropy.coordinates.SkyCoord.from_name` method of |skycoord|
uses `Sesame <http://cds.u-strasbg.fr/cgi-bin/Sesame>`_ to retrieve coordinates
for a particular named object::

    >>> SkyCoord.from_name("PSR J1012+5307")  # doctest: +REMOTE_DATA +FLOAT_CMP
    <SkyCoord (ICRS): (ra, dec) in deg
        (153.1393271, 53.117343)>

In some cases, the coordinates are embedded in the catalog name of the object.
For such object names, `~astropy.coordinates.SkyCoord.from_name` is able
to parse the coordinates from the name if given the ``parse=True`` option.
For slow connections, this may be much faster than a sesame query for the same
object name. It's worth noting, however, that the coordinates extracted in this
way may differ from the database coordinates by a few deci-arcseconds, so only
use this option if you do not need sub-arcsecond accuracy for your coordinates::

    >>> SkyCoord.from_name("CRTS SSS100805 J194428-420209", parse=True)  # doctest: +FLOAT_CMP
    <SkyCoord (ICRS): (ra, dec) in deg
        (296.11666667, -42.03583333)>


For sites (primarily observatories) on the Earth, `astropy.coordinates` provides
a quick way to get an `~astropy.coordinates.EarthLocation` - the
`~astropy.coordinates.EarthLocation.of_site` method::

    >>> from astropy.coordinates import EarthLocation
    >>> EarthLocation.of_site('Apache Point Observatory')  # doctest: +REMOTE_DATA +FLOAT_CMP
    <EarthLocation (-1463969.30185172, -5166673.34223433,  3434985.71204565) m>

To see the list of site names available, use
:func:`astropy.coordinates.EarthLocation.get_site_names`.

For arbitrary Earth addresses (e.g., not observatory sites), use the
`~astropy.coordinates.EarthLocation.of_address` classmethod. Any address passed
to this function uses Google maps to retrieve the latitude and longitude and can
also (optionally) query Google maps to get the height of the location. As with
Google maps, this works with fully specified addresses, location names, city
names, etc.:

.. doctest-skip::

    >>> EarthLocation.of_address('1002 Holy Grail Court, St. Louis, MO')
    <EarthLocation (-26726.98216371, -4997009.8604809, 3950271.16507911) m>
    >>> EarthLocation.of_address('1002 Holy Grail Court, St. Louis, MO',
    ...                          get_height=True)
    <EarthLocation (-26727.6272786, -4997130.47437768, 3950367.15622108) m>
    >>> EarthLocation.of_address('Danbury, CT')
    <EarthLocation ( 1364606.64511651, -4593292.9428273,  4195415.93695139) m>

This functionality can be combined to do more complicated tasks like computing
barycentric corrections to radial velocity observations (also a supported
high-level |skycoord| method::

    >>> from astropy.time import Time
    >>> obstime = Time('2017-2-14')
    >>> target = SkyCoord.from_name('M31')  # doctest: +REMOTE_DATA
    >>> keck = EarthLocation.of_site('Keck')  # doctest: +REMOTE_DATA
    >>> target.radial_velocity_correction(obstime=obstime, location=keck).to('km/s')  # doctest: +REMOTE_DATA +FLOAT_CMP
    <Quantity -22.359784554780255 km / s>

Velocities (Proper Motions and Radial Velocities)
-------------------------------------------------

In addition to positional coordinates, `~astropy.coordinates` supports storing
and transforming velocities::

    >>> sc = SkyCoord(1*u.deg, 2*u.deg, radial_velocity=20*u.km/u.s)
    >>> sc  # doctest: +SKIP
    <SkyCoord (ICRS): (ra, dec) in deg
        ( 1.,  2.)
     (radial_velocity) in km / s
        ( 20.,)>

.. the SKIP above in the ``sc`` line is because NumPy has a subtly different output in versions < 12 - the trailing comma is missing. If a NPY_LT_1_12 comes in to being this can switch to that. But don't forget to *also* change this in the velocities.rst file.

.. |skycoord| replace:: `~astropy.coordinates.SkyCoord`
