---
layout: post
title: Reusing Code Leads to.. Reusable Code
---

# Reusing Code Leads to.. Reusable Code

I know this sounds like something that came from the department of redundancy department, but every day I see myself and other developers I work with writing code that is too closely dependent on some framework or component and I often have trouble reusing large portions of our code because of this. There is nothing more frustrating than having all the functionality you need to accomplish something right in front of you, but not being able to use it because it is so tightly integrated into ASP.Net WebForms or some other technology that it would take longer to break the dependencies than it would to write it from scratch.

This weekend something clicked while I was trying to figure out how to work around some dependencies our API code has on WCF. I was struggling to reuse this code and finding it difficult. While I have yet to figure out how exactly to break the dependencies and use the code, something clicked and I realized that if the original authors committed to covering as much of this with unit tests as they could that this would have been a form of reuse that would have made it extremely simple for me to reuse the code, since it would have already been reused.

For me, this was somewhat pivotal because I also realized that writing unit tests is a form of reuse. This really hammered it home for me that unit tests are not just about the tests. They are just as much about the design feedback and the hard constraint for reusable code as they are for the assertions they contain.

So, the bottom line is if you want reusable code, write some unit tests to reuse it while you are developing the code and it will be easy to reuse it later when deadlines are tight and you need to.