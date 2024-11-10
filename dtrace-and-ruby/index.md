---
title: "DTrace and Ruby"
date: "2005-08-21"
categories: 
  - "solaris"
---

It's been an exciting few weeks for [DTrace](http://opensolaris.org/os/community/dtrace/). The party got started with [Wez Furlong's](http://netevil.org/) new [PHP DTrace provider](http://blogs.sun.com/roller/page/bmc?entry=dtrace_and_php_demonstrated) at OSCON. Then [Devon O'Dell](http://www.sitetronics.com/wordpress/) announced that he was starting to work in earnest on a [DTrace port to FreeBSD](http://blogs.sun.com/roller/page/bmc?entry=dtrace_on_freebsd). And now, [Rich Lowe](mailto:richlowe@richlowe.net) has made available a prototype [Ruby DTrace provider](http://richlowe.net/~richlowe/patches/ruby-1.8.2-dtrace.diff). To install this, grab [Ruby 1.8.2](ftp://ftp.ruby-lang.org/pub/ruby/ruby-1.8.2.tar.gz), apply [Rich's patch](http://richlowe.net/~richlowe/patches/ruby-1.8.2-dtrace.diff), and run `./configure` with the `--enable-dtrace` option. When you run the resulting `ruby`, you'll see two probes: `function-entry` and `function-return`. The arguments to these probes are as follows:

- `arg0` is the name of the class (a pointer to a string within Ruby)
    
- `arg1` is the name of the method (also a pointer to a string within Ruby)
    
- `arg2` is the name of the file containing the call site (again, a pointer to a string within Ruby)
    
- `arg3` is the line number of the call site.
    

So if, for example, you'd like to know the classes and methods that are called in a particular Ruby script, you could do it with this simple D script:

```
#pragma D option quiet
ruby$target:::function-entry
{
@[copyinstr(arg0), copyinstr(arg1)] = count();
}
END
{
printf("%15s %30s   %s\n", "CLASS", "METHOD", "COUNT");
printa("%15s %30s   %@d\n", @);
}

```

To run this against the `cal.rb` that ships in the `sample` directory of Ruby, call the above script `whatmethods.d` and run it this way:

```
# dtrace -s ./whatmethods.d -c "../ruby ./cal.rb"
August 2005
S  M Tu  W Th  F  S
1  2  3  4  5  6
7  8  9 10 11 12 13
14 15 16 17 18 19 20
21 22 23 24 25 26 27
28 29 30 31
CLASS                         METHOD   COUNT
Array                             <=   2
Hash                           each   2
Hash                           keys   2
Module                           attr   2
Module               method_undefined   2
Module                         public   2
Rational                         coerce   2
Array                              +   3
Class                    civil_to_jd   3
Hash                             []   3
Object                        collect   3
Array                        collect   4
Class                      inherited   4
Range                           each   4
String                           size   4
Module           private_class_method   5
Object                           eval   5
Object                        require   5
String                           gsub   5
Class                     jd_to_wday   7
Class                           once   7
Date                       __8713__   7
Date                           wday   7
Fixnum                              %   8
Array                           join   10
Hash                            []=   10
String                              +   10
Array                           each   11
NilClass                           to_s   11
Module                   alias_method   22
Module                        private   22
Symbol                           to_s   26
Module                    module_eval   28
Date                           mday   31
Object                           send   31
Date                            mon   42
Date                      __11105__   43
Class                    jd_to_civil   45
Date                           succ   47
Class                            os?   48
Date                              +   49
Fixnum                             <=   49
String                          rjust   49
Class                      ajd_to_jd   50
Class                        clfloor   50
Date                      __10417__   50
Integer                           to_i   50
Object                        Integer   50
Rational                         divmod   50
Rational                           to_i   50
Date                               51
Date                            ajd   51
Date                             jd   51
Rational                               51
Class                           new0   52
Date                     initialize   52
Integer                           to_r   52
Object         singleton_method_added   67
Date                          civil   75
Symbol                           to_i   91
Float                              *   96
Float                         coerce   96
Fixnum                              /   97
Object                        frozen?   100
Rational                              - 104
Fixnum                           to_s   123
Array                             []   141
Object                          class   150
Module                   method_added   154
Float                              /   186
Module                            ===   200
Rational                              /   204
Rational                              +   248
Float                          floor   282
Fixnum                             <<   306
Class                         reduce   356
Integer                            gcd   356
Object                       Rational   356
Fixnum                              +   414
Class                           new!   610
Rational                     initialize   610
Class                            new   612
Fixnum                            abs   712
Fixnum                            div   762
Fixnum                              *   1046
Fixnum                                 1970
Fixnum                              - 2398
Object                       kind_of?   2439
Fixnum                             >>   4698
Fixnum                             []   7689
Fixnum                             ==   11436

```

This may leave us with many questions. For example, there are a couple of calls to construct new objects -- where are they coming from? To answer that question:

```
#pragma D option quiet
ruby$target:::function-entry
/copyinstr(arg1) == "initialize"/
{
@[copyinstr(arg0), copyinstr(arg2), arg3] = count();
}
END
{
printf("%-10s %-40s %-10s %s\n", "CLASS",
"INITIALIZED IN FILE", "AT LINE", "COUNT");
printa("%-10s %-40s %-10d %@d\n", @);
}

```

Calling the above `whereinit.d`, we can run it in a similar manner:

```
# dtrace -s ./whereinit.d -c "../ruby ./cal.rb"
August 2005
S  M Tu  W Th  F  S
1  2  3  4  5  6
7  8  9 10 11 12 13
14 15 16 17 18 19 20
21 22 23 24 25 26 27
28 29 30 31
CLASS      INITIALIZED IN FILE                      AT LINE    COUNT
Cal        ./cal.rb                                 144        1
Date       /usr/local/lib/ruby/1.8/date.rb          593        1
Date       /usr/local/lib/ruby/1.8/date.rb          703        1
Date       /usr/local/lib/ruby/1.8/date.rb          916        1
Time       /usr/local/lib/ruby/1.8/date.rb          702        1
Date       /usr/local/lib/ruby/1.8/date.rb          901        49
Rational   /usr/local/lib/ruby/1.8/rational.rb      374        610

```

Looking at the `Date` class, it's interesting to look at line 901 of file `/usr/local/lib/ruby/1.8/date.rb`:

```
897   # If +n+ is not a Numeric, a TypeError will be thrown.  In
898   # particular, two Dates cannot be added to each other.
899   def + (n)
900     case n
   901     when Numeric; return self.class.new0(@ajd + n, @of, @sg)
902     end
903     raise TypeError, 'expected numeric'
904   end

```

This makes sense: we're initializing new `Date` instances in the `+` method for `Date`. And where are _those_ coming from? It's not hard to build a script that will tell us the file and line for the call site of an arbitrary class and method:

```
#pragma D option quiet
ruby$target:::function-entry
/copyinstr(arg0) == $$1 && copyinstr(arg1) == $$2/
{
@[copyinstr(arg2), arg3] = count();
}
END
{
printf("%-40s %-10s %s\n", "FILE", "LINE", "COUNT");
printa("%-40s %-10d %@d\n", @);
}

```

For this particular example (`Date#+()`), call the above `wherecall.d` and run it this way:

```
# dtrace -s ./wherecall.d "Date" "+" -c "../ruby ./cal.rb"
August 2005
S  M Tu  W Th  F  S
1  2  3  4  5  6
7  8  9 10 11 12 13
14 15 16 17 18 19 20
21 22 23 24 25 26 27
28 29 30 31
FILE                                     LINE       COUNT
./cal.rb                                 102        2
./cal.rb                                 60         6
./cal.rb                                 63         41

```

And looking at the indicated lines in cal.rb:

```
55   def pict(y, m)
56     d = (1..31).detect{|d| Date.valid_date?(y, m, d, @start)}
57     fi = Date.new(y, m, d, @start)
58     fi -= (fi.jd - @k + 1) % 7
59
    60     ve  = (fi..fi +  6).collect{|cu|
61       %w(S M Tu W Th F S)[cu.wday]
62     }
    63     ve += (fi..fi + 41).collect{|cu|
64       if cu.mon == m then cu.send(@da) end.to_s
65     }
66

```

So this is doing exactly what we would expect, given the code. Now, if we were interested in making this perform a little better, we might be interested to know the work that is being induced by `Date#+()`. Here's a script that reports the classes and methods called by a given class/method -- and its callees:

```
#pragma D option quiet
ruby$target:::function-entry
/copyinstr(arg0) == $$1 && copyinstr(arg1) == $$2/
{
self->date = 1;
}
ruby$target:::function-entry
/self->date/
{
@[strjoin(strjoin(copyinstr(arg0), "#"),
copyinstr(arg1))] = count();
}
ruby$target:::function-return
/copyinstr(arg0) == $$1 && copyinstr(arg1) == $$2/
{
self->date = 0;
ndates++;
}
END
{
normalize(@, ndates);
printf("Each call to %s#%s() induced:\n\n", $$1, $$2);
printa("%@8d call(s) to %s()\n", @);
}

```

Calling the above `whatcalls.d`, we can answer the question about `Date#+()` this way:

```
# dtrace -s ./whatcalls.d "Date" "+" -c "../ruby ./cal.rb"
August 2005
S  M Tu  W Th  F  S
1  2  3  4  5  6
7  8  9 10 11 12 13
14 15 16 17 18 19 20
21 22 23 24 25 26 27
28 29 30 31
Each call to Date#+() induced:
1 call(s) to Class#new0()
1 call(s) to Class#reduce()
1 call(s) to Date#+()
1 call(s) to Date#initialize()
1 call(s) to Fixnum#+()
1 call(s) to Fixnum#<<()
1 call(s) to Integer#gcd()
1 call(s) to Module#===()
1 call(s) to Object#Rational()
1 call(s) to Object#class()
2 call(s) to Class#new()
2 call(s) to Class#new!()
2 call(s) to Fixnum#abs()
2 call(s) to Fixnum#div()
2 call(s) to Rational#+()
2 call(s) to Rational#initialize()
3 call(s) to Fixnum#*()
3 call(s) to Fixnum#()
23 call(s) to Fixnum#>>()
37 call(s) to Fixnum#[]()
52 call(s) to Fixnum#==()

```

That's a lot of work for something that should be pretty simple! Indeed, it's counterintuitive that, say, `Integer#gcd()` would be called from `Date#+()` -- and it certainly seems suboptimal. I'll leave further exploration into this as an exercise to the reader, but suffice it to say that this has to do with the use of a rational number in the `Date` class -- the elimination of which would elminate most of the above calls and presumably greatly improve the performance of `Date#+()`.

Now, Ruby aficionados may note that some of the above functionality has been available in Ruby by setting the `set_trace_func` function (upon which the [Ruby profiler](http://www.rubygarden.org/ruby?Ruby_Profiler) is implemented). While that's true (to a point -- the `set_trace_func` seems to be a pretty limited mechanism), the Ruby DTrace provider is nonetheless a great lurch forward for Ruby developers: it allows developers to use DTrace-specific constructs like aggregations and thread-local variables to hone in on a problem; it allows Ruby-related work performed lower in the stack (e.g., in the I/O subsystem, CPU dispatcher or network stack) to be connected to the Ruby code inducing it; it allows a running Ruby script to be instrumented (and reinstrumented) without stopping or restarting it; and it allows multiple, disjoint Ruby scripts to be coherently observed and understood as a single entity. More succinctly, it's just damned cool. So thanks to Rich for developing the prototype provider -- and if you're a Ruby developer, enjoy!
