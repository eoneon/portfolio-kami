---
layout: post
title: Using Request/Response Cycle to See the Forest Through the Trees
feature-img: "img/sample_feature_img.png"
---
For my first project as part of an online software engineering program (Bloc), I built a Rails app similar to Reddit where users interact with an index of `topics` and their associated `posts` and `comments`. The idea is that with the help of a mentor, students learn Rails by incrementally building interrelated resources and features until they amount to a functional web app. The process is highly systematic but being new to Rails I often found myself losing the forest through the trees.

As a novice I find learning to develop is less about syntax and more about internalizing programming patterns in the context of an app or framework as a whole. I initially thought the process would be similar to learning a second language like French. It didn't take long to see that the two disciplines were totally different; fluency in English provides a built in reference point for making translations into French. I had no analogous reference point for programming.

I was confronted with the same problem in my first year of law school. Learning to "think like a lawyer" involves reading hundreds of cases, extracting their discrete and interrelated premises, before synthesizing them into a legal patterns that can be applied to future legal questions (exams). My first instinct as a novice developer was to approach programming as I did law school; by making hierarchical outlines documenting the process.

This wasn't entirely unhelpful, but without an organizing principle or sense of scope for an app's components, my outlines devolved into ever-expanding overlapping lists. I had lists related to the command line to remind me how to navigate and build directories; lists for Git and Rails commands; lists for each domain of MVC architecture; and lists for Ruby concepts.

Keeping track of sequential steps gave me a warm feeling like I had breadcrumbs to find my way in and out of the forest, but they lacked any consistent organizational framework. I also tried relying on the structure of Test Driven Development as a learning device, but it was too abstract; controller tests _simulate_ requests, thus skipping over the machinery that trigger requests in the first place -- links, form submission and the HTTP protocol.

It wasn't until I started mentally framing my development work flow for a web app around a user's experience as it tracked with the Request/Response (R/R) cycle that I began to internalize the process. Using a user's experience and R/R cycle as a reference point led me to fully understanding the HTTP protocol and the logic beneath Rails path and URL helper methods.

Finally, most every other feature I incrementally added to my app could be mentally organized in this way, whether I was an authentication and authorization component, permissions and roles, or exposing features of the app through an API. Aside from added features, it also helped understanding added wrinkles like polymorphic resources or single table inheritance because these concepts didn't feel like they were living in a vacuum but within a framework I already had a grasp on.

I'm not sure how helpful this may be to others, but finding a reliable learning device helped me impose order on the million little details regarding building an app. It helped me to move back and forth between the perspective of countless development details and a handful of larger components, each of which functioned according to a singular cycle. In short, it helped me see the forest from the trees. 
