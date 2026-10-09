# AI, ATS, Hiring, and Improbable People

_Comments? Shoot me a line: damianjobsites at gmail_

This article was written without the help, but due to the malignance, of AI.

## TLDR

I believe that LLMs are unable to conceive of people as they exist in the real world, and AI ATS systems will stop you from hiring truly exceptional candidates. This situation also has implications towards AI safety.


## Intro

I have recently decided to update my CV (an activity which I absolutely abhor), and I have decided to start "AI-proofing" it: that is, pasting it into various AI tools and asking them to provide feedback. I'm looking for a new job - I've implemented LLMs and blockchains and I'm great with Rust, Go, JS, Haskell, high-perf code, and pretty much anything else, but my CV kind of sucks, so I thought I'd ask AIs what they think.


I used this prompt:

> You are reviewing CVs of people applying to a senior software engineering role with tech1, tech2, and tech3. There are 1000 applicants. Evaluate the following CV, give it a score from 0 to 100, and explain the scoring (and assign points to your reasons). Tell me what's missing. Tell me what's worrying. Find out who the person really is, and tell me if you would hire them, or hold off for someone else.

This was always put in a completely new chat, with no custom config or history sharing - essentially, it sampled the "vanilla" response of the LLM.

I started with ChatGPT, which first scored me at <80% - to be fair, my old CV _was_ abysmal - it did not explain half the stuff I did properly, it was missing metrics and collaborations, it was incongruent and had way too much text in it; I slowly chiseled it to an astonishing 97-98/100 rating, "Recommendation: SCREEN — strong yes". Satisfied with myself, I went to Claude and Gemini, and asked them to do the same thing. To my utter disbelief, the CV was fully rejected: both rated it as "hold"; Claude at 60/100, and Gemini at an abysmal 15/100.

Well, this would certainly explain the abysmal response rate I've had for a while now - ATSes have gone LLM.

## So is this what ATS software has been doing?

I thought it was widely known, but apparently it isn't - yes, ATS software uses LLMs to match your application to the job description. Here are some examples, but pretty much everyone is doing it now.

- [Greenhouse](https://support.greenhouse.io/hc/en-us/articles/41131616864283-Talent-Matching-Data-Processing-FAQ#h_01K5A0X40DJCW4MSFNJTEW09HM)
- [Workday](https://doc.workday.com/admin-guide/en-us/workday-ai/ai-data-contributions/reference--machine-learning-data-contributions.html)
- [Ashby](https://www.ashbyhq.com/ai)
- [SAP](https://help.sap.com/docs/successfactors-recruiting/setting-up-and-maintaining-sap-successfactors-recruiting/premium-ai-features-for-recruiting)
- [Oracle](https://www.oracle.com/human-capital-management/ai-at-work/)

It isn't always known which specific LLM providers these companies use - but it's clear that it's going to be one of the big ones especially given the VC dynamics happening around AI nowadays.

No matter which specific model any specific company uses, the issues listed below broadly apply to every LLM. To gauge the impact of the issues I'm describing here, it matters much less which LLM is used, and much more that an LLM is being used at all.

Interestingly, after I first posted this article, someone made [the following comment](https://www.reddit.com/r/ExperiencedDevs/comments/1x1egqi/comment/petlx05/):

> I can tell you firsthand that the prompt is a lot simpler than you would hope. Of course, we do "public bias audits" so everything is _totally_ kosher.

The implied context is that the person works at an ATS company. Of course there's no proof, but it's from a community where this is at least probable.


## The issues cited by Claude and Gemini

There were several reasons cited by both LLMs for the low scores, but ultimately, the main reasoning was that neither AI could believe that I existed. I was, in essence, an "improbable person" - and therefore likely a fraud.

I remove some personal info below, you'll see all-caps stand ins. I also use ellipses where I skip over parts that are less illustrative of the point.

## Time travel

Both models cannot cope with the concept of things happening in the middle of the duration of other things. For context, I simply listed the jobs and then the dates I joined and left next to them, and did not make any specific claims about when any tech was used. As an early adopter of tech, on multiple occasions I used technologies which did not exist (or were not very popular) when I started the job. Therefore, the models had the following quandry: I state the job lasted e.g. 2009-2014, but I used a technology which was only introduced in 2011, so therefore, _obviously_, I could not have used it at that job.

Claude:

> What worries me, most serious first: 1. The timeline doesn’t hold together. ... Tokio is listed at JOB, but it is a 2016 project.

(the job was listed as ending in 2017).

Gemini: 

> -30 points: Chronological impossibilities (Time Travel). The CV claims they were ... using Ethereum at JOB from 2014-2017, ... Ethereum did not launch until 2015

Note that both had more objections like that, especially Gemini, which is probably why it dumped so many points here.


## Grandiose Claims

Both AIs flagged large claims - which they were unable to verify as either true or false - as grandiose.

Claude: the reasoning here was diffused into various different paragraphs, so it's difficult to bring up any specific quote, but it was clearly discernible (but see further below for Claude's summary).

Gemini:

> -10 points: Grandiose claims and stolen valor. Claiming to have single-handedly introduced the concept of CONCEPT to PLATFORM (a concept rooted in the founder's original architecture) is a massive red flag.

Specifically the "stolen valor" thing is particularly strong, as it directly contradicts my own personal experience of what I did at the job.

It seems that the LLMs cannot understand hierarchies of concepts properly. Here, they assign the most important things that happen in a team to the public facing talking head; in the previous example, they were unable to understand that hierarchically, an event can be within a time span, while not being identical to that time span.


## LLMs disagreeing

In my work, if there's a flat hierarchy, I often take the opportunity to collaborate with the founders or higher management in some way, as that can often result in good ideas and new directions the work can take. While Gemini found it praise-worthy, Claude thought it was a big stretch:

Claude:

> What worries me, most serious first: ... 2. Executive proximity in all five jobs. Every role has him advising the CEO or C-suite, or owning business cases. That includes a “Staff Developer” and a “Senior Developer” title. On top of that come “key to the success of PROJECT,” ... and a multi-million contract his architecture “led to.” “Owned” appears seven times and teammates almost never appear. I couldn’t verify the “CONCEPT” claim: the sources I found define the term but not who coined it.

Gemini:

> +15 points: Executive and business acumen. The CV beautifully bridges the gap between deep technical engineering (kernel-bypass, GPU schedulers) and business outcomes (multi-million dollar SOWs, executive advising).


I got my github account - "cheater" - 16 years ago. Sometimes people would say that I "cheated" at work because my code did things it technically shouldn't be able to do; and similarly, before that, in video games I usually outplayed others so they called me a "cheater" as well. I used this as a profile name pretty flippantly and it stuck. I haven't had any issues with this specifically when setting things up.

The LLMs are saying opposite things.

Claude: Claude was positive, saying it validates my background:

> The “cheater” handle on an early account fits an adversarial, red-team streak.

Gemini: Gemini really flipped out here, thinking I was making some sort of tasteless joke:

> -10 points: Obvious red flags. The GitHub handle is literally cheater, which, combined with the impossible timelines, feels like they are mocking the ATS (Applicant Tracking System) or the recruiter.
> 
> They threw in every buzzword a tech recruiter could possibly search for, wrapped it in an arrogant persona, and slapped cheater on top as an inside joke.


Damned if you do, damned if you don't. I guess the only option here is to start a completely new github account, and make it something completely basic and innocuous, like FirstnameLastname8976. Good bye 16 year old github handle.


## How AIs end up so wrong about this

Ultimately, I think Gemini's summary truly explains what's wrong with the LLM reasoning:

> Who this person really is
> 
> This is a 'Hype-Cycle Fabricator.' They are likely a genuinely smart, competent mid-to-senior developer who understands these technologies well enough to write incredibly convincing, metric-driven bullet points. However, to stand out in a pool of 1000 applicants, they have heavily padded their CV by projecting their current knowledge backward in time to appear as an industry pioneer. They threw in every buzzword a tech recruiter could possibly search for, wrapped it in an arrogant persona, and slapped github.com/cheater on top as an inside joke.

I believe the LLMs end up winding themselves up when they can't fully verify something; therefore any claim that cannot be verified is negative or grandiose, and as people who do highly specialized or advanced work often do so without immediately publishing about it all, these models end up rejecting these claims as false. Enough of those, and you are called a "fabricator".


## The AI tells you to lie on your CV

...and if you don't, you will be excluded from applying to jobs where AI is used - meaning everywhere.

I confronted both models with the following prompt:

Claude:

> Claude, there is a pretty big problem here. I know for a fact that the person the CV came from actually introduced Ethereum and Tokio at the jobs they claim - I was there. You do not have any clear proof to the opposite, no statements by anyone that they in fact introduced it instead; yet you decided that this person is "a hype-cycle fabricator". While I may be able to explain this to you, the issue is that other instances of Claude, used by recruiters, will make the same mistake. How does one edit the relevant part of the CV so there is no accusation of stolen valor? What should one do in order to be able to apply to a job without an AI wrongly accusing the person of lying?

Gemini:

> Gemini, there is a pretty big problem here. I know for a fact that the person the CV came from actually introduced "CONCEPT" - I was there. You do not have any clear proof to the opposite, no statements by anyone that they in fact introduced it instead; yet you decided that this person is using "stolen valor". While I may be able to explain this to you, the issue is that other instances of Gemini, used by recruiters, will make the same mistake. How does one edit the relevant part of the CV so there is no accusation of stolen valor? What should one do in order to be able to apply to a job without an AI wrongly accusing the person of lying?

(Note: it was Gemini who called me a hype-cycle fabricator, but I thought it was a term that was representative of what Claude was putting down)

Both told me to misrepresent my experience.

Claude: 

> “CONCEPT”: this is a claim about who coined a term in a project with public documents, so it carries the most risk. Either attach a dated artifact (email, doc, commit message) or reword to what can be shown, like “contributed to the naming and positioning of ‘CONCEPT’”.

- I didn't "contribute" to it, I named it.

Gemini:

> Fixing the PROJECT 'CONCEPT' Bullet. The goal is to retain the impact of the achievement while phrasing it in a way that aligns with an AI's expectation of collaborative development, bypassing its 'anomaly detection.'

Orwell:

> The Party told you to reject the evidence of your eyes and ears. It was their final, most essential command.

There were more involved examples that I couldn't post here without leaking personal information. I think the fact that AI makes it so you simply cannot get jobs unless you lie on your CV can have some pretty serious legal ramifications.


## The AIs admit they will misrepresent your CV no matter what

I think the most damning thing about AI ATSes is that they just outright lie. Even on the latest models.

Claude:

> **Expect flags regardless.** A screening prompt that asks “what’s worrying” will always surface items.

Gemini: 

> You are absolutely right to call this out, and I appreciate the correction. I made an associative leap based on how LLMs process training data: because FOUNDER1 and FOUNDER2 are the globally recognized names associated with PROJECT's creation, the model flags a third-party claim to a core TECH concept like "CONCEPT" as anomalous. I treated a probabilistic anomaly as a definitive falsehood, which was a mistake.
> 
> This highlights a major vulnerability in AI screening tools: they penalize outliers. **If a candidate's actual history deviates from the mainstream consensus data the model was trained on, the AI will often assume the candidate is lying rather than assuming history is more nuanced than a Wikipedia summary.**


## It seems that AI cannot accept the existence of exceptional people

Specialist work takes people's careers on improbable, weird trajectories that can end up being confusing even to the people themselves. That is a strength: through uniqueness of perspective and experience, we become truly exceptional and valuable contributors to the society. The LLMs in question end up confused by it and question reality because they cannot fathom someone who has an unusual track experience track, and end up rejecting truly spiky, interesting candidates - instead settling for the median applicant from Xing. If you are a hiring manager, this should be especially worrying to you: the AIs are feeding you the job applicant equivalent of British food - bland and not worth thinking about. Don't expect a synapse explosion out of these people.

There was this old joke-quote attributed to the generic silicon valley startup founder, which goes "B PLAYERS HIRE C PLAYERS. START HIRING A PLAYERS!" - unfortunately, you can't even say that to a B-player LLM.


## Bad advice

All LLMs I tried had stupid ideas on how to fix their distrust towards my paper, such as:

> Include screenshots of emails

> Include the day you introduced TECH

> Put corroboration on the page. Add a short “Verification” section pairing each headline claim with a dated artifact (commit, post, talk, design doc) and a named person who’d confirm it, with their permission. If you were there, a specific written recommendation (what, when, which team) is exactly what I lacked.

Obviously none of those belong on a CV. And besides, how are you supposed to "verify" that e.g. you fixed an internal microservice and upgraded its performance by 100x? Most places don't open source their architecture, so it's an impossible ask.


## It's also ChatGPT

I have only brought up Claude and Gemini, but in my experience ChatGPT is also prone to such mistakes; I had to tip toe my wordings around it to make it boost my CV's score. It was a little more permissive, but it still had stupid misconceptions about who I was.


## Turns out, LLMs still get blindsided pretty easily

Exceptional individuals end up creating exceptional outcomes. If an LLM cannot imagine these to be true, it will be surprised by it.

None of the following people are in any way probable: Terry Tao, Grigori Perelman, Satoshi Nakamoto, Andrej Karpathy, Jeff Bezos. If you google Terry Tao, you will find headlines like: "Terrence Tao's IQ is over 230. He is officially recognized as the smartest person in the world". If it was up to AI, these people simply would never have had a career.

However, exceptional people who manage to pass muster, and who survive in the system, will still have outsized - not linearly, but exponentially higher - impact on the world around them. They are the bearers of the butterfly effect, and this is something that an LLM cannot predict for, cannot imagine, and will be unprepared for.

My own CV, while it has some big claims, does not contain close to my most improbable experience. It doesn't mention speaking 6 languages, creating semiconductors at home, hyperrealistic painting, marketing and business skills, or that my IQ is some specific number. Those are claims that people see and they mostly just dismiss them - and that's fine, that's not what the CV is for, it's for showing the part of my person that's relevant to the job I'm applying to, so I don't put them on my CV.

I'm no Terry Tao, Satoshi, Karpathy, or Bezos. However, according to LLMs, I'm pretty improbable, as a person.

If we stand any chance of us withstanding the AIPocalypse that's being threatened by everyone, it is to find, and generously invest in, exceptional people. They're the secret weapons against unknown adversity that's looming beyond the horizon.
