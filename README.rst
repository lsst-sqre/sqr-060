.. image:: https://img.shields.io/badge/sqr--060-lsst.io-brightgreen.svg
   :target: https://sqr-060.lsst.io/
.. image:: https://github.com/lsst-sqre/sqr-060/workflows/CI/badge.svg
   :target: https://github.com/lsst-sqre/sqr-060/actions/

####################################################################################
Design of the Semaphore user broadcast message system for the Rubin Science Platform
####################################################################################

SQR-060
=======

Semaphore is a service that provides user messaging services for Rubin Science Platform (RSP). The initial launch of Semaphore focuses on broadcast-type messages that are shown to all RSP users. This system leverages GitHub as a content management system in conjunction with the Sempahore API service itself and client-facing UIs that are embedded in applications such as Squareone (science platform homepage). This technical note describes the data model and functionality of Semaphore broadcast messaging.

**Links:**

- Publication URL: https://sqr-060.lsst.io/
- Alternative editions: https://sqr-060.lsst.io/v
- GitHub repository: https://github.com/lsst-sqre/sqr-060
- Build system: https://github.com/lsst-sqre/sqr-060/actions/

Build this technical note
=========================

You can clone this repository and build the technote locally if your system has Python 3.11 or later:

.. code-block:: bash

   git clone https://github.com/lsst-sqre/sqr-060
   cd sqr-060
   make init
   make html

Repeat the ``make html`` command to rebuild the technote after making changes.
If you need to delete any intermediate files for a clean build, run ``make clean``.

The built technote is located at ``_build/html/index.html``.

Publishing changes to the web
=============================

This technote is published to https://sqr-060.lsst.io/ whenever you push changes to the ``main`` branch on GitHub.
When you push changes to a another branch, a preview of the technote is published to https://sqr-060.lsst.io/v.

Editing this technical note
===========================

The main content of this technote is in ``index.rst`` (a reStructuredText file).
Metadata and configuration is in the ``technote.toml`` file.
For guidance on creating content and information about specifying metadata and configuration, see the Documenteer documentation: https://documenteer.lsst.io/technotes.
