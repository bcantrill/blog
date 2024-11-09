---
title: "DTrace and PHP"
date: "2005-08-04"
categories: 
  - "solaris"
---

Tonight during our [OpenSolaris](http://opensolaris.org) BOF at OSCON, PHP core developer [Wez Furlong](http://netevil.org/wiki.php?WezFurlong) was busy adding a [DTrace](http://opensolaris.org/os/community/dtrace) provider to PHP. After a little bit of work (and a little bit of debugging), we got it working -- and **damn** is it cool. Wez implemented it as a shared object, which may then be loaded via an explicit [extension](http://www.mcs.vuw.ac.nz/technical/software/PHP/configuration.html#ini.extension) directive in [php.ini](http://www.mcs.vuw.ac.nz/technical/software/PHP/configuration.html). Once loaded, two probes show up: `function-entry` and `function-return`. These probes have as their arguments a pointer to the function name, a pointer to the file name, and a line number. This allows one to, for example, get a count of all PHP functions being called:

```
# dtrace -n function-entry'{@[copyinstr(arg0)] = count()}'

```

Or you can aggregate on file name and quantize by line number:

```
# dtrace -n function-entry'{@[copyinstr(arg1)] = lquantize(arg2, 0, 5000)}'

```

Or you can determine the amount of wall time taken by a given PHP function:

```
# dtrace -n function-entry'/copyinstr(arg0) == "myfunc"/{self->ts = timestamp}'
-n function-return'/self->ts/{@ = avg(timestamp - self->ts); self->ts = 0)}'

```

And because it's DTrace, this can all be done on a production box -- and without regard to the number of PHP processes. (So if you have 200 Apache processes handling PHP, the above invocations would aggregate across them.) When I get back, I'll download Wez's provider and post some more comprehensive examples. In the meantime, if you're a PHP developer at OSCON, stop Wez if you see him and ask him to give you a demo -- it's the kind of thing that needs to be seen to be appreciated...

Finally, if you're interested in adding your own DTrace provider to the application, language or system that **you** care about, be sure to check out my presentation on DTrace tomorrow at 4:30 in Portland room 255. (Hopefully this time I won't be tortured by memories of being mindfucked by [Inside UFO 54-40](http://gamebooks.org/show_item.php?id=554).)
