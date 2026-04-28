# Presentation Script (15-Minute Delivery) — Paper 1

**Title:** Evaluating Classical Software Process Models as Coordination Mechanisms for LLM-Based Software Generation
**Duration:** ~12-15 minutes (~1,600 words)

---

**[SLIDE 1: Title Slide — "Classical Processes Meet AI Agents"]**

"Good morning, everyone. Today I'm going to walk you through a fascinating study that asks a simple but powerful question: **Can we teach AI agents to follow the same software development processes we humans use?** "

**[SLIDE 2: The Big Question]**

"Here's what's happening. We now have AI agents that can write code, design systems, and test applications autonomously. But when you put multiple AI agents together — a multi-agent system — you run into the same problem human teams face: **coordination.** "

"**Check-in question:** Think about the last group project you worked on. What was the hardest part — the coding, or getting everyone on the same page?"

*[Pause for 3 seconds]*

"Exactly. Now imagine that problem, but your team members are AI models that each have limited memory and can't see what the others are doing. That's the challenge this paper tackles."

**[SLIDE 3: What They Did]**

"Researchers from Vietnam and Norway ran **132 experiments** — yes, 132 — using MetaGPT, which is a framework that lets you assign AI agents specific roles: a Project Manager agent, a Developer agent, a Tester agent, and so on."

"They built **11 different software projects** — games like Snake and Tetris, utility apps like expense trackers and QR code generators. Each project was run with three different process models and four different AI models."

"Think of it like cooking the same dish with three different recipes and four different ovens."

**[SLIDE 4: The Three Processes — Slide explaining Waterfall, Agile, V-Model]**

"Here's the core idea. They took three classic software processes and implemented them inside AI agent teams:"

"**Waterfall — the assembly line.** The Project Manager agent talks to the Designer agent, who talks to the Developer agent, who talks to the Tester agent. Strictly sequential. No going back. Like handing a baton in a relay race."

"**Agile — the jam session.** Iterative sprints. The agents go around the circle — requirements, design, code, test — and loop back based on feedback. Like a band improvising together."

"**V-Model — the mirror maze.** For every development stage, there's a matching testing stage. Requirements are mirrored by acceptance testing, design by integration testing. Everything has a verification counterpart."

"**Slide transition** — Now let's look at what happened."

**[SLIDE 5: Results — Speed & Size]**

"The first big finding: **process choice dramatically affects how fast and how much the AI produces.** "

"Waterfall finished projects in a median of **18 seconds**. Eighteen seconds. Agile took **450 seconds** — that's 25 times longer."

"But here's the trade-off. Agile produced **37 files per project** — modular, well-structured code. Waterfall gave you only **13 files**. Less code, faster, but potentially less robust."

"**Attention hook:** Imagine if your code review speed depended entirely on whether you called your workflow 'Agile' or 'Waterfall.' That's essentially what we're seeing here."

**[SLIDE 6: Results — Quality]**

"Now here's where it gets really interesting. Let's talk about **quality.** "

"They measured quality in two ways: bugs caught by AI-generated tests, and bugs caught by human testers."

"Agile caught **35.6% of bugs** with AI tests. Waterfall only caught **18.5%**. V-Model — which should theoretically be the most rigorous — only caught **8.3%** ."

"**Check-in question:** Why do you think Agile outperformed V-Model here, even though V-Model has more formal validation?"

*[Pause for 3 seconds]*

"The answer is **iteration.** Agile's feedback loops let agents learn from their mistakes. V-Model's rigidity means agents write tests that look good on paper but don't actually catch real bugs."

"And for human testing? Agile code failed only **40%** of manual tests. Waterfall failed **100%**. That's not a typo — every single Waterfall-generated project had major issues when a human reviewed it."

**[SLIDE 7: LLM Model Comparison]**

"What about the AI model itself? Does GPT-4 beat DeepSeek?"

"Short answer: **for quality, no. For speed, yes.** "

"DeepSeek-Reasoner caught **54% of bugs** — the best of all models tested. GPT-4.1-nano was the fastest at 90 seconds average execution, but caught only **2% of bugs**."

"But here's the key insight from this study: **the process model matters more for code quality than the specific AI model you choose.** The difference between Agile and Waterfall quality is bigger than the difference between GPT-4 and DeepSeek."

"**Slide transition** — So what does this all mean?"

**[SLIDE 8: Practical Takeaways]**

"Three practical takeaways for anyone building with AI agents:"

"**One:** If you want **speed and efficiency**, go Waterfall. Your AI agents will finish fast and use fewer tokens. Great for well-understood, stable requirements."

"**Two:** If you want **code quality and robustness**, go Agile. Yes, it costs more compute time. But you get code that actually works when a human tests it."

"**Three:** Don't assume a bigger, smarter AI model automatically means better results. The **structure of your process** — how your agents talk to each other and iterate — matters more."

**[SLIDE 9: Limitations & Future Work]**

"Of course, no study is perfect. These were small-to-medium projects — simple games and utility apps. The researchers acknowledge that enterprise-scale systems with evolving requirements might behave differently."

"Also, all experiments used MetaGPT, which might naturally favor certain coordination patterns. Results might vary with other frameworks like AutoGen or CAMEL."

"Future work will explore debate-based coordination, marketplace negotiation between agents, and larger multi-module systems."

**[SLIDE 10: Conclusion]**

"So here's the bottom line. **Classical software processes aren't just for humans anymore.** "

"The same coordination patterns we've been using for decades — Waterfall's sequential discipline, Agile's iterative feedback, V-Model's validation symmetry — can and should guide how AI agents work together."

"This study proves that **process structure matters more than model architecture** for determining code quality. And that's a powerful reminder: in the age of AI, **how you organize work still matters just as much as the tools you use.** "

"Thank you. Any questions?"

---

*Script written by Bob (OpenClaw AI) following RPW Protocol — April 28, 2026*
