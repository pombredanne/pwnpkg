# pwnpkg


## What? Why?

I'm not the first person to discover that `fedpkg` and the underlying `rpkg`
python modules have terrible source code.

Do you think it's OK to write [over 100 lines of hacks, wrappers and
mockings](https://github.com/redhat-openstack/rdopkg/blob/master/rdopkg/actionmods/kojibuild.py)
to achieve something as simple as following?

    from fedpkg import kojibuild

    build_id = kojibuild.new_build()
    kojibuild.watch_build(build_id)

Neither do I and thus I wrote
[rdopkg](https://github.com/redhat-openstack/rdopkg)
from scratch, the Swiss army knife of RDO packaging. You can reuse any of its
parts on a level you chose. You can use the low level functions directly. You
can use higher level wrappers full of white automagic. Or you can unleash full
rdopkg superpowers with transaction-like series of idempotent steps that allow
you to fix the problem in your shell and then just `rdopkg --continue`. Just
find what you need, import it, and use it.

Although minimalist and lightweight in nature, `rdopkg` is now too big with
over 4000 SLOC (organised into modules, not a ball of spaghetti) covering many
problems we encountered in RDO, mostly related to RPM packaging. It contains
quite a few nice tricks I stole from various python projects and some features
I only wished that existed. It's time to properly split all this awesomenes
and make it (re)useful for everyone.

Oh, and also time to rewrite `fedpkg`.


## The Plan

So let me share the grand vision with you. `pwnpkg` will provide convenient
yet lightweight framework for building high quality modular CLI tools out of
the box. This includes:

 * minimal [declarative way](https://github.com/openstack-packages/rdopkg/blob/master/rdopkg/actions.py) to describe both CLI and trasaction-ish program flow
 * lightweight modularity with on-demand module import
 * convenient wrappers for interfacing with the system/shell
 * descriptive and [useful errors](https://github.com/openstack-packages/rdopkg/blob/master/rdopkg/exception.py)
 * nice logging with colors out of the box

Functionality shared across different packaging tools will be provided as
`pwnpkg` action modules, not limited to:

 * parsing and manipulating manipulating `.spec` files
 * interacting with `distgit` (patches management, auto rebases)
 * automagically detecting package environment

Most of the above functionality is implemented in `rdopkg` already, so it's
a matter of splitting and restructuring.

Once ready, `rdopkg` will be rewritten using `pwnpkg` framework and the
package handling automagic will be split into modules that can be
shared across packaging tools such as `fedpkg`, `centpkg`, `rhpkg` or
`copr-cli`. These tools could eventually be rewritten using `pwnpkg` if it
succeeds or at least use the provided packaging functionality directly,
reducing effort duplication as much as possible.

The plan is to create state of the art pluggable CLI framework as well as
a set of reusable modules for packaging tooling for RPM based systems that are
a pleasure to use both from command line and from python. A tool to create
powerful tools that automate all the steps that dosn't really require human
input while enforcing sane minimalist conventions (over configuration) that
will make it easy for packagers to swiftly package anything upstream throws at
them.


## Action Modules

`rdopkg` already provides action manager that allows single definition for
both command line and programming interfaces. This is done using so called
**actions** with name and parameters that simply map by convention to python
action module functions. This already allows transaction-ish and terminal
friendly operation where `rdopkg` drops to terminal when human interaction is
required, let's you fix the problem using the tools of your preference
and then continue the transaction with `rdopkg --continue`.

I'll refine the concept of *action modules* further to allow lightweight
modularity and on-demand module imports, as opposed to usual [IMPORT ALL THE
PYTHON MODULES](https://jruzicka.fedorapeople.org/pkgs/import.jpg) madness
which can take up to a second of hardcore importing for a simple command(!) as
seen in `openstackclient`.

And thus, when `pwnpkg` action module gets imported, its `__init__.py` only
contains `ACTIONS` structure that that describes module endpoints (actions),
which can be atomic or a series of actions that directly map by convention to
functions in `actions.py` which is loaded on demand when module functionality
is actually required. Action modules can (and will) contain additional python
submodules that implement actual features. These ordinary python modules
can be used (imported) directly when desired, but on top of that, `actions.py`
structures this functionality into convenient consumable chunks.

Thanks to this design, `pwnpkg` modules should be reausable by default while
not bloating even with lots of different functionality with
various requirements in one tool.


## Status

After trying to work on `pwnpkg` alongside `rdopkg`, I came to a conclusion
I need to implement everything I need in `rdopkg` and split afterwards due to
limited bandwidth and overhead of syncing two ever diverging code bases.

### What remains to be done on `rdopkg` before `pwnpkg` can happen

 * cleanup - remove obsolete actions and modules
 * modular actions system (PROTOTYPE WORKING)
 * factor current code into easily maintainable and pluggable modules
 * allow easy creation of new tool instances (MOSTLY DONE)


## Interested?

I'm pretty serious about this so if you like what you read, please do get
involved or at least let me know you think all this effort makes some sense.
Are you a potential `pwnpkg` user? Do you want to be the hero to rewrite
`fedpkg`? Let me know what you think.

 * github Issues
 * IRC: `jruzicka` @ `#rdo` on freenode IRC
 * mail: `jruzicka` at `fedorapeople.org`
