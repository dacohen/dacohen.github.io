---
layout: post
title: 'Technicians vs Engineers'
date: 2025-08-14 16:05:00 -0400
categories: technology
subtitle: ''
background: '/images/bridge.jpg'
---

Even though I work in Computer Science, I know a lot of engineers.

I purposefully differentiate these terms, because in many jurisdictions, being an engineer means something very specific. To be an engineer requires a four-year degree from an accredited institution, then a period of “apprenticeship” under a qualified engineer, followed by a rigorous examination. Only at the conclusion of this process is one a bona fide engineer. This qualification entitles one to be the final sign off on engineering drawings, guaranteeing their correctness.

In the United States, these people are called Professional Engineers, to differentiate them, I suppose, from the ostensibly amateur variety. Elsewhere, the very title “engineer” has legal ramifications the way “doctor” or “lawyer” does here.

I once met a Canadian engineer who showed me his [Iron Ring](https://en.wikipedia.org/wiki/Iron_Ring). These rings are given to all engineering graduates to be worn on the pinky finger of the working hand, so that it always rubs on their drawings, reminding them of their professional obligations.

If you work in an engineering discipline like civil or nuclear engineering, with high levels of public trust, where the consequences of a design mistake are often measured in the hundreds of human lives, this sort of nearly fanatical devotion to correctness truly matters.

It’s worth noting that plenty of Mechanical, Electrical, and Industrial engineers, to name a few, work their entire careers without being Professional Engineers, because their their work doesn’t incur the same risk to human life.

The National Council of Examiners for Engineering and Surveying (NCEES) which administers PE exams across the United States, trialed a track for Software Engineering in 2013, in an attempt to provide a path to a similar level of professional rigor for the software industry. When they chose to cancel the exam in 2019, a total of 81 candidates had taken the exam in the five years it was administered.[^1] In contrast, the US Bureau of Labor Statistics estimates that there are nearly 1.9 million “Software Developers” in the country.[^2] The exam was, suffice it to say, not popular. There are various reasons why this might be. An easy answer might be poor marketing to prospective test takers, or limited recognition on the part of employers. I graduated in 2016, and only learned it existed when it was canceled.

 A more difficult answer might be a culture in the broader engineering community of assuming the software will fail in safety-critical systems, and designing the system as a whole with this in mind. In this case, additionally shoring up the software would be of little marginal value.

Nonetheless, nearly all engineers, professional or otherwise, have a four-year degree. The expectation is that a high level of math and physics is a prerequisite to doing a good job. Someone once told me that an engineering education isn’t about learning the mechanics of being an engineer, it’s about learning how to think like one.

Engineers are assisted by technicians. Technicians almost universally don’t have a four-year engineering degree. Their role is to assemble components designed by engineers, and perform basic troubleshooting. A car mechanic is a technician, albeit not one directly supervised by an engineer.

It might be tempting to assume that this is some sort of elitism, or a holdover from feudalism like the distinction between soldiers and officers in most militaries, but like doctors and nurses they’re fundamentally different roles, with different skillsets. Occasionally a nurse will go to medical school, but this is not a typical career path by any means.

The software industry needs technicians. I want to stress that the technicians vs engineers “axis” is orthogonal to seniority; you can have junior engineers, and senior technicians, working side by side. In fact, I’d argue that the failure to make this distinction clear is part of why the industry can’t seem to agree on whether LLMs are actually useful.

Among the laypeople of the world, there’s a fundamental misunderstanding of what “Software Engineers” (or equivalent titles) actually do. Acquaintances frequently ask if I’m a “coder”. What I explain, with varying levels of success, is that code is a means to solving a business problem, not a solution in of itself. This might sound like a distinction without a difference, but most Software Engineers spend the majority of their time on activities besides actively typing out code on a keyboard. Worryingly, this misunderstanding seems to afflict many managers who indirectly oversee Software Engineering.

Imagine the head of a package delivery company, say 80 years ago, who observes his drivers spend their entire shift operating the steering wheel and pedals of their trucks. One day, he goes to a conference and learns about cruise control. He buys a new fleet of cruise control-capable trucks on the spot, thinking about just how much more efficient the fleet will be, if the drivers can focus on just steering. Returning with the trucks, he extols the virtues of the new technology, and the efficiencies it will bring. A few months later, he’s shocked to realize most of his drivers have never even touched the cruise control, because they spend half their day running in and out of buildings, and the other half trying not to hit anything. Operating the steering wheel and pedals is an essential part of the job, and lots of time is spent doing it, but it is not the limiting factor in package delivery.

This parable will likely sound familiar if you flash forward to the present day, and swap out “LLMs” (or even more non-specifically, “AI”) for “Cruise Control”. Software engineers spend most of their time converting poorly defined requirements and concepts to extremely specific ones that just happen to be expressed as “code”, and then iterating when that code doesn’t perform as expected.

When asked by a non-engineer to add, say, a new type of item to an application, an engineer will often ask a seemingly inscrutable question like “How many of these items will there be?” To the uninitiated, the connection is not clear, but the engineer asks because it has profound implications for how the data is modeled, and how it can be changed later as the application evolves. In my experience, LLM won’t ask this type of question, and will just assume, often to everyone’s detriment.

When code doesn’t perform as expected, it’s often not possible to just read the code and see where the problem is. A classic example is the “[500 mile email](https://www.ibiblio.org/harris/500milemail.html)”, where a university was unable to send emails to servers more than 500 miles away. The scientists reporting the issue were dismissed by the system administrators, until they did what scientists do, and ran an experiment to prove they were right. Armed with irrefutable evidence of this phenomenon, the sysadmins finally traced the problem to an overly aggressive timeout in the email service that was causing the university’s server to hang up preemptively if the remote server took too long to reply.

This issue is certainly more abstruse than most problems an engineer will typically encounter, but its characteristics are the same. Going into this problem, the protagonist had very minimal information (couldn’t send emails more than 500 miles), and nearly no prior art on which to rely. These are both areas where an LLM fails, yet the basic skills to tackle problems like these are necessary even for the most junior software engineer.

Where then, does the industry go from here? I believe the first step is to differentiate Software Technicians from Software Engineers when hiring and promoting. Software companies that customize Wordpress installations, for example, will want to hire almost exclusively technicians, whereas a company building a new database will hire primarily engineers.

Technicians aren’t “second class citizens” and shouldn’t be treated as such, it’s just a different role in the engineering organization. While many, if not most, software engineers will have college degrees, I don’t think this is as necessary as it is for traditional engineers.

This separation also helps clarify the role of LLMs. Some technician roles may be able to be replaced by LLMs, but many won’t. It also helps humans find the role that fits them, and cut down on the part of the interview cycle where the candidate figures out where on the technician-engineering continuum this particular role falls. Finally, this could be a catalyst for improving the interview processes for both roles. Technician interviews will likely focus more on the mechanistic aspects of software creation, such as writing algorithms and basic debugging, while engineering interviews can focus on design, organization and scalability.

Like the humans that populate them, every industry needs to adapt and change as it grows. Over 36 years out from the creation of the World Wide Web, I think software is in need of such a moment.

[^1]: [https://ncees.org/ncees-discontinuing-pe-software-engineering-exam/](https://ncees.org/ncees-discontinuing-pe-software-engineering-exam/)
[^2]: [https://www.bls.gov/ooh/computer-and-information-technology/software-developers.htm](https://www.bls.gov/ooh/computer-and-information-technology/software-developers.htm)
