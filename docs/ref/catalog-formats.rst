***************
Catalog formats
***************

Catalog format (configured by :conf:`format` option) refers to file format of
offline catalog. This format is never used in production, because it's compiled
into JS module. The reason behind this build step is that choice of catalog
format depends on individual internationalization workflow. On the other hand
runtime catalog should be as simple as possible so it parsed quickly without
additional overhead.

PO File (recommended)
=====================

PO files are translation files used by gettext_ internationalization system.
This format is recommended and in LinguiJS v3 it'll be the default catalog format.

The advantages of this format are:

- readable even for large messages
- supports comments for translators
- supports metadata (origin, flags)
- standard format supported by many localization tools

.. code-block:: po

   #: src/App.js:3
   #  Comment for translators
   msgctx "context"
   msgid "messageId"
   msgstr "Translated Message"

   #: src/App.js:3
   #, obsolete
   msgid "obsoleteId"
   msgstr "Obsolete Message"

.. _gettext: https://www.gnu.org/software/gettext/manual/html_node/PO-Files.html

.. _po-gettext:

PO File with gettext Plurals
============================

When using localization backends that don't understand the ICU plural syntax exported by the default `po` formatter,
**po-gettext** can be used to read and write to PO files using gettext-native plurals.

This is how the regular PO format exports plurals:

.. code-block:: po

   msgid "{count, plural, one {Message} other {Messages}}"
   msgstr "{count, plural, one {Message} other {Messages}}"

With `po-gettext`, plural messages are exported in the following way, depending on wheter an explicit ID is set:

.. code-block:: po

   # Message with custom ID "my_message" that is pluralized on property "someCount".
   #
   # Notice that 'msgid_plural' was generad by appending a '_plural' suffix.
   #. js-lingui:pluralize_on=someCount
   msgid "my_message"
   msgid_plural "my_message_plural"
   msgstr[0] "Singular case"
   msgstr[1] "Case number {someCount}"

   # Message without custom ID that is pluralized on property "anotherCount".
   #
   # Notice how 'msgid' and 'msgid_plural' were extracted from original message.
   #
   # To allow matching this PO item to the appropriate catalog entry when deserializing,
   # the original ICU message is also stored in the generated comment.
   #. js-lingui:icu=%7BanotherCount%2C+plural%2C+one+%7BSingular+case%7D+other+%7BCase+number+%7BanotherCount%7D%7D%7D&pluralize_on=anotherCount
   msgid "Singular case"
   msgid_plural "Case number {anotherCount}"
   msgstr[0] "Singular case"
   msgstr[1] "Case number {anotherCount}"

Note that this format comes with several caveats and should therefore only be used if using ICU plurals in PO files is
not an option:

  - Nested/multiple plurals in one message as shown in :jsmacro:`plural` are not supported as they cannot be expressed
    with gettext plurals. Messages containing nested/multiple formats will not be output correctly.

  - :jsmacro:`select` and :jsmacro:`selectOrdinal` cannot be expressed with gettext plurals, but the original ICU format
    is still saved to the `msgid`/`msgstr` properties. To disable the warning that this might not be the expected
    behavior, include :code:`{ disableSelectWarning: true }` in the :conf:`formatOptions`.

  - Source/development languages with more than two plurals could experience difficulties when no custom IDs are used,
    as gettext cannot have more than two plurals cases identifying an item (:code:`msgid` and :code:`msgid_plural`).

  - Gettext doesn't support plurals for negative and fractional numbers even though some languages have special rules
    for these cases.


JSON
====

Simple JSON file where each key is message ID and value is translation. The JSON
is flat and there's no reason to use nested keys. The usual motivation behind nested
JSON is to save filespace, but this file format is used offline only.

The drawback of this format is that all metadata about message are lost. That includes
default message, origin of message and any message flags (obsolete, fuzzy, etc).

.. code-block:: json

   {
      "messageId": "translation"
   }

Lingui (raw)
============

This file format simply outputs all internal data in JSON format. It's the original
file format used by LinguiJS library before support for other catalog formats were added.
It might be useful for tools build on top of Lingui CLI which needs to further
process catalog data.

.. code-block:: json

   {
      "messageId": {
         "translation": "Translated message",
         "defaults": "Default message",
         "context": "more context",
         "description": "Comment for translators",
         "origin": [["src/App.js", 3]]
      },
      "obsoleteId": {
         "translation": "Obsolete message",
         "origin": [["src/App.js", 3]],
         "obsolete": true
      }
   }
