---
title: "Wandering Models"
date: 2024-10-16
categories: agents
---
During the pandemic, I was looking into machine learning in my 'spare time'.  I was still working, so my time was limited.  I ran across this [fastai](https://course.fast.ai/) course, and the approach that [Jeremy Howard](https://www.youtube.com/@howardjeremyp) takes towards software development.  One way to describe his approach is in terms of cognitive load -- with fastai, fasthtml, fastlite -- these all make very complex things as simple as possible.

There is a huge amount of machine learning content, deep and detailed and mathematical and theoretical, throw in a library here and there, which requires all kinds of new concepts, which may or may not be related to theory or math or architecture.  Anyway, its overwhelming to take on as a side project, when you just want to experiment and do something like detect a bear in an image, which is in fact useful.   My brother-in-law gets security video notifications like "89% bear".   Fastai was the first thing that, for me, was understandable and usable.  

FastHTML and fastlite are similar -- simple, straightfoward, a practical way to accomplish a lot.  The other way to look at this approach is as a Deep Module as described in [A Philosophy of Software Design](https://a.co/d/3SOj8Um) -- the simple interface hides a lot complexity.   But it's really doing more than that.  It's repackaging the interface in a way  that makes it easier to consume for a normal human.

---

#### What Happened to Wandering Models???

So I've been developing using mostly Claude Project with fasthtml and fastlite content, which was recommended in https://www.youtube.com/watch?v=kfEpk6njb4s&t=16s.

Developing with both a fastHTML specific Claude Project and Cursor IDE.  The Claude Project works better for larger scale, more complex changes.  The Cursor IDE is easier to use, no cut-and-paste required, just review/accept.

But with both, the quality varies a huge amount.  Sometimes a code change reverts to something that is NOT fastHTML or fastlite.  All of a sudden we are doing a non-fastlite database connection, and writing SQL.  Or we suddenly introduce a FastAPI app for new features, and forgot all about the fastHTML application.  Whoops.

This seems like the same sort of problem the trustcall library is dealing with -- the model is being sloppy, forgetting things, so trustcall looks for this, and asks the model to fix all those problems.

There isn't anything doing that for the code generators (yet).
