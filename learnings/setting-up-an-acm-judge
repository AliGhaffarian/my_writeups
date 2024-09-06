it's not cool to have cheaters in acm compitetions so i volunteered to setup a fairly secure envirenmont to relief the pain

we had some options:
1_allow internet connection, no local judge server and run the acm in some 3rd party platform
	-need for firewall and ip filtering

2_run a local acm judge server and cut the internet connection, and monitor the universities network for any cheaters trying to get to internet
	-need monitoring scripts
	-need server
	
	
so we chose 2 because it was more flexible
the first time i tried to build and run [an open source acm judge](https://github.com/DMOJ/online-judge) i encountered tons of build errors probably of conficts with my already installed packages
why would you do something remotely dumb to that? you may ask
well because i evaded learning docker for sometime, and as you can tell it's time to embrace my fear of virtualization

i watched an [introduction to docker](https://www.youtube.com/watch?v=eGz9DS-aIeY&t=672s)
since i had some clue about docker because of CTFs i have some business with [Dockerfiles](https://docs.docker.com/reference/dockerfile)

watched this https://www.simplilearn.com/tutorials/docker-tutorial/what-is-dockerfile

from what i understood, a dockerfile is kinda an script that initializes your container to actually be usable

some attributes i used:
**FROM**
declare what image you are using
**RUN**
run command
**CMD**
when the container is runned run this

[what  is RUN --network](https://stackoverflow.com/questions/43316376/what-does-network-host-option-in-docker-command-really-do)

now i need to learn how to use docker to not blow up my dear arch build
i watched some

after some more studies i found out about [docker hub](https://hub.docker.com) and how people may have already done something im trying to

found [domjudge](https://hub.docker.com/r/domjudge/judgehost) project
ah, Semnan flashbacks

to get this thing up and running i needed to setup [cgroups](https://wiki.archlinux.org/title/Cgroups) version 1

## cgroups
Control groups (or cgroups as they are commonly known) are a feature provided by the Linux kernel to manage, restrict, and audit groups of processes.

because of cgroups v1 being deprecated (at least from what i understood at the time) i accidentally locked myself out of the linux machine, because linux refuses to boot if you try to boot with cgroupsv1 and dont have the force option set

modified the kernel parameters at the bootloader menu and fixed it

to be continued
