####################################################################################
Design of the Semaphore user broadcast message system for the Rubin Science Platform
####################################################################################

.. abstract::

   Semaphore is a service that provides user messaging services for Rubin Science Platform (RSP).
   The initial launch of Semaphore focuses on broadcast-type messages that are shown to all RSP users.
   This system leverages GitHub as a content management system in conjunction with the Sempahore API service itself and client-facing UIs that are embedded in applications such as Squareone_ (the science platform homepage).
   This technical note describes the data model and functionality of Semaphore broadcast messaging.

.. _intro:

Introduction to Semaphore
=========================

Semaphore is the centralized service for messaging users on the |RSP| through multiple appropriate channels, including:

- embedded |RSP| user interfaces
- email
- Community forum

Our vision is for Semaphore to support two categories of messages:

1. **Broadcast messages,** that apply to all users and are visible in a specific time window
2. **Direct messages,** that are written for specific users in response to user activity

At launch, Semaphore focuses entirely on the broadcast message use-case, with direct messaging support being added later.

The broadcast message use-case is vitally important during |DP01|. :cite:`RTN-004`
During development, we have relied on Slack to notify users about maintenance events and system outages.
Since |DP01| users won't be serviced through Slack :cite:`RTN-006`, we need to leverage user-facing channels to begin communicating this information.
While we could directly use the `Community forum`_, we don't believe that the forum is the right channel for providing timely information to users.
Instead, messages about things like maintenance events need to be served to users where they are — namely in the |RSP| user interfaces, such as the Squareone_ homepage, the Notebook Aspect, and the Portal Aspect.
This use case implies the need for a small web service, embedded in the |RSP| deployment, that can serve messages for display on |RSP| user interfaces.
This service is Semaphore.

.. _stack:

Technical stack
===============

Semaphore, with respect to the initial use case of broadcast messaging, consists of these technical components:

- GitHub repository serving as a content management system (CMS)
- Semaphore service layer, providing an API
- UI components embedded in Squareone_, and possibly other |RSP| applications, to consume the Semaphore API and display messages to the user

GitHub makes sense as a CMS because it provides both a user interface and an authorization system (including pull request reviews) that the team is already familiar with.
Further, the cadence of broadcast messages is slow enough that a GitHub-based process of committing content changes and merging pull requests is reasonably efficient.
We anticipate that a GitHub repository could be dedicated to |RSP| broadcast messages (or the Phalanx deployment repository could be co-opted for broadcast messages).
Messages are individual files in a plain text format that support metadata.
Two possible formats are YAML or Markdown with YAML-formatted front-matter.
A pure YAML structure optimizes for machine readability, but is slightly inconvenient for writing multi-paragraph messages because those paragraphs must be indented in a YAML field.
The markdown option optimizes writing structured messages, while also providing machine-readable metadata.
This is how posts in static blogging engines such as Jekyll are written, for example, and we believe that the broadcast message use-case roughly follows that pattern.
For the initial launch, we opt for the Markdown with YAML front-matter option.
Later sections of this document develop the data model for these message files.

Semaphore is implemented as a FastAPI application that runs within each |RSP| environment.
With respect to the global broadcast use-case, Semaphore consumes the GitHub content repository, sorts messages based on environmental tags or timing factors encoded in the message's metadata, and provides those messages through a web API to clients such as the Squareone_ homepage, Notebook Aspect, or Portal Aspect.
The default API format is REST, though we believe that a GraphQL API that implements a subscription query would be the easiest for client applications to consume and provide a real-time experience to users.

React-based client web applications consume the Semaphore GraphQL using the `Apollo React Client`_.

.. _ui:

User interface design considerations
====================================

How messages are displayed to users drives the design of the message data model.

The most basic visual presentation of a broadcast message is as a banner across the Squareone_ page.
The most important part of this banner is the summary sentence: a quickly-digestible message.
Making the default message presentation be a summary sentence also ensures that broadcast messages can fit into a variety of user interfaces, such as the JupyterLab status bar, without disrupting the overall user experience of that application.

Some message may require more explanation, including paragraphs, lists, links, and so on.
The user interface can accommodate this information by including a disclosure affordance on the banner that either opens a modal with the message body or expands the banner to reveal the message body.
Both of these approaches avoid taking the user away from the UI (and thus the work) that they are currently doing.
On a technical level, this message body is written as markdown, providing the authors all the tools needed to link elsewhere or provide a formatted message.

All broadcast messages are time-sensitive, on some level, and both the visual presentation and message data model need to be built around that.
First, "old" broadcast messages should never be shown.
Thus all messages should have expiration dates encoded in their metadata.
Second, a message may not yet be relevant; thus messages should be able to provide defer dates that schedule their appearance.
Both of these considerations ensure that broadcast messages are relevant to users.

A common use-case for broadcast-style messages is to pre-announce an event.
Messages should provide an event date in a structured format so that the client can display the date and time in the user's timezone, and also so that this date information can be displayed both prominently and consistently alongside the message's summary.
For events that are about to happen imminently, the client could also opt to display this event timestamp as a countdown clock (for example, 10 minutes until maintenance).

.. _data-model:

Message data model
==================

This section describes the data model for messages.
As discussed in :ref:`stack`, each message is a plain-text file committed into a GitHub repository.

Basic message
-------------

A basic message is a markdown file with YAML front-matter.
The ``summary`` field is the text that is shown persistently in the message banner.
The optional message body can be shown when a user clicks on a message to view additional information.
Both the summary and body are encoded in markdown so that the author can include basic formatting.
The summary should only include inline formatting (such as making text bold or monospace, or including links), which the body can also include block-elements such as lists or code blocks.

.. code-block:: text

   ---
   summary: The markdown-formatted broadcast message.
   ---

   The extended message body, shown *only* when the user
   interacts with the message, and formatted as markdown.

Tagging a science platform environment
--------------------------------------

Some messages should only in a single |RSP| environment, or a select group of environments.
These environments can be specified as a comma-separated list of Phalanx environment names.
The Semaphore service is configured with the environment it runs it, so only messages that are untagged, or tagged with that environment name are broadcast within that environment:

.. code-block:: text

   ---
   summary: The markdown-formatted broadcast message.
   env: idfprod,base
   ---

   The extended message body, shown *only* when the user
   interacts with the message, and formatted as markdown.

Deferring and expiring a message
--------------------------------

This example features the ``defer`` and ``expire`` fields:

.. code-block:: text

   ---
   summary: The markdown-formatted broadcast message.
   env: idfprod
   defer: 2021-01-01:00:00:00
   expire: 2021-01-02:00:00:00
   ---

   The extended message body, shown *only* when the user
   interacts with the message, and formatted as markdown.

The ``defer`` field is the time when the message becomes available, while ``expire`` specifies when the message is no longer available (see also :ref:`message-ttl`).
This feature allows message authors to pre-schedule a message without having to interact with the GitHub repository in real-time.
See :ref:`human-dates` for the formatting of these timestamps.

.. _message-ttl:

Expiring a message with time-to-live
------------------------------------

The default approach to expiring a message is with an ``expire`` field.
An additional, and alternative approach, is to replace the ``expire`` field with a ``ttl`` field, which is a duration for the message to be broadcast *after* the ``defer`` timestamp:

.. code-block:: text

   ---
   summary: The markdown-formatted broadcast message.
   env: idfprod
   defer: 2021-01-01:00:00:00
   ttl: 2h
   ---

   The extended message body, shown *only* when the user
   interacts with the message, and formatted as markdown.

Repeating messages
------------------

Some messages may need to repeat.
A common use case is a weekly system maintenance window.
Cron_ is likely the best syntax for describing periodic events.
This this scenario, ``cron`` would replace the ``defer`` field, and ``ttl`` would express the duration a message is broadcast after each cron event:

.. code-block:: text

   ---
   summary: The plain-text broadcast message.
   env: idfprod
   cron: 0 13 * * THU
   timezone: -7:00
   ttl: 2h
   ---

   The extended message body, shown *only* when the user
   interacts with the message, and formatted as markdown.

This example also demonstrates the application of a ``timezone`` field that provides time zone context to the ``cron`` field (potentially the ``timezone`` field could also augment the ``defer`` and ``expire`` fields if they do not have an explicit timezone; see also :ref:`timezones`).

.. note::

   pycron_ provides a simple API for parsing cron events and whether a cron event is active on the basis of the ttl message.

.. _human-dates:

Human-writeable dates, durations and timezones
==============================================

The timestamp (``defer``, ``expire``), duration (``ttl``) and timezone (``timezone``) fields are parsed with the arrow_ Python package so that "humanized" dates and durations can be parsed, in addition to structured dates (ISO 8601).
As such, arrow_ is the standard for determining if timestamp, duration, or timezone field is well-formatted.
Semaphore provides a linting facility to give an author feedback before merging a pull request with messages on GitHub.

.. _timezones:

Timezones
=========

Semaphore internally stores all timestamps as UTC.
Likewise, Semaphore APIs serve timestamps as UTC with the expectation that clients can convert those timestamps either to relative dates or localized times as needed.

By default, timestamps and cron events in broadcast messages are assumed to be UTC.
Timezone information can be included with the ``defer`` and ``expire`` messages with ISO 8601 formatting.
Alternatively the timezone can be specified for a message with a ``timezone`` field, which is :ref:`parsed by arrow <human-dates>`.
Finally, the root directory of the GitHub repository containing messages can include a configuration file named ``.semaphore.yaml`` includes a ``timezone`` field.
This timezone applies to all messages unless they contain more specific timezone information.

References
==========

.. bibliography::


.. |RSP| replace:: Rubin Science Platform
.. |DP01| replace:: :abbr:`DP0.1 (Data Preview 0.1)`

.. _Community forum: https://community.lsst.org
.. _Squareone: https://squareone.lsst.io
.. _Apollo React Client: https://www.apollographql.com/docs/react/
.. _Cron: https://en.wikipedia.org/wiki/Cron
.. _pycron: https://github.com/kipe/pycron
.. _arrow: https://arrow.readthedocs.io/en/latest/
