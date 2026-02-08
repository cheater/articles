# Dev web of trust and ecosystem

## Where we are

AI push requests have gotten so bad and annoying, that people have started organizing active defenses against them:

- https://twitter.com/mitchellh/status/2020252149117313349
- https://github.com/mitchellh/vouch

Essentially, you're trawling through so much garbage to find value, that whatever value there is turns out not to be worth the effort.

## A little background

I've tried coming up with a very similar web of trust problem a while back when peer review controversies abound about a year or two ago. There's some deeper structure to this that isn't apparent at first glance.

## Keeping your identity should be important

One of the biggest problems is that people who get blacklisted shouldn't be able to perform a Sybil attack by restarting with a brand new identity. That means that identities must be non-trivial to create, and a matured identity (with work on it, not just time spent ageing, like a wheel of cheese) should be preferred by trust algorithms. It doesn't stop people from trying, but think of it like of reddit - there are subreddits where you need a minimum account age AND a minimum of karma in order to be able to post on them. The usual thing to do when creating an anon account on reddit is to go to /r/funny or some other sub where there's no minimum, post some karmaslop and wait for updoots, and only then post in your desired sub. While that doesn't prevent automated submissions, it frustrates them, and in a world where tokens cost money, automated creation of new identities would by force cost money as well. Going back to the dev web of trust: keeping a persistent identity implies keeping a ledger of actions through which your persona can be evaluated. And that ledger should either be public, or at least available to entities you want to work with, i.e. maintainers of repos you want to push to.

## Oracles and layered graphs of trust

There is also the possibility of assigning trust oracles, such as trusted first parties (eg notaries located in countries) that only hand out one identity per person. That means that a typical scammer is limited to as many identities as they can buy from grandmas and grandpas who don't push code but sure could use 20 bucks. This sort of thing is being done by gamers who cheat and get their identities burned in competitive games.

Another trust oracle could be companies. Companies that employ developers are a typical, natural proving ground for devs. An example of this for game developers is Mobygames which lists every person's credits in industry (non-indie) games. It's like the IMDB of video games. If you say you worked at Blixxxrd for 20 years as a developer, and on Mobygames you only have two tiny engineering titles that your dad provided you with, you are probably lying. A company could provide you with a trust token saying "Yes, Joe is a real person, they worked for us and we've interacted a lot and since we are a respectable company, you can Trust Me Bro that this person truly is a real person, and they do their job Well Enough to be entrusted to do a specific thing".

Having any sort of oracles implies a second-level trust graph. The first level, call it L1, is just normal humans (and the bots that snuck in). The second level is the network of oracles that can vouch for each other, or not. Alternatively, you can have vouches for all L2 oracles be provided only by a single (or small amount) of L3 oracles.

You can have multiple L2s, for example L2_notaries, L2_startups, L2_whatever. Then on top of those would be some sort of regulatory body that can introduce or kick out members of a specific L2, for example L3_notaries would be a global entity (or network thereof) that can kick out notaries from shady places for handing out bogus papers, as is known to happen in real life. There could be L4_notaries and L5_notaries and so on. There would be a governance structure, and probably a different one per separate L2. For example L2_notaries might have a governance structure that involves actual country-specific government entities, whereas L2_YCstartups might be governed by investors. Governance structures would be able to repudiate all the trust provided by a certain entity within their remit.

## Degenerated webs of trust and trust collapse

The next issue is trust collapse. A typical web of trust will be sparsely connected, there will be a lot of nodes connected by 1-2 edges and then central nodes that hand out trust, essentially working as oracles. That is a vulnerability - it's vulnerable to what is essentially a supply chain attack, but in the supply chain of trust rather than of technology. A single central node lowering its standards, changing hands, or becoming compromised can collapse the web of trust for a complete neighborhood around a group of projects utilizing trust. It is important to come up with schemes that 1. disincentivize handing out too much trust (but allow it where necessary) 2. punish badly allocated trust (but not draconically and irreversibly so, as errors happen to everyone, especially if they do a thing a lot).

## How do you build interop?

You probably want some sort of p2p network. Ultimately a cryptographically signed, unfalsifiable, immutable, public ledger of trust operations and trust violations holding history for every identity is going to be the optimum that everything will tend towards. Yes, one solution is a Merkle tree, no, it does not have to be a crypto coin, but yes, it could enable a developer economy where there doesn't exist one currently, but that's beside the point - the important part is immutability and public openness. With a full history of actions every entity can employ their own analysis algorithms in order to assess exposure and whether they do or do not want to interact with a (person, institution, ...) holding a certain identity.

## Some issues

I think Mitchell Hashimoto created an interesting first step towards solving the problem. However, it has some issues. That is no slight, as what he built in a short while is a million times better than no one having built anything at all, but I can see some problems. I am 100% sure Mitchell will improve on his system, and I think discussing the issues is the first step towards that. However, ultimately, I envision a somewhat differently implemented system, which makes things more explicit.

- There is no interop, so you'll have to manually copy-paste allow/block lists between repositories, and it's all fragmented and will run out of sync and that's a pain
- People will converge towards keeping shared block lists that anyone can end up on, which is essentially oracles, but they're not expressed explicitly as a first-class term in the system, they're instead an implicit process which is not introspectable (you can see the final block list, but have no idea how someone ended up there or why). While you can use other repositories' vouch lists, I have a feeling a lot of projects will be collating lists behind the scenes; meanwhile repositories that are used by a lot of third parties for vouch-import are essentially oracles without being given thought of their special role in the trust graph.
- There will be no accountability for just putting people you don't like on a block list, and this will be ripe for terrible abuse. Shared block lists are a festering breeding ground for controversy as block lists on Twitch have proven time and again. There will be no way to find out who put someone on a block list and there will be no way to extract a repair of this. Ultimately, this is a situation that can and will end up in civil lawsuits, e.g. defamation lawsuits. On-system governance would solve this, as it would provide methods for creating systems of transparency and accountability. The only accountability a flat file in a repo supports is unfortunately "Well I felt like it, don't like it? Scram" as there will always be come people who have absolute access to write to a repo and even rewrite its history, e.g. to re-write why some person was blacklisted, in order to manipulate discourse.
- A user name can be protected data, so GDPR says that people can request removal of their identity from such a list, and you HAVE to comply, therefore making a block list ineffective.
- There isn't really an identity other than a user name, and there is no good reason not to start over if you've misbehaved, i.e. if you start misbehaving, there's no reason for you to clean up your act - it's easier to just start a new GH account. That means that the system will degenerate to a deluge of "seemingly new" identities asking for vouches, and you're back to square 1 of having to trawl through tonnes of garbage to find a one-dollar bank note, and again it might turn out not to be worth it.
- There is no central information service about an identity's deeds and misdeeds, so profiling whether they're a bad actor requires piecing together data spread across bug trackers, commit histories, and other disparate data sources that have issues such as suddenly becoming unavailable, or with past entries being manipuated by owners of those third-party systems (due to lack of immutability).
- Since there is no way to deeply inspect a person's history of trust actions, you have to take other entities' vouches for that person at face value; there is no deep understanding to be had. Deep understanding could help a lot to differentiate between good and bad actors which is the biggest problem of such a system, therefore methods for deep understanding of an identity's history should be promoted and as many methods of storing and understanding such information as possible should be made available.

## Learn from and avoid past mistakes

As there is governance and there is a ledger, it is a very good idea to implement on-system governance and on-system rule mutability and first-class interop with L2's (and higher layers). On-system mutability (i.e. being a nomic) is important for governance without hard forks and drama surrounding people refusing or failing to upgrade. I'm biased, since I worked on it, but Tezos is doing exactly that, so it could at least provide some answers on how such things could be done successfully. Tezos has the second-system advantage of learning from the failure of others who came before (essentially Bitcoin and Ethereum), and they continue to do some pretty good things. Learn from it or just wholesale use it, but by no means build something that makes mistakes that Tezos avoids.

## Not just for devs

Webs of trust are important in other situations as well, not just for devs. Trust should be separable, but should also be holistic. Someone I'd trust to clean my carpet is not someone whom I'd trust to perform eye surgery... and vice versa, probably, therefore trust should be separable for both. However, if someone is trusted to e.g. drive a car, but also is trusted to work as a line chef, and after a history of good chef-being they suddenly fall out of trust, then their ability to drive a car should also be put into question in some situations - as it might indicate a deeper issue that does not limit itself to just the capability to cook a good pasta. 

## Escape hatches

Ethical questions abound whether such absolutist panopticon is acceptable or whether a person should be allowed to compartmentalize their life and make mistakes in one part of their life while they continue to do well in other parts. After all, no scoring system is ever fully just - and in fact, I expect this sort of thing to be very prone to failure. We're dealing with real humans scoring real humans, either by hand or by using algorithms that probably haven't been taught every special circumstance, and all of that adds to the probability of essentially random failure. Therefore there should be escape hatches built into governance. In real life, you can appeal a parking ticket, you can defend yourself if someone accuses you of theft, etc. This should be reflected in governance. While no court is fully just, additional scrutiny on contentious situations does increase the likelihood of success. However, a person should not be able to just blanket file any trust violation into a moderation queue - they should have to wager something, either money or trust score. This ought to discourage abuse of second-opinion systems.

Ultimately a complaints court is yet another oracle, so we're not adding a new structure here - it's nice to be able to loop around to ground that's already been covered.

## Dev ecosystem

An open source dev ecosystem could be built where webs of trust could be used to distribute money among contributors. How that would work exactly can be decided, either trickle-down or some other algorithm. Conversely, entities with high trust could offer their services to low-trust entities: you want code pushed to a repository, I have high trust there, pay me to read over your code and ensure you're not doing anything sneaky and I will push it in for you, and now your touchpad driver is in the kernel tree. This feels like it could be a more overt, less annoying way to do FOSS commercially. Users who want features could do fund raisers, companies could have a direct way of paying contributors.

## Aside

I'm always open to work as a dev, IC, EM, or C-level, so if you think my ideas are good, drop me a note on my GH commit email or at https://twitter.com/PLT_cheater

