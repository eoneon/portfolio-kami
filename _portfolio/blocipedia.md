---
layout: post
title: Blocipedia
thumbnail-path: "img/blocipedia.png"
short-description: Blocipedia is an app that allows users to create and collaborate on wikis.

---

{:.center}
![]({{ site.baseurl }}/img/blocipedia.png)

## Explanation

Blocipedia allows users to create and collaborate on wikis. Wikis are create and edited directly from the browser using markup. Blocipedia was the second app I developed from scratch acting as lead developer under the guidance of a mentor, Adam Louis.

## Problem

Wikis are collaborative by definition, but we wanted to offer different levels of collaborative access for the twin goals of generating of traffic through non-paying users, while charging for extra features for users wanting to upgrade their accounts. Aside from these goals, we also wanted to allow for moderation of the app's content by building in an admin role. Of course, providing different levels of access requires an authentication and authorization system that is integrated into some mechanism to actually take payments from users. Finally, the wikis themselves are created and modified using markup so we needed to incorporate a component where content could be edited in markup but rendered in HTML for preview and display.

## Solution

#### Authentication
As a threshold matter, we used Devise for Blocipedia's authentication system. Using a custom solution abstracts away much of the logic for facilitating signing-up, logging-in, and logging-out. For authentication params, Devise uses email and password by default, but we easily added a username param by simply adding a before action filter in the application controller for any actions routed to the Devise controller.

#### Authorization
Like Blocipedia's authentication system, we used a custom permissions system (Pundit) to govern the hierarchical levels of access to the app's functionality. In order to attract new users, we offer free accounts which enable standard users full access to CRUD operations over their own content. Since the point of wikis is collaboration, we also permit standard users to view and edit public wikis regardless of authorship.

Realizing some users might want more control over their content, we enabled users to create private wikis by upgrading their accounts. Private wikis are only viewable by other premium users, admins, and standard members that have been added as collaborators. The ability to add and remove collaborators from a private wiki is within the exclusive domain of the wiki's author.

Admin users are permitted to view both public and private wikis, and remove any wiki that violates the spirit of Blocipedia. The role of admin users is only to regulate the site's content, so they still may not edit private wikis unless they have been added as a collaborator by its author. In this way, admins have similar standing to standard members.

#### Upgrading & Downgrading Accounts
To handle the details of accepting and refunding payments, we integrated Stripe into our app. The benefit of Stripe is that the machinery of accepting and refunding payments is easily integrated, while also handling verification of credit card information and ensuring payments are secure.

Requests for upgrading and downgrading accounts are handled by a `Charges` controller which is itself governed by a companion policy ensuring only premium users may downgrade, and only standard users may upgrade. To keep `Charges` neat, we stored upgrade and downgrade logic in separate modules in the services directory.

Updating an account involves two steps, facilitating a charge using Stripe's API and upgrading a user's access within Blocipedia. With a goal of breaking the process down into smaller steps, we execute these two steps by calling a dedicated class method and pass in the following parameters: a `user` object and Stripe `token` object which stores the encrypted credit card information.

After initializing these objects, a Stripe `charge` object is created consisting of the encrypted credit card info, user info, and the amount to be charged. Because credit card data is encrypted and sent directly to Stripe rather being persisted to the app's database, these transactions are secure. If a charge is successful, a user's role and access are upgraded. Downgrading an account is handled in a similar fashion.

#### Editing Wikis using Markdown
Finally, for editing wikis we used EpicEditor which allows for split fullscreen editing and live previewing. The editor is created by embedding a Javascript script on the edit view. The Javascript logic that controls the editor's functionality is contained within files in the assets directory, and reference a selector id that targets the form input field where wikis are edited.

## Results & Conclusion
As a novice developer, I begin projects with a sense that every detail needs to be planned out before any code is written. But with Blocipedia I realized that some app features have consequences that I might not anticipate at the onset. Specifically, when designing the machinery for downgrading accounts, I didn't consider what should happen to their associated private wikis and collaborators.

On one hand, wiki privacy and the power to add collaborators is what distinguishes a premium and standard user is the sole justification for charging for this status. But if a user had premium status when they created private wikis, shouldn't they retain this value? If so, how would we handle their permission; would we have to add them as a collaborator to their own wiki? Also, should admins have the power to edit private wikis? Is this central to an admin's role or does this encroach on the exclusive domain of premium users?

We chose to update private wikis to public and delete the associated collaborators for downgraded accounts. The decision turned on how we set up our payment system. We charged a flat fee, but if we used a subscription setup, we would have gone the other way. Determining admin access to private wikis was a balancing act between the utility of moderating and the paid benefit of selecting collaborators. We chose to permit admins to remove wikis, but we didn't want to encroach on premium user's exclusive domain to add collaborators.

This project taught me to think of every development choice as having a sequence of consequence. In the case of Blocipedia, I didn't anticipate some of these consequence when trying to design the application before I started to write any code. But in retrospect I would have been in a better position to anticipate these implications if I followed the request and response cycle and user flow for downgrading an account and tracked every model involved.

Finally, while some decisions are more process-based in terms of a chain of sequences triggered by a given action, others are require balancing an app's competing goals. It was easy to balance the need to acquire users with free access to the app, with the desire to monetize premium access. But deciding where the line between moderating the site and devaluing premium use was more difficult.
