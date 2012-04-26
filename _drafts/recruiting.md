

i know a guy running a software shop. he's an active participant in the local meetup community, and he cares about people. it's no surpise that he has all sorts of talented people trying to work with him, and not enough funding to hire them all. (at least, i think that's why he hasn't hired me ;)
i also know a few other people running software shops, who don't participate in the local meetup community. none of the best local developers even know who these other employers are. i'm sort of sick of hearing them complain about how hard it is to find talent.
the flip side of this, is that if I as an engineer want to have a steady stream of future opportunities with the best employers, I need to make myself known to them, by, you guessed it! making myself visible in the meetup community, and internet community at large.
ez game.

http://news.ycombinator.com/item?id=3779410
the thing a lot of these articles don't talk about is how to attract these employees to your startup in particular, because the class of person a startup wants to hire, can get hired by a lot of other startups too.
two factors, I think:
1. invest significant effort in networking in your local tech community. why should I want to work with you? how do I know you understand technology? speak at conferences and meetups, show up at hack nights, interact with the local talent so they can get to know you, and when they decide to switch jobs, they will approach you.
2. pick an interesting business with interesting tech. All engineers good and bad exist on a spectrum of value. The harder your technical problems, the more you need the very best engineers. If your problem is mostly not technical, you don't need as many rockstars, and they don't have to be quite as strong.
there was a recent YC job post for a "recruiting engineer" which made me groan inside. Your best engineers are the only ones who can attract other best-of-class engineers. typical recruiters, of course, are out of the question.


http://news.ycombinator.com/item?id=3669568
4 points by dustingetz 50 days ago | link | parent | on: How to Hire a Programmer

the challenges you describe are, i think, why networking is really the only viable way to find the best talent - self-selection can work in your favor, if you're doing it right.
One of the leaders here in the Philadelphia tech community, Kyle Burton, talks about networking on HN here[1] - he is running a Clojure startup and has had little difficulty hiring due to the time he spent cultivating the local community. he hosts his own clojure meetup which it appears he can hire from at will. edit: i just emailed him to belatedly ask permission to quote him, maybe he will chime in here.
"I was actively recruited to the team. I personally believe that this was because I made a conscious effort in the fall of 2008 to start being much more active in the community at large. I started doing more visibile things on the internet: a blog, putting up a sandbox on github. I also started speaking at local user groups (god bless them for listening to my first talks and my horrible, horrendous presentation skills). ...
"By 'recruited into' I mean that not in the sense that either side used a recruiter. I mean I was approached through my network because of the effort I put into building the netowrk and the effort I put into building a personal brand (as slimy as that sounds, it is working). ...
"Actively working at networking has been wonderful. I can't recommend it enough. I was involved in starting a "tech breakfast" meeting that takes place 1x a month. The local groups (esp volunteering to speak), the breakfast sessions, and buying people lunch (an hour of interesting conversation is totally worth the $10, and every time it has come back to me) has gone a long way to building a local network."
[1] http://news.ycombinator.com/item?id=2832571




62 points by dustingetz 50 days ago | link | parent | on: How to Hire a Programmer

John D Cook has a lot to say about this: [1]
"One of the marks of a professional programmer is knowing how to organize software so that the complexity remains manageable as the size increases. Even among professionals there are large differences in ability. The programmers who can effectively manage 100,000-line projects are in a different league than those who can manage 10,000-line projects. ... Writing large buggy programs is hard. ... Writing large correct programs is much harder."
Jeff Atwood's metrics will help you filter out engineers whose complexity ceiling is <1k lines -- StackOverflow answers, whoopee -- but that's not a terribly hard thing to interview for. Much harder to interview for the very best, the mythical 10x productivity programmers[2], those who can handle 100k LOC, 1M, or more. Perhaps this is the difference between an experienced non-expert and a real expert[3].
In my experience not a lot of employers care about this, perhaps because their challenges aren't those of complexity-in-scale, or perhaps because complexity hasn't bit them hard enough yet, or perhaps because they are "unconsciously incompetent"[4]. About the only hiring signal I've identified for this is interest in functional programming -- languages like Clojure and Scala exist precisely to raise the ceiling of complexity a human can handle[6] -- and as such I'm trying to learn this stuff and trying to find people via the community who care to hire engineers with these skills. Unfortunately my own bias may be blinding me, you never know which side of Dunning-Kruger[5] you're on until it's too late.
If you care about these things: I'd love to know who you are and what you're working on, email me.
[1] http://www.johndcook.com/blog/2008/09/19/writes-large-correc... [2] I am not one of these, but I strive to be one someday. [3] http://www.dustingetz.com/how-to-become-an-expert-swegr [4] http://en.wikipedia.org/wiki/Four_stages_of_competence [5] http://en.wikipedia.org/wiki/Dunning%E2%80%93Kruger_effect [6] Clojure creator Rich Hickey talking about complexity: http://www.infoq.com/presentations/Simple-Made-Easy




http://news.ycombinator.com/item?id=3668152
34 points by dustingetz 51 days ago | link | parent | on: Seth Levine: Sick of start-up BS

i work at a J2EE shop. We pivoted recently. We feel like a startup, we engineer like a startup, we work as hard as a startup, but we also have existing client relationships, existing revenue streams funding our pivot, and we already have a team of domain experts and strong engineers.
When I give a talk and my bio says J2EE I'm a bit sheepish, and all the twenty-somethings are like LOL you use java, but that's silly - I work for a better, stronger business than all the little startups[1] emailing me on careers.stackoverflow. And the money is better. And we dodge the whole class of issues that come with taking money[2], like being told by your VC to collect and sell personal information.
That's why I'm still here. I believe in this business, and when interviewees ask the founder about the business model, the answer isn't "freemium" - its how we've been paid to contract a product for a huge client but retain IP rights so we can reinvest the profit to productize something that the entire industry wants. and we have assets (domain experts, software, client relationships, functional sales team) that a startup doesn't have - there are tremendous barriers for competition to overcome. we're halfway through this, and it's working, and when you look at it you're like, duh, of course its working.
[1] i don't want to pick on anyone, and there are a few awesome startups out there that I'd love to work for, and maybe yours is in this set, and maybe i'm not good enough for the ones in this set. What I do see is that the startups emailing me seem to need a decent rails guy to build their commodity webapp. rails apps are important, and i'm sure they have all sorts of challenges which i'm not aware of, but they aren't particularly interesting, to me. J2EE on a good team is about building out < http://c2.com/cgi/wiki?WhyIsPayrollHard >, which is in a whole new class of scale/complexity that is, well, the type of hard which causes languages like Clojure and Scala to exist. [2] http://37signals.com/svn/posts/3107-99-problems-but-money-ai...



careful, some of those dime-a-dozen java devs are solving way harder problems than the hipsters and their rails CMSs. strong rails hipsters grow up to be VP of some ycombinator webapp. strong enterprise devs grow up to become VP at google.
