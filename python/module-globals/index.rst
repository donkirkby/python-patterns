
================
 Module Globals
================

*A “Creational Pattern” inspired by Modula-2 and Modula-3*

.. admonition:: Verdict

   The core Python language,
   its Standard Library,
   and many third party packages
   offer access to convenient pre-built objects
   by assigning them a name in a module’s global namespace.
   Global objects are easy to understand, easy to use,
   and can never conflict with identical names in other packages
   because every Python module is its own namespace.


.. TODO Add this one I do the singleton:
   Module globals are more common in Python
   than the Gang of Four’s :doc:`gang-of-four/singleton`,
   which was a trick to avoid creating any more global names than necessary
   in languages without the benefit of a module system.

.. contents:: Contents:
   :backlinks: none

.. TODO mention how for verbs, not nouns, we put methods in the global
   namespace; exmaples are random and json modules

global level
============

functions classes

and named tuples

functions: random; dumps loads pickle? definitely json. requests?

does double duty:
namespace has both the things you’re defining,
but also the things you yourself need

simple alias:
operator.py __lt__ = lt

show how to tell which functions and classes belong to the module

as from . import context

you can name mod.thing
or thing
or renamed_thing, even built in to syntax
as “from...as”

I don’t mean things imported
it’s generally bad to use those unless specific “api” module
pattern is BAD of having __init__.py load things

_ForkingPickler = context.reduction.ForkingPickler

constant values
===============

constants (strings.* etc) vs live objects

computed constants

File: Lib/unittest/test/testmock/__init__.py
6:1:here = os.path.dirname(__file__)
^ is this anywhere non-test?

or for efficiency or for readability? ONE_SIXTH = 1.0/6.0

File: Lib/json/encoder.py
34:1:INFINITY = float('inf')

can also be used to avoid recompute “if”:
COPY_BUFSIZE = 1024 * 1024 if _WINDOWS else 16 * 1024

composite
File: Lib/datetime.py
2218:1:_EPOCH = datetime(1970, 1, 1, tzinfo=timezone.utc)

“I could have done that!”
from types.py:
File: Lib/types.py
12:1:FunctionType = type(_f)
LambdaType = type(lambda: None)
File: Lib/_collections_abc.py
36:1:bytes_iterator = type(iter(b''))
37:1:bytearray_iterator = type(iter(bytearray()))
39:1:dict_keyiterator = type(iter({}.keys()))
40:1:dict_valueiterator = type(iter({}.values()))
41:1:dict_itemiterator = type(iter({}.items()))
42:1:list_iterator = type(iter([]))

dunder constants
================

dunder metadata

File: Lib/xdrlib.py
__all__ = ["Error", "Packer", "Unpacker", "ConversionError"]

File: Lib/__future__.py
50:1:all_feature_names = [
63:1:__all__ = ["all_feature_names"] + all_feature_names
128:1:print_function = _Feature((2, 6, 0, "alpha", 2),
??

not constant at all
not only can you reassign, BUT often not even immutable data structs
why list?
doing tuple for all saves at least 16 bytes? and level of indirection
File: Lib/multiprocessing/context.py
8:1:__all__ = ()

Lib/asyncio/*.py use tuple for all
File: Lib/contextvars.py
4:1:__all__ = ('Context', 'ContextVar', 'Token', 'copy_context')
File: Lib/concurrent/futures/__init__.py
20:1:__all__ = (

File: Lib/tkinter/font.py
6:1:__version__ = "0.9"

File: Lib/turtle.py
103:1:_ver = "turtle 1.1b- - for Python 3.1   -  4. 5. 2009"

but should it be tuple or string?

File: Lib/bz2.py
10:1:__author__ = "Nadeem Vawda <nadeem.vawda@gmail.com>"

__author__ = ("Guido van Rossum <guido@python.org>, "

constant collections
====================

File: Parser/asdl.py
builtin_types = {'identifier', 'string', 'bytes', 'int', 'object', 'singleton',

File: Lib/asyncore.py
60:1:_DISCONNECTED = frozenset({ECONNRESET, ENOTCONN, ESHUTDOWN, ECONNABORTED, EPIPE,

Lib/asyncore.py
60:1:_DISCONNECTED = frozenset({ECONNRESET, ENOTCONN, ESHUTDOWN, ECONNABORTED, EPIPE,
^ differing levels of effort to make it constant

shutil.py
585:1:_use_fd_functions = ({os.open, os.stat, os.unlink, os.rmdir} <=
                     os.supports_dir_fd and
                     os.scandir in os.supports_fd and
                     os.stat in os.supports_follow_symlinks)
BARELY made sense

sentinels   <----------- pull out into its own article?
=========

File: Lib/bz2.py
27:1:_sentinel = object()  <--- line occurs several times
^ token? no.

Lib/functools.py
_NOT_FOUND = object()
val = cache.get(self.attrname, _NOT_FOUND)

File: Lib/configparser.py
357:1:_UNSET = object()

only for efficiency if used in only one routine

Precompiled globals
===================

compile re’s once
File: Lib/glob.py
142:1:magic_check = re.compile('([*?[])')

File: Lib/email/policy.py
23:1:linesep_splitter = re.compile(r'\n|\r')

File: Lib/signal.py
6:1:_globals = globals()

File: Lib/email/header.py
31:1:USASCII = Charset('us-ascii')

File: Lib/re.py
262:1:Pattern = type(sre_compile.compile('', 0))
263:1:Match = type(sre_compile.compile('', 0).match(''))

mutable globals
===============

everything is an object BUT I MEAN:

Pattern - “singleton” object

File: Lib/os.py
759:1:environ = _createenviron()

217:1:default = EmailPolicy()
^ useful objects

File: Lib/logging/__init__.py
641:1:_defaultFormatter = Formatter()
1156:1:_defaultLastResort = _StderrHandler(WARNING)
1834:1:root = RootLogger(WARNING)

Pattern - dispatch

File: Lib/copyreg.py
10:1:dispatch_table = {}
^ global mutable registry

don’t do I/O at top level to create object
if you really need to have a separate init or setup routine for it

private globals - somewhat different from ones that we want to share
File: Lib/multiprocessing/process.py
363:1:_current_process = _MainProcess()
364:1:_process_counter = itertools.count(1)

File: Lib/pydoc.py
1626:1:text = TextDoc()
1627:1:plaintext = _PlainTextDoc()
1628:1:html = HTMLDoc()
2101:1:help = Helper()

sometimes almost to make up for the lack of builtins

File: Lib/smtpd.py
106:1:DEBUGSTREAM = Devnull()
^ where messages are sent by default; you can replace with NOT:
class Devnull:
    def write(self, msg): pass
    def flush(self): pass
