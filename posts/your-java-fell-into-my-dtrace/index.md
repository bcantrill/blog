---
title: "Your Java fell into my DTrace!"
date: "2005-04-18"
categories: 
  - "solaris"
---

So I've been following with excitement the [JVMPI and JVMTI agents](https://solaris10-dtrace-vm-agents.dev.java.net/) prototyped by Jarod Jenson of [Aeysis](http://www.aeysis.com) and developed by [Kelly O'Hair](http://blogs.sun.com/roller/page/kto) of Sun (with some helpful guidance from [Adam](http://blogs.sun.com/ahl) on Team DTrace). The agent exports DTrace probes that correspond to well-known Java phenomena ("method-entry", "gc-start", "vm-init", etc), and while it isn't a perfect solution from a DTrace perspective (the requirement that one restart the JVM means it's still more appropriate for developers than for dynamic production use, for example), **damn** is it a big leap forward. For an example of its power, check out [Adam's blog entry](http://blogs.sun.com/roller/page/ahl/20050418#dtracing_java) following code flow from Java through native code and into the kernel (and back). Such a view of code flow is **without precedent** -- it is awesome (in all senses of that word) to see the progression through the stack of software abstraction.

Wanting to get in on the fun, I've been playing around with it myself. To get a sample program, I downloaded [Per Cerderberg](http://www.percederberg.net/)'s [Java implementation of Tetris](http://www.percederberg.net/home/java/tetris/tetris-1.2-bin.jar). To get to run with the agent, I run it this way:

```
# export LD_LIBRARY_PATH=/path/to/djvm/build/i386/lib
# java -Xrundjvmti:all -cp $LD_LIBRARY_PATH -jar tetris-1.2-bin.jar

```

The "`-Xrundjvmti`" pulls in the JVMTI agent, and the "`:all`" following it denotes that I want to get DTrace probes for all possible events. (It defaults to less than all events to minimize probe effect.) Running with this agent creates a new provider in the JVM process called `djvm`. Listing its probes:

```
# dtrace -l -P 'djvm*'
ID   PROVIDER            MODULE                          FUNCTION NAME
4    djvm793      libdjvmti.so                  gc_finish_worker gc-stats
5    djvm793      libdjvmti.so         cbGarbageCollectionFinish gc-finish
6    djvm793      libdjvmti.so                     cbThreadStart thread-start
7    djvm793      libdjvmti.so                      _method_exit method-return
8    djvm793      libdjvmti.so                          cbVMInit vm-init
9    djvm793      libdjvmti.so                       cbThreadEnd thread-end
10    djvm793      libdjvmti.so           cbMonitorContendedEnter monitor-contended-enter
11    djvm793      libdjvmti.so                     cbMonitorWait monitor-wait
12    djvm793      libdjvmti.so                     _method_entry method-entry
13    djvm793      libdjvmti.so                  track_allocation object-alloc
14    djvm793      libdjvmti.so                   cbMonitorWaited monitor-waited
15    djvm793      libdjvmti.so         cbMonitorContendedEntered monitor-contended-entered
16    djvm793      libdjvmti.so                      cbObjectFree object-free
17    djvm793      libdjvmti.so          cbGarbageCollectionStart gc-start
18    djvm793      libdjvmti.so                      cbObjectFree class-unload
19    djvm793      libdjvmti.so                         cbVMDeath vm-death
20    djvm793      libdjvmti.so               cbClassFileLoadHook class-load
#

```

I can enable these just like any DTrace probe. For example, if I want to see output whenever a method is entered, I could enable the `method-entry` probe. This probe has two arguments: the first is a pointer to the class name, and the second is a pointer to the method name. These are both in user-level, so you need to use `copyinstr()` to get at them. So, to see which methods that Tetris is calling, it's as simple as:

```
# dtrace -n djvm793:::method-entry'{printf("called %s.%s\n", copyinstr(arg0), co pyinstr(arg1))}' -q

```

Just dragging my mouse into the window containing Tetris generates a flurry of output:

```
called sun/awt/AWTAutoShutdown.notifyToolkitThreadBusy
called sun/awt/AWTAutoShutdown.getInstance
called sun/awt/AWTAutoShutdown.setToolkitBusy
called sun/awt/AWTAutoShutdown.isReadyToShutdown
called java/util/Hashtable.isEmpty
called java/awt/event/MouseEvent.<init>
called java/awt/event/InputEvent.<init>
called sun/reflect/DelegatingMethodAccessorImpl.invoke
called sun/reflect/GeneratedMethodAccessor1.invoke
called java/lang/Boolean.booleanValue
called java/awt/EventQueue.wakeup
called sun/awt/motif/MWindowPeer.handleWindowFocusIn
called java/awt/event/WindowEvent.<init>
called java/awt/event/WindowEvent.<init>
called java/awt/event/ComponentEvent.<init>
called java/awt/AWTEvent.<init>
called java/util/EventObject.<init>
called java/awt/SequencedEvent.<init>
called java/util/EventObject.getSource
called java/awt/AWTEvent.<init>
called java/util/EventObject.<init>
called java/util/LinkedList.add
called java/util/LinkedList.addBefore
called java/util/LinkedList$Entry.<init>
...

```

This is way too much output to sift through, so let's instead aggregate on class and method name:

```
#pragma D option quiet
djvm$1:::method-entry
{
@[strjoin(strjoin(basename(copyinstr(arg0)), "."),
copyinstr(arg1))] = count();
}
END
{
printa("%-70s %8@d\n", @);
}

```

Running this (providing our pid -- 793 -- as the argument), fooling around with Tetris for a second or two, and then `^C`'ing gave me this output:

```
String.equals                                                                 1
PlatformFont.makeConvertedMultiFontString                                     1
String.toCharArray                                                            1
PlatformFont.makeConvertedMultiFontChars                                      1
CharToByteISO8859_1.getMaxBytesPerChar                                        1
CharToByteISO8859_1.reset                                                     1
CharToByteConverter.setSubstitutionMode                                       1
SquareBoard.getBoardHeight                                                    1
X11InputMethod.deactivate                                                     1
ExecutableInputMethodManager.setInputContext                                  1
SquareBoard$SquareBoardComponent.redrawAll                                    1
SquareBoard.clear                                                             1
Figure.getRotation                                                            1
InputMethodManager.getInstance                                                1
Figure.rotateRandom                                                           1
FontDescriptor.isExcluded                                                     1
CharToByteISO8859_1.canConvert                                                1
PlatformFont$PlatformFontCache.<init>                                         1
InputContext.deactivateInputMethod                                            1
...
SquareBoard.access$100                                                      807
AWTEvent.getID                                                              883
Figure.getRelativeY                                                         959
X11Renderer.drawLine                                                       1168
SunGraphics2D.drawLine                                                     1168
Rectangle.intersects                                                       1246
SquareBoard$SquareBoardComponent.paintSquare                               1246
SunGraphics2D.getClipBounds                                                1284
X11Renderer.validate                                                       1352
SurfaceData.getNativeOps                                                   1352
Rectangle.translate                                                        1361
Figure.getRelativeX                                                        1383
Rectangle2D.<init>                                                         1516
RectangularShape.<init>                                                    1516
SurfaceData.isValid                                                        1620
SunGraphics2D.getCompClip                                                  1620
Rectangle.<init>                                                           2839
SquareBoard.access$000                                                     8019
SquareBoard.access$200                                                     8496

```

That's a lot of calls to the `Rectangle` constructor, so a natural question might be "where are calling this constructor?" For this, we can use the `djvm` provider along with the `jstack()` action:

```
djvm$1:::method-entry
/basename(copyinstr(arg0)) == "Rectangle" && copyinstr(arg1) == "<init>"/
{
@[jstack()] = count();
}

```

Running the above generated a bunch of output, with stack traces sorted in order of popularity; here was the most popular stacktrace:

```
libdjvmti.so`_method_entry+0x5c
com/sun/tools/dtrace/internal/Tracker._method_entry*
java/awt/Rectangle.<init>*
sun/java2d/SunGraphics2D.getClipBounds*
net/percederberg/tetris/SquareBoard$SquareBoardComponent.paintSquare*
net/percederberg/tetris/SquareBoard$SquareBoardComponent.paintComponent*
net/percederberg/tetris/SquareBoard$SquareBoardComponent.paint
net/percederberg/tetris/SquareBoard$SquareBoardComponent.redraw
net/percederberg/tetris/SquareBoard.update
net/percederberg/tetris/Figure.moveDown
0xf9002a7b
0xf9002a7b
0xf9002a7b
...

```

Note those Java frames: we can see exactly why we're creating a new `Rectangle`. We might have all sorts of questions from the above output. The one that occurred to me was "just how much work do we do from that `paint` method?" Answering this question is a snap in DTrace using bread-and-butter like thread-locals and aggregations:

```
#pragma D option quiet
self int follow;
int ttl;
djvm$1:::method-entry
/self->follow/
{
@[basename(copyinstr(arg0)), copyinstr(arg1)] = count();
}
djvm$1:::method-entry,
djvm$1:::method-return
/basename(copyinstr(arg0)) == "SquareBoard$SquareBoardComponent" &&
copyinstr(arg1) == "paint"/
{
ttl += self->follow;
self->follow ^= 1;
}
END
{
normalize(@, ttl);
printa("%35s %35s %8@d\n", @);
}

```

Running the above for a bit and `^C`'ing generated ~200 lines of output; here are the last 30:

```
CompositeGlyphMapper                  getCachedGlyphCode        4
SquareBoard$SquareBoardComponent                      getDarkerColor       14
SquareBoard$SquareBoardComponent                     getLighterColor       14
X11Renderer                            fillRect       15
SunGraphics2D                            fillRect       15
SquareBoard                          access$100       19
Color                            hashCode       28
Color                              equals       28
Hashtable                                 get       29
Color                              getRGB       44
SunGraphics2D                            setColor       44
PixelConverter$Xrgb                          rgbToPixel       45
SurfaceType                            pixelFor       45
SurfaceData                            pixelFor       45
SquareBoard$SquareBoardComponent                         paintSquare       65
Rectangle                          intersects       65
SunGraphics2D                       getClipBounds       66
Rectangle                           translate       67
RectangularShape                              <init>       69
Rectangle2D                              <init>       69
X11Renderer                            drawLine      115
SunGraphics2D                            drawLine      115
SurfaceData                        getNativeOps      130
X11Renderer                            validate      130
SunGraphics2D                         getCompClip      134
SurfaceData                             isValid      134
Rectangle                              <init>      137
SquareBoard                          access$000      191
SquareBoard                          access$200      238

```

This means that (on average) _each_ call to `SquareBoard$SquareBoardComponent`'s `paint` method induced 137 creations of `Rectangle`. Seeing this, I thought it would be cool to write a little "stat" utility that let me know how many of each kind of object was being constructed on a real-time, per-second basis. Here's that script:

```
#pragma D option quiet
djvm$1:::method-entry
/copyinstr(arg1) == "<init>"/
{
@[basename(copyinstr(arg0))] = count();
}
profile:::tick-1sec
{
trunc(@, 20);
printf("%-70s %8s\n", "CLASS", "COUNT");
printa("%-70s %8@d\n", @);
printf("\n");
clear(@);
}

```

Running the above gives output once per-second. Initially, it didn't provide any output -- but as soon as I moused into the Tetris window, I saw a (small) explosion of mouse-related object construction:

```
CLASS                                                                     COUNT
Region                                                                        0
RectangularShape                                                              0
Rectangle2D                                                                   0
Rectangle                                                                     0
LinkedList$ListItr                                                            0
SentEvent                                                                     0
LinkedList$Entry                                                              0
WindowEvent                                                                   0
InvocationEvent                                                              15
HashMap$Entry                                                                69
ComponentEvent                                                               74
InputEvent                                                                   74
MouseEvent                                                                   74
WeakReference                                                                78
AWTEvent                                                                     79
EventObject                                                                  79
Point2D                                                                     128
Reference                                                                   156
EventQueueItem                                                              158
Point                                                                       192

```

As soon as I move the mouse out of the window, the rates dropped back to zero (as you would hope and expect from a well-behaved app). When I started actually playing Tetris, I saw a different kind of ouput:

```
CLASS                                                                     COUNT
Dimension                                                                     7
HashMap$Entry                                                                 8
EventObject                                                                   8
AWTEvent                                                                      8
ComponentEvent                                                                8
InputEvent                                                                    8
Finalizer                                                                    14
AffineTransform                                                              14
Graphics                                                                     14
SunGraphics2D                                                                14
Graphics2D                                                                   14
FinalReference                                                               14
EventQueueItem                                                               16
KeyEvent                                                                     16
WeakReference                                                                24
Region                                                                       42
Reference                                                                    62
Rectangle2D                                                                 105
RectangularShape                                                            105
Rectangle                                                                   175

```

This means that we're seeing 175 `Rectangle` creations a second when I'm playing Tetris as fast as my [BattleTris](http://www.cs.brown.edu/people/mws/bt/bthome.html)-hardened fingers can play. That doesn't seem so bad, but you can easily see how useful this is going to be on those pathological Java apps!

Needless to say, the ability to peer into a Java app with DTrace is a very exciting development. To catch the fever, [download the JVMTI and JVMPI agents](https://solaris10-dtrace-vm-agents.dev.java.net/), head to your nearest [Solaris 10](http://www.sun.com/software/solaris/10/) machine (or [download Solaris 10](http://www.sun.com/software/solaris/get.jsp) if there isn't one), and have at it!

* * *

Technorati tags: [DTrace](http://technorati.com/tag/DTrace) [Java](http://technorati.com/tag/Java)
