.. |predefinition| replace:: sub defined above content.

#############
Substitutions
#############

This page contains reStructuredText substitutions that are used in the example content.

An example outside: |name|.

.. example:: Internally defined substitution
   :tags: substitutions

   .. |preinternal| replace:: sub defined at top of directive

   |preinternal|

   |postinternal|

   This substitution is defined outside the scope of the example: |ap|.

   |predefinition|

   .. |postinternal| replace:: sub defined at bottom of directive

.. |ap| replace:: `The Astropy Project <http://astropy.org>`__

.. |name| replace:: replacement *text*

More content after the substitution definitions.

Hello world!
