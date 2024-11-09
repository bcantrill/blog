---
title: "On Modalities and Misadventures"
date: "2008-11-16"
categories: 
  - "fishworks"
---

Part of the design center of [Fishworks](http://blogs.sun.com/fishworks) is to develop powerful infrastructure that is also _easy to use_, with the browser as the vector for that usability. So it's been incredibly vindicating to hear [some of the initial reaction to our first product](http://weblog.infoworld.com/venezia/archives/018859.html), which uses words like "truly breathtaking" to describe the interface.

But as much as we believe in the browser as system interface, we also recognize that it cannot be the only modality for interacting with the system: there remain too many occasions when one needs the lightweight precision of a command line interface. These occasions may be due to usability concerns (starting a browser can be an unnecessary and unwelcome burden for a simple configuration change), but they may also arise because there is no alternative: one cannot use a browser to troubleshoot a machine that cannot communicate over the network or to automate interaction with the system. For various reasons, I came to be the one working on this problem at Fishworks -- and my experiences (and misadventures) in solving it present several interesting object lessons in software engineering.

Before I get to those, a brief aside about our architecture: as will come to no surprise to anyone who has used our applaince and is familiar with browser-based technologies, our interface is AJAX-based. In developing an AJAX-based application, one needs to select a protocol for client-server communication, and for a variety of reasons, we selected [XML-RPC](http://en.wikipedia.org/wiki/XML-RPC) -- a simple XML-based remote procedure call protocol. XML-RPC has ample client support (client libraries are readily available for JavaScript, Perl, Python, etc.), and it was a snap to write a C-based XML-RPC server. This allowed us to cleanly separate our server logic (the "controller" in [MVC](http://en.wikipedia.org/wiki/Model-view-controller) parlance) from our client (the "view"), and (more importantly) it allowed us to easily develop an automated test suite to test server-side logic. Now, (very) early in our development I had written a simple Perl script to allow for us to manually test the server-side that I called "aksh" -- the appliance kit shell. It provided a simple, captive, command-line interface with features like command history, and it gave developers (that is to say, [us](http://blogs.sun.com/fishworks)) the ability to manually tickle their server-side logic without having to write client-side code.

In part because I had written this primordial shell, the task of writing the command line interface fell to me. And when I first approached the problem it seemed natural to simply extend that little Perl script into something more elaborate. That is, I didn't stop to think if Perl was still the right tool for what was a much larger job. In not stopping to reconsider, I was committed a classic software engineering blunder: the problem had changed (and in particular, it had grown quite a bit larger than I understood it to be), but I was still thinking in terms of my existing (outmoded) solution.

As I progressed through the project -- as my shell surpassed 1,000 lines and made its way towards 10,000 -- I was learning of a painful truth about Perl that many others had discovered before me: that it is an undesigned dung heap of a language entirely inappropriate for software in-the-large. As a coping mechanism, I began to vent my frustrations at the language with comments in the source, like this vitriolic gem around exceptions:

```
eval {
        #
        # We need to install our own private __WARN__ handler that
        # calls die() to be able to catch any non-numeric exception
        # from the below coercion without inducing a tedious error
        # message as a side-effect.  And has it been said recently that
        # Perl is a trash heap of a language?  Indeed, it reminds one
        # of a reeking metropolis like Lagos or Nairobi:  having long
        # since outgrown its original design (such as there was ever
        # any design to begin with), it is now teeming but crippled --
        # sewage and vigilantes run amok.  And yet, still the masses
        # come.	 But not because it is Utopia.	No, they come only
        # because this dystopia is marginally better than the only
        # alternatives that they know...
        #
        local $SIG{'__WARN__'} = sub { die(); };
        $val = substr($value, 0, length($value) - 1) + 0.0;
};

```

(In an attempt to prevent roaming gangs of glue-huffing Perl-coding teenagers from staging raids on my comments section: I don't doubt that there's a better way to do what I was trying to achieve above. But I would counter that there's also a way to live like a king in Lagos or Nairobi -- that doesn't make it them tourist destinations.)

Most disconcertingly, the further I got into the project, the more the language became an impediment -- exactly the wrong trajectory for an evolving software system. And so as I wrote more and more code -- and wrestled more and more with the ill-suited environment -- the feeling haunting me became unmistakable: _this is the wrong path_. There's no worse feeling for a software engineer: knowing that you have made what is likely the wrong decision, but feeling that you can't revisit that decision because of the time already spent down the wrong path. And so, further down the wrong path you go...

Meanwhile, as I was paying the price of my hasty decison, [Eric](http://blogs.sun.com/eschrock) -- always looking for a way to better test our code -- was experimenting writing a test harness in which he embedded [SpiderMonkey](http://www.mozilla.org/js/spidermonkey/) and emulated a DOM layer. These experiments were a complete success: Eric found that embedding SpiderMonkey into a C program was a snap, and the end result allowed us to get automated test coverage over client JavaScript code that previously had to be tested by hand.

Given both Eric's results and my increasing frustrations with Perl, an answer was becoming clear: I needed to rewrite the appliance shell as a JavaScript/C hybrid, with the bulk of the logic living in JavaScript and system interaction bits living in C. This would allow our two interface modalities (the shell and the web) to commonalize logic, and it would eradicate a complicated and burdensome language from our product. While this seemed like the right direction, I was wary of making another hasty decision. So I started down the new path by writing a library in C that could translate JavaScript objects into an XML-RPC request (and the response back into JavaScript objects). My thinking here was that if the JavaScript approach turned out to be the wrong approach for the shell, we could still use the library in Eric's new testing harness to allow a wider range of testing. As an aside, this is a software engineering technique that I have learned over the years: when faced with a decision, determine if there are elements that are common to both paths, and implement them first, thereby deferring the decision. In my experience, making the decision after having tackled some of its elements greatly informs the decision -- and because the work done was common, no time (or less time) was lost.

In this case, I had the XML-RPC client library rock-solid after about a week of work. The decision could be deferred no longer: it was time to rewrite Perl functionality in JavaScript -- time that would indeed be wasted if the JavaScript path was a dead-end. So I decided that I would give myself a week. If, after that week, it wasn't working out, at least I would know why, and I would be able to return to the squalor of Perl with fewer doubts.

As it turns out, after that week, it was clear that the JavaScript/C hybrid approach was the much better approach -- Perl's death warrant had been signed. And here we come to another lesson that I learned that merits an aside: in the development of DTrace, one regret was we did not start the development of our test suite earlier. We didn't make that mistake with Fishworks: the first test was created just minutes after the first line of code. With my need to now rewrite the shell, this approach paid an unexpected dividend: because I had written many tests for the old, Perl-based shell, I had a ready-made test suite for my new work. Therefore, contrary to the impressions of some about test-driven development, the presence of tests actually _accelerated_ the development of the new shell tremendously. And of course, once I integrated the new shell, I could say with confidence that it did not contain regressions over the old shell. (Indeed, the only user visible change was that it was faster. Much, much, much faster.)

While it was frustrating to think of the lost time (it ultimately took me six weeks to get the new JavaScript-based shell back to where the old Perl-based shell had been), it was a great relief to know that we had put the right architecture in place. And as often happens when the right software architecture is in place, the further I went down the path of the JavaScript/C hybrid, the more often I had the experience of new, interesting functionality simply falling out. In particular, it became clear that I could easily add a second JavaScript instance to the shell to allow for a scripting environment. This allows users to build full, programmatic flow control into their automation infrastructure without ever having to "screen scrape" output. For example, here's a script to display the used and available space in each share on the appliance:

```
script
        run('shares');
        projects = list();
        printf('%-40s %-10s %-10s\n', 'SHARE', 'USED', 'AVAILABLE');

        for (i = 0; i < projects.length; i++) {
                run('select ' + projects[i]);
                shares = list();
                for (j = 0; j < shares.length; j++) {
                        run('select ' + shares[j]);
                        share = projects[i] + '/' + shares[j];
                        used = run('get space_data').split(/\s+/)[3];
                        avail = run('get space_available').split(/\s+/)[3];
                        printf('%-40s %-10s %-10s\n', share, used, avail);
                        run('cd ..');
                }
                run('cd ..');
        }

```

If you saved the above to file named "space.aksh", you could run it this way:

```
% ssh root@myappliance < space.aksh
Password:
SHARE                                    USED       AVAILABLE
admin/accounts                           18K        248G
admin/exports                            18K        248G
admin/primary                            18K        248G
admin/traffic                            18K        248G
admin/workflow                           18K        248G
aleventhal/hw_eng                        18K        248G
bcantrill/analytx                        1.00G      248G
bgregg/dashbd                            18K        248G
bgregg/filesys01                         25.5K      100G
bpijewski/access_ctrl                    18K        248G
...

```

(You can also upload SSH keys to the appliance if you do not wish to be prompted for the password.)

As always, don't take our word for it -- [download the appliance](http://www.sun.com/storage/disk_systems/unified_storage/resources.jsp) and check it out yourself! And if you have the appliance (virtual or otherwise), click on "HELP" and then type "scripting" into the search box to get full documentation on the appliance scripting environment!
