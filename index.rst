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

References
==========

.. bibliography::
