+++
title = "How I Org, A Software Engineer's Process"
author = ["Rod Morison"]
date = 2022-08-10T20:56:00-07:00
tags = ["productivity", "personal-process"]
draft = false
topics = ["Emacs", "Org-mode"]
description = "Emacs and Org-mode to track work, improve flow, and reduce (cognitive) stress"
images = ["img/task-state-cover.png"]
+++

## How I Org {#how-i-org}

I've encountered a half dozen or so personal productivity, task management, organization and motivation systems. (I will spare the list.) The latest I've read is David Allen's [Getting Things Done](https://gettingthingsdone.com/). Having been there and done this, I read Allen's book with a "Just take me to the algorithm" mentality.

Also, I create software for living. Software development is an occupation that is both creative and technical. It's also a craft that is mostly performed at a computer and is now more virtual than ever. A lot of text in _Getting Things Done_ covers tools and techniques that support the algorithm, but are incidental: there are many ways to implement, track, and execute Allen's techniques. A salesperson's needs here are different than a manager's, are different than a software engineer's, and so on.

My own personal organization needs are nearly full-virtual. Either I'm in a virtual meeting, preparing for one, working on tasks from one, or---and what I do best---designing and writing code and building software products. And finally, I spend as much time as possible in the Emacs editor. There are many, many code editors and IDEs out there.  Yet once you've paid the admittedly steep learning curve for Emacs, it's the most effective "flow tool" I know of. What I do in Emacs, including this very text, I can get lost in, which means maximum productivity.

By the way, I do not mash up personal todos with my work as GTD encourages. I prefer a simpler in-&gt;due?-&gt;done for this part of my life. For personal work I use [Todoist](https://todoist.com/) for it's clean UX, mobile app, and sync across devices. Todoist recently added [Kanban boards,](https://todoist.com/kanban-board) so I may toy with a GTD-esque flow sans Emacs (which is a fiddly challenge to sync across devices.) That notwithstanding, I'll continue with my Emacs setup.


### Why Emacs? {#why-emacs}

I admit, I've got years of Emacs key bindings in my fingers. But like playing the piano, practice makes plenty good. More keyboard, less mousing around many windows lends to good flow. I can now capture meeting notes, reference tidbits, fleeting ideas, and so on, with confidence I can forget them now, find them later, and fold follow ups into todo lists and projects plans.


### References {#references}

The following works were highly influential in my own productivity thinking and, of course, this article.

-   [Getting Things Done](https://gettingthingsdone.com/) by David Allen: This highly popular "work of the day" got me re-thinking about personal productivity again, and tweaking some of my own practice as a result.
-   [Getting Things Done: The Science Behind Stress-Free Productivity](https://www.researchgate.net/publication/222552899_Getting_Things_Done_The_Science_Behind_Stress-Free_Productivity) by Francis Heylighen and Clément Vidal: An academic paper on GTD that lays a foundation for what I think is the biggest return on investment for GTD: reducing cognitive stress and overload. No less important, the authors provide an extremely concise and on-point summary of Allen's method, including a comprehensive (wait for it) [flowchart of the algorithm](https://www.researchgate.net/profile/Francis-Heylighen/publication/222552899/figure/fig1/AS:304772235186176@1449674771358/a-flowchart-depicting-the-GTD-process-for-organizing-and-processing-incoming-stuff.png)! Recommended reading.
-   [Trying Not To Try](https://www.edwardslingerland.com/trying-not-to-try) by Edward Slingerland: This trade book is very thought provoking if you have interest in the convergence of history, culture, religion, and creativity. Of most relevance here is the connection to "flow state". Every section of Slingerland's book got me thinking about my own creativity, workplace and historical bias, and where I fit on the spectrum between Confucius and Laozi. _Wu-wei_ ftw!
-   [Org Mode Basics](https://www.youtube.com/watch?v=VcgjTEa0kU4) and [Organize Your Life with Org Mode](https://www.youtube.com/watch?v=PNE-mgkZ6HM) (videos): [System Crafters](https://systemcrafters.cc/) is a great source of live code videos and one of the best ways to get into Emacs.


## Emacs Config {#emacs-config}

If you're interested in my detailed Emacs config, i.e., `init.el` you can find my setup in my [GitHub dotfiles repo](https://github.com/rmorison/dotfiles). My Org-mode config started with [Organize Your Life with Org Mode](https://www.youtube.com/watch?v=PNE-mgkZ6HM). If you're new to Emacs or Org-mode, start there.


## A Walk-through {#a-walk-through}

The rest of this doc is a walk-through with animated GIFs to illustrate.


### It All Starts with a Meeting {#it-all-starts-with-a-meeting}

Let's start with a meeting, which was my original use-case. Alt-tab to Emacs, then `C-c c m` to open an entry in `meetings.org` from the [meeting template](https://github.com/rmorison/dotfiles/blob/main/org/templates/meeting.org). The template will ask who the meeting is with and what it's about. (There's also a 1-on-1 meeting template, with just the "who?" question, `C-c c 1`.)

{{< figure src="/ox-hugo/new-meeting.gif" caption="<span class=\"figure-number\">Figure 1: </span>Fave Music meeting with Lunis and Lucy" >}}

When the meeting is over and I'm ready to finish the capture I'd normally hit `C-c C-c`. Instead, I'll use `C-u C-u C-c C-c` to _follow_ the capture buffer into the filed location in `meetings.org`. I'll add a `TODO` manually, without a template, by typing `M-enter` to open a new outline node and then `TODO Learn Für Elise on piano`. Text files ftw!

{{< figure src="/ox-hugo/file-meeting.gif" caption="<span class=\"figure-number\">Figure 2: </span>File the meeting, create a task" >}}


### Check the Backlog, Schedule the Task {#check-the-backlog-schedule-the-task}

`C-c a a` brings up my agenda for the week. It's empty right now, nothing scheduled. `C-c a b` brings up my backlog view. I'll I see the "Learn Für Elise on piano" todo. Using GTD practice, I can either do this right away (2m rule, iirc) and make it DONE, move it to NEXT state, or realize it's too big for a single task and break it up.
\#+CAPTION Agenda check
![](/ox-hugo/check-backlog.gif)


### My Algorithm {#my-algorithm}

Now's a good time to highlight the tasks states I use and my typical path through them.

{{< figure src="/ox-hugo/task-state-cover.png" >}}


### Review Backlog, Schedule Work {#review-backlog-schedule-work}

`C-c a b` into my backlog view, I realize my task is a project, too big on its own. (I try to visit the backlog at least once a day.) I'm going to set the task to `BREAKDOWN-PLAN`. (For example, I have to get a keyboard, buy the sheet music, hire a music teacher) and schedule the planning work for tomorrow.

In the backlog view cursor to the "`TODO` Learn Für Elise on piano", use `C-c C-t` and choose the new state with `b`. Then, `C-c C-s` brings up the calendar. `S-→` navigates the calendar. I'll finish with `C-c a a` to double check my agenda.
![](/ox-hugo/schedule-planning.gif)


### Check Agenda, Plan Project {#check-agenda-plan-project}

Later, I'll check my agenda, `C-c a a`, arrow down to the `BREAKDOWN-PLAN` task, hit `ENTER` to jump to the task. I'll drop `TODO` entries right under that in the outline. I'll change that `BREAKDOWN-PLAN` state to `PLANNED`, then `C-x C-s` to save.
![](/ox-hugo/plan-project.gif)


### Refile Into Projects File, Set Deadlines, Tee up Tasks {#refile-into-projects-file-set-deadlines-tee-up-tasks}

Refile is the gem of Org-mode. I don't want to track and annotate my project in my meetings folder, that's not the way. I'm going to move the whole project outline to my projects folder.

`C-x C-f meetings.org` to open my meetings file, arrow to the `PLANNED` project, and `C-c C-w` to invoke Org-mode refile.
![](/ox-hugo/refile-project.gif)


### Prep and Schedule my Backlog {#prep-and-schedule-my-backlog}

Next, I'll set deadlines and move tasks to `NEXT` state. `C-c a b` to the backlog, `C-c C-t n` for `NEXT` state and `C-c C-d` to set deadlines, `C-c C-s` for scheduled dates. Then I hand edit the "Practice Weekly" task scheduled date for a 1 week [repeated task](https://orgmode.org/manual/Repeated-tasks.html). (Either you love text files, or you don't.)
![](/ox-hugo/schedule-backlog.gif)


### Add Reference Note, Link it in Project {#add-reference-note-link-it-in-project}

Finally I'll take some historical notes about Für Elise and put a link to those notes into the project outline. I open a reference note with `C-c c n` and save it with `C-u C-u C-c C-c`. The template asks for a title, then I can tap in text. I've bound `C-c l` to save an "anchor link" at the current point, and can then open `projects.org` and use `C-c C-l` to paste it in.
![](/ox-hugo/note-taking.gif)


## Features, Features, and more Features {#features-features-and-more-features}

Emacs Org-mode has a dizzying array of [features](https://orgmode.org/features.html) and this walk-through is only meant to capture the essence of how I org. I don't use all of these, but for reference, here goes...

-   [Tags](https://orgmode.org/manual/Tags.html)
-   [Habits](https://orgmode.org/manual/Tracking-your-habits.html)
-   [Time tracking](https://orgmode.org/manual/Clocking-Work-Time.html)
-   [Executing code blocks (Babel)](https://orgmode.org/worg/org-contrib/babel/)
-   [Exporting](https://orgmode.org/manual/Exporting.html) (say, to markdown or HTML) and [publishing](https://orgmode.org/manual/Publishing.html)
-   [Blogging and content sites](https://orgmode.org/worg/org-blog-wiki.html)

And of course, we're talking Emacs here. If you can code it in Elisp, you can do it.
