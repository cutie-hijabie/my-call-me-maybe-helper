# I Got Frustrated and Restarted the Work

At some point during Call Me Maybe, I got frustrated with the way I was working.

I was making progress, but I kept moving forward, discovering that something earlier was incomplete, going backwards, fixing it, and then trying to continue again. I had code and I was getting results, but I did not feel like I understood the project as clearly as I wanted to.

So I restarted **the way I work on the project**.

I did not throw away everything I had already written. A lot of my old work is still useful and is being reused. What changed is how I organize the work, how I decide what I am building next, and how I separate unfinished features from actual bugs.

Instead of trying to finish everything at once, I now work through clear stages. Each stage represents an actual capability of the program, and I only move forward once I understand what that stage is supposed to do, how it connects to the rest of the project, and what still belongs to later work.

The first stage of this restart was about getting the existing pieces connected into one real running journey: loading the runtime files, validating the function definitions, building the model prompt, reaching the real model, running constrained generation, decoding the generated tokens, and getting readable output. The output is not fully correct yet, but the program now runs end-to-end and failures are much easier to locate.

This way of working has been much better for me. I can understand what I am building, define what “done” means for the part I am currently working on, and get useful results without treating every unfinished feature as if the whole project is broken.

## Logs From This Point On

I am continuing my project logs here from **Day 12 onward**.

The older logs are staying where they are. They are an honest record of the first way I worked through Call Me Maybe, including the confusion, experiments, mistakes, and things I learned along the way.

The [logs](daily_logs-2.0/) in this folder document the project after I changed my workflow: what I am trying to make work, what I reuse from the old implementation, what breaks when the pieces are connected, what I fix, and what I understand from it.

## Later: What Should You Do?

After I finish Call Me Maybe, I want to add another folder called something like **What Should You Do?**

That folder will not be a copy of these daily logs. The logs show what actually happened while I was learning, including wrong turns and fixes.

The future folder will reorganize everything after I know the finished project. It will contain the corrected order I would recommend, the things I wish I had understood earlier, and a cleaner explanation of how I think the project should be approached.
