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


## The plan

So let me share the grand vision with you. `pwnpkg` will replace `rpkg` with
the generic functionality shared between the packaging tools, such as

 * minimal [declarative way](https://github.com/redhat-openstack/rdopkg/blob/master/rdopkg/actions.py) to describe both CLI and trasaction-ish program flow
 * convenient wrappers for interfacing with the system/shell
 * descriptive and [useful errors](https://github.com/redhat-openstack/rdopkg/blob/master/rdopkg/exception.py)
 * nice logging with colors out of the box
 * full modularity without importing modules that aren't needed

and also shared higher level utilities and conventions such as

 * parsing and manipulating manipulating `.spec` files
 * interacting with `dist-git`
 * automagically detecting package environment

and whatever else will be required.

Most of the above is implemented in `rdopkg` already, so it's a matter of
splitting and restructuring.

Finally, `rdopkg` and `fedpkg` will be rewritten to use `pwnpkg` framework and
if it succeeds, other tools such as `centpkg`, `rhpkg` or `copr-cli` can join
the revolution.

The plan is to create state of the art packaging tooling for RPM based systems
which is a pleasure to use both from command line and from python. A powerful
tool that automates all the steps that dosn't really require human input while
enforcing sane minimalist conventions (instead of configuration) that will
make it easy for packagers to swiftly package anything upstream throws at
them.


## Interested?

I'm pretty serious about this so if you like what you read, please do get
involved in the packaging revolution!

You can use

 * github Issues
 * IRC: `jruzicka` @ `#rdo` on freenode IRC
 * mail: `jruzicka` at `fedorapeople.org`
