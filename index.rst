:tocdepth: 1

.. sectnum::

Summary
=======

Semaphore is a service that provides user messaging services for Rubin Science Platform (RSP).
The initial launch of Semaphore focuses on broadcast-type messages that are shown to all RSP users.
This system leverages GitHub as a content management system in conjunction with the Sempahore API service itself and client-facing UIs that are embedded in applications such as Squareone (science platform homepage).
This technical note describes the data model and functionality of Semaphore broadcast messaging.

.. _intro:

Introduction to Semaphore
=========================

Semaphore is intended to become a centralized service for messaging users on the |RSP| through multiple appropriate channels, including:

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

- GitHub repository serving as a content management system
- Semaphore service layer, providing an API
- UI components embedded in Squareone_, and possibly other |RSP| applications, to consume the Semaphore API and display messages to the user

GitHub makes sense as a CMS because it provides both a user interface and an authorization system (including pull request reviews) that the team is already familiar with.
Further, the cadence of broadcast messages is slow enough that a GitHub-based process of committing a content change and merging a pull request is reasonably efficient.
We anticipate that a GitHub repository could be dedicated to |RSP| broadcast messages (or the Phalanx deployment repository could be co-opted for broadcast messages).
Messages are individual files in a plain text format that supports metadata.
Two possible formats are YAML or Markdown with YAML-formatted front-matter (the latter option makes it easier for authors to write and format body content).
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
This user interface can accommodate this information by including a disclosure affordance on the banner that either opens a modal with the message body, or expands the banner to reveal the message body.
Both of these approaches avoid taking the user away from the UI (and thus the work) that they are currently doing.
On a technical level, this message body is written as markdown, providing the authors all the tools needed to link elsewhere or provide a formatted message.

All broadcast messages are time-sensitive, on some level, and both the visual presentation and message data model need to be built around that.
First, "old" broadcast messages should never be shown.
Thus all messages should have expiration dates encoded in their metadata.
Second, a message may be not yet relevant; thus messages should be able to provide defer dates that schedule their appearance.
Both of these considerations ensure that broadcast messages are relevant to users.

A common use-case for broadcast-style messages is to pre-announce an event.
Messages should provide an event date in a structured format so that the client can display the date and time in the user's timezone, and also so that this date information can be displayed both prominently and consistently alongside the message's summary.
For events that are about to happen imminently, the client could also opt to display this event timestamp as a countdown clock (for example, 10 minutes until maintenance).

References
==========

.. bibliography::
