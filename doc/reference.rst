telepot 10.1 reference
======================

Telepot has two versions:

- **Traditional version works on Python 2.7 and Python 3.** It uses
  `urllib3 <https://urllib3.readthedocs.io/en/latest/>`_ to make HTTP requests,
  and uses threads to achieve delegation by default.
- **Async version works on Python 3.5 or above.** It is based on
  `asyncio <https://docs.python.org/3/library/asyncio.html>`_, uses
  `aiohttp <http://aiohttp.readthedocs.io/en/stable/>`_ to make asynchronous
  HTTP requests, and uses asyncio tasks to achieve delegation.

This page focuses on traditional version. Async version's usage is very similar,
the most significant differences being:

- Blocking methods (mostly network operations) become coroutines, and should be
  called with ``await``.
- Delegation is achieved by tasks, instead of threads. Thread-safety ceases to
  be a concern.

Traditional modules are under the package :mod:`telepot`, while async modules are
under :mod:`telepot.aio`:

+-------------------+-----------------------+
| Traditional       | Async                 |
+===================+=======================+
| telepot           | telepot.aio           |
+-------------------+-----------------------+
| telepot.delegate  | telepot.aio.delegate  |
+-------------------+-----------------------+
| telepot.helper    | telepot.aio.helper    |
+-------------------+-----------------------+
| telepot.routing   | telepot.aio.routing   |
+-------------------+-----------------------+
| telepot.api       | telepot.aio.api       |
+-------------------+-----------------------+

Some modules do not have async counterparts, e.g. :mod:`telepot.namedtuple` and
:mod:`telepot.exception`, because they are shared.

Try to combine this reading with the provided
`examples <https://github.com/nickoala/telepot/tree/master/examples>`_ .
One example is worth a thousand words. I hope they make things clear.

Basic Bot
---------

The ``Bot`` class is mostly a wrapper around `Telegram Bot API <https://core.telegram.org/bots/api>`_.
Many methods are straight mappings to Bot API methods. Where appropriate,
I only give links below. No point to duplicate all the details.

.. autoclass:: telepot.Bot
   :members:

Functions
---------

.. autofunction:: telepot.flavor
.. autofunction:: telepot.glance
.. autofunction:: telepot.flance
.. autofunction:: telepot.peel
.. autofunction:: telepot.fleece
.. autofunction:: telepot.is_event
.. autofunction:: telepot.message_identifier
.. autofunction:: telepot.origin_identifier

DelegatorBot
------------

.. autoclass:: telepot.DelegatorBot

A *seeder* is a function that:

- takes one argument - a message
- returns a *seed*. Depending on the nature of the seed, behavior is as follows:
    - if the seed is a hashable (e.g. number, string, tuple), it looks for a
      *delegate* associated with the seed. (Think of a dictionary of ``{seed: delegate}``)
        - if such a delegate exists and is alive, it is assumed that the message
          will be picked up by the delegate. Nothing more is done.
        - if no delegate exists or that delegate is no longer alive, a new
          delegate is obtained by calling the delegator function. The new
          delegate is associated with the seed.
        - **In essence, when the seed is a hashable, only one delegate is running
          for a given seed.**
    - if the seed is a non-hashable, (e.g. list), a new delegate is always
      obtained by calling the delegator function. No seed-delegate association
      occurs.
    - if the seed is ``None``, nothing is done.

A *delegator* is a function that:

- takes one argument - a (bot, message, seed) tuple. This is called a *seed tuple*.
- returns a *delegate*, which can be one of the following:
    - an object that has methods ``start()`` and ``is_alive()``. Therefore, a
      ``threading.Thread`` object is a natural delegate. Once returned, the
      object's ``start()`` method is called.
    - a function. Once returned, it is wrapped in a ``Thread(target=function)``
      and started.
    - a (function, args, kwargs) tuple. Once returned, it is wrapped in a
      ``Thread(target=function, args=args, kwargs=kwargs)`` and started.

The above logic is implemented in the ``handle`` method.
You only have to call :meth:`.Bot.message_loop` with no ``callback``
argument, the above logic will be executed for every message received.

In the list of delegation patterns, all seeder functions are evaluated in order.
One message may start multiple delegates.

The module :mod:`telepot.delegate` has a bunch of seeder factories
and delegator factories, which greatly ease the use of DelegatorBot. The module
:mod:`telepot.helper` also has a number of ``*Handler`` classes which provide
a connection-like interface to deal with individual chats or users.

In the rest of discussions, *seed tuple* means a (bot, message, seed) tuple,
referring to the single argument taken by delegator functions.

``telepot.delegate``
--------------------

.. automodule:: telepot.delegate
   :members:

``telepot.helper``
------------------

Handlers
++++++++

.. autoclass:: telepot.helper.Monitor
   :show-inheritance:
   :members:

.. autoclass:: telepot.helper.ChatHandler
   :show-inheritance:
   :members:

.. autoclass:: telepot.helper.UserHandler
   :show-inheritance:
   :members:

.. autoclass:: telepot.helper.InlineUserHandler
   :show-inheritance:
   :members:

.. autoclass:: telepot.helper.CallbackQueryOriginHandler
   :show-inheritance:
   :members:

Contexts
++++++++

.. autoclass:: telepot.helper.ListenerContext
   :members:
   :undoc-members:

.. autoclass:: telepot.helper.ChatContext
   :show-inheritance:
   :members:
   :undoc-members:

.. autoclass:: telepot.helper.UserContext
   :show-inheritance:
   :members:
   :undoc-members:

.. autoclass:: telepot.helper.CallbackQueryOriginContext
   :show-inheritance:
   :members:
   :undoc-members:

.. autoclass:: telepot.helper.Sender
   :members:

.. autoclass:: telepot.helper.Administrator
   :members:

.. autoclass:: telepot.helper.Editor
   :members:

.. autoclass:: telepot.helper.Listener
   :members:

Mixins
++++++

.. autoclass:: telepot.helper.Router
   :members:

.. autoclass:: telepot.helper.DefaultRouterMixin
   :members:
   :undoc-members:

.. autoclass:: telepot.helper.StandardEventScheduler
   :members:
   :undoc-members:

.. autoclass:: telepot.helper.StandardEventMixin
   :members:
   :undoc-members:
   :exclude-members: StandardEventScheduler

.. autoclass:: telepot.helper.IdleEventCoordinator
   :members:

.. autoclass:: telepot.helper.IdleTerminateMixin
   :members:
   :undoc-members:
   :exclude-members: IdleEventCoordinator

.. autoclass:: telepot.helper.CallbackQueryCoordinator
   :members:
   :undoc-members:

.. autoclass:: telepot.helper.InterceptCallbackQueryMixin
   :members:
   :undoc-members:
   :exclude-members: CallbackQueryCoordinator

.. autoclass:: telepot.helper.Answerer
   :members:

.. autoclass:: telepot.helper.AnswererMixin
   :members:
   :undoc-members:
   :exclude-members: Answerer

Utilities
+++++++++

.. autoclass:: telepot.helper.SafeDict
   :members:

.. autofunction:: telepot.helper.openable

``telepot.exception``
---------------------

.. automodule:: telepot.exception
   :members:
   :undoc-members:

``telepot.namedtuple``
----------------------

Telepot's custom is to represent Bot API object as *dictionary*.
On the other hand, the module :mod:`telepot.namedtuple` also provide namedtuple
classes mirroring those objects. The reasons are twofold:

1. Under some situations, you may want an object with a complete set of fields,
   including those whose values are ``None``. A dictionary translated from Bot API's response
   would have those ``None`` fields absent. By converting such a dictionary to a namedtuple,
   all fields are guaranteed to be present, even if their values are ``None``.
   This usage is for **incoming** objects received from Telegram servers.

2. Namedtuple allows easier construction of objects like
   `ReplyKeyboardMarkup <https://core.telegram.org/bots/api#replykeyboardmarkup>`_,
   `InlineKeyboardMarkup <https://core.telegram.org/bots/api#inlinekeyboardmarkup>`_,
   and various `InlineQueryResult <https://core.telegram.org/bots/api#inlinequeryresult>`_, etc.
   This usage is for **outgoing** objects sent to Telegram servers.

Incoming objects include:

- `User <https://core.telegram.org/bots/api#user>`_
- `Chat <https://core.telegram.org/bots/api#chat>`_
- `Message <https://core.telegram.org/bots/api#message>`_
- `MessageEntity <https://core.telegram.org/bots/api#messageentity>`_
- `PhotoSize <https://core.telegram.org/bots/api#photosize>`_
- `Audio <https://core.telegram.org/bots/api#audio>`_
- `Document <https://core.telegram.org/bots/api#document>`_
- `Sticker <https://core.telegram.org/bots/api#sticker>`_
- `Video <https://core.telegram.org/bots/api#video>`_
- `Voice <https://core.telegram.org/bots/api#voice>`_
- `Contact <https://core.telegram.org/bots/api#contact>`_
- `Location <https://core.telegram.org/bots/api#location>`_
- `Venue <https://core.telegram.org/bots/api#venue>`_
- `UserProfilePhotos <https://core.telegram.org/bots/api#userprofilephotos>`_
- `File <https://core.telegram.org/bots/api#file>`_
- `ChatMember <https://core.telegram.org/bots/api#chatmember>`_
- `CallbackQuery <https://core.telegram.org/bots/api#callbackquery>`_
- `InlineQuery <https://core.telegram.org/bots/api#inlinequery>`_
- `ChosenInlineResult <https://core.telegram.org/bots/api#choseninlineresult>`_

Outgoing objects include:

- `ReplyKeyboardMarkup <https://core.telegram.org/bots/api#replykeyboardmarkup>`_
- `KeyboardButton <https://core.telegram.org/bots/api#keyboardbutton>`_
- `ReplyKeyboardHide <https://core.telegram.org/bots/api#replykeyboardhide>`_
- `InlineKeyboardMarkup <https://core.telegram.org/bots/api#inlinekeyboardmarkup>`_
- `InlineKeyboardButton <https://core.telegram.org/bots/api#inlinekeyboardbutton>`_
- `ForceReply <https://core.telegram.org/bots/api#forcereply>`_
- Various types of `InlineQueryResult <https://core.telegram.org/bots/api#inlinequeryresult>`_
- Various types of `InputMessageContent <https://core.telegram.org/bots/api#inputmessagecontent>`_

``telepot.routing``
-------------------

.. automodule:: telepot.routing
   :members:
