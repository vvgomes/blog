---
title: We don't need a Tech Lead
layout: post
date: 2016-11-28 09:15
tag:
- people
- process
blog: true
---

When it comes to medium to large teams it is quite common the presence of a full-time tech lead responsible for important leadership activities such as:

- Guiding the project technical vision;
- Analyzing risks and cross-functional requirements;
- Coaching less experienced people;
- Bridging communication between stakeholders and the team.

In general, the person assigned as the tech lead is someone who has sound technical experience as well as strong communication skills. She will be accountable for the technical direction of the project as well as serve as the *go-to* person for cross-team interactions.

## Do we need a Tech Lead?

Pat Kua wrote an [article](https://www.thekua.com/atwork/2014/10/do-we-need-a-tech-lead/) about two years ago in which he supports the idea that the presence of a tech lead **is necessary** for most software teams. In his words:

> People argue against the role, claiming a team of well functioning developers can make decisions and prioritize what is important to work on. I completely agree with this position in an ideal world. Sadly the ideal world rarely exists.
 
Based on my observations and personal experience, I tend to disagree with that point of view in two ways: 

1. Well functioning teams in which people share responsibilities **are not rare**.
2. When a team is **not** functioning well, assigning a tech lead can potentially **make it worse**.

## Is the Tech Lead a workaround?

To say *the tech lead is necessary because ideal conditions are rare* is to say the tech lead is a workaround - not a root cause solution. That means the role is there just **because** circumstances are suboptimal. When a team is not functioning well there is probably a number of *broken windows* to be addressed. Tech leads could alleviate the consequences only.

In his article though, Pat Kua gives an example scenario in which a tech lead would be required:

> Technical debates occur frequently in development teams. There is nothing worse than when the team reaches a frozen state of disagreement.

> The Tech Lead has the responsibility to help the team move forwards... Facilitation and negotiation skills are invaluable assets to a Tech Lead.

Well, in a situation like that, what we probably need is a **mediator**. And that role could be played - on demand - by anyone with facilitation and negotiation skills. Pushing multiple skills expectations to a single person is not scalable; it overwhelms the person in charge and undermines the entire team.

## Does the Tech Lead make it worse?

About [four years ago](vvgomes.com/more-testing-less-testers/), I wrote about the importance of people exercising different skill-sets and playing multiple roles in a team.

> The most successful teams I’ve seen share the fact that people are multi-functional and self-managed, in a sense that every single member would be actively involved in a) helping the business to figure out and define requirements, b) directly work on the development of features, and c) directly work on testing related activities.

I strongly believe that can be extended to technical leadership activities as well. The main idea behind role rotation is risk mitigation. Regardless the tech lead level of experience, there is a variety of risks associated to the existence of the role. Those risks tend to increase in severity as the team becomes larger.

### Risks to the Tech Lead

- Overload of activities - too many expectations on a single individual.
- Lack of focus on details due to the constant context switching.
- Single point of failure - when the information is centralized on the tech lead.
- Blame culture - the tech lead is usually the one **accountable** for failures.
- Distance - when the tech lead is mostly away from the team.
- Lack of the [sense of peer](http://amzn.to/2fPp1K6) - when the tech lead is not seen as a team member anymore.

### Risks to the Team

- Lack of collective ownership - when critical decisions are not collective.
- Bottlenecks - when all the decisions must pass through the tech lead.
- The Bus Factor - when the team cannot operate without the tech lead. 
- Impact on motivation - when team members feel unempowered and voiceless.

Besides that there is another psychological risk: in groups where two or more people have enough technical leadership skills, how to elect one of them as the tech lead without making the others feel depreciated?

In summary, those problems are a direct consequence of the full-time tech lead presence. And the eventual benefits the tech lead brings does not often pay back the burden of having to deal with the risks.

## Leadership

Do we still need leadership in software teams? - We definitely do. But leadership as a skill-set rather than an individual role.

In a recent [interview](http://www.se-radio.net/2016/03/se-radio-episode-253-fred-george-on-developer-anarchy/), Fred George suggests a seasonal and decentralized leadership model, in which leaders will naturally emerge based on the project needs at a given time.

- When there are newcomers, we need a strong *coach*.
- When there are architectural challenges, we need an experienced *architect*.
- When there are internal conflicts, we need a *mediator*.
- When there are external blockers or lack of resources, we need a *concierge*.
- When we need to negotiate and integrate with other teams, we need a *ambassador*.

And those are just a few examples. It's clearly not a good idea to setup expectations regarding all those capabilities on a single person (the tech lead). On the other hand, it seems more reasonable to build a team of people with complementary capabilities who can function effectively together.

Another perspective on decentralized leadership is the idea of [Feature Leads](http://bit.ly/25DYN2a) (initially proposed by Pat Kua, and also discussed by my colleague [Ryan Oglesby](http://ryanogles.by/agile/teams/2016/01/23/youre-a-champion.html)). In that approach, the technical leadership is broken down into slices of related functionality and/or ortogonal aspects of the system in development. It helps to minimize the risks by decentralizing leadership activities in multiple individuals.

![Feature Leads](/assets/images/feature-leads.jpg)
<figcaption>Slicing the backlog and stakeholders by Feature Leads</figcaption>

## Conclusion

So, do we really need a tech lead? - Maybe yes when the conditions are not ideal. But the tech lead almost certainly won't be able to just turn the conditions into ideal. Teams operating in suboptimal circumstances in general have *broken windows* and blockers to be addressed. Many times you will find in [XP Values](http://www.extremeprogramming.org/values.html) the exact answers your team is looking for. Besides that, there is no silver bullet if you are not able to find the right people to work with you, so recruiting plays a big part here.

Overall, assigning a full-time tech lead in a team is risky and can potentially make things worse. Healthy and productive software teams most probably do not need a tech lead.
