---
layout: post
title: Bloccit
thumbnail-path: "img/blocflix.png"
short-description: Bloccit is a Reddit replica for creating categorized posts that may be up-voted, down-voted, commented on and more.

---

{:.center}
![]({{ site.baseurl }}/img/blocflix.png)

## Explanation

Does the world need another Reddit style content management system? Of course not! But what better way to learn Rails than to build a CMS from the ground up. Using Bloc's backend software engineering curriculum as a roadmap, **Bloccit** was built from the ground up with a host of features users have come to expect from a web app geared toward publishing and interacting with content .

## Problem

Bloccit provides a community of users the ability to respectfully share and interact with categorized posts. As with any social app, the spirit of that interaction is defined by both its members and moderators. From the member-perspective, this implies an ability to author user-specific content, as well as interact with content from other users. From a moderator-perspective, this implies an admin role to regulate content and interaction in a way that conforms to Bloccit's purpose. This latter role entails creating topical categories for content, as well as removing user-content that violates the spirt of the app.

## Solution

#### Signing-Up, Signing-In & Signing-Out
Content is central to Bloccit, but `posts` and `comments` would be meaningless if they weren't associated with the `users` that authored them. Thus, Bloccit's models, controllers, and routing are authentication-based.

>Bloccit's authentication system starts with creating users with validated emails and passwords, the latter of which is facilitated by the `BCrypt` gem and ActiveModel's `has_secure_password` method for ensuring secure passwords.

Custom systems like Devise are the norm, but we built this system from scratch in order to demonstrate a clear understanding of the relationship between a session and the sequence of signing-up, logging-in, and logging-out.

>Sessions are handled by a sessions controller and module that (i) authenticate a user with their log-in credentials, (ii) creates a session object with a user's unique id, and (iii) creates a current user.

#### Levels of Access
After a user signs-up, their permission-level is set by default to **member** status to distinguish their access relative to **admins** and **guests**. Member-users have permission for all CRUD operations regarding their own content. Members may interact with posts they don't own by commenting them, favoriting them, up-voting & down-voting them, or tagging them with labels. Members may also view topics which are publicly scoped.

Admin users' access to CRUD operations extends to both `topics` (whether public or private), and `posts` (regardless of ownership). Non-authenticated users (guests) may view public `topics` and their associated `posts` and `comments`, but are restricted from all writing CRUD operations. An API is not strictly related to authorization, but users with a token may perform CRUD operations (according to their role) from outside the system.

#### Commenting Topics & Posts
So that `comments` may be added to either `topics` or `posts`,  we schematically define its foreign key as polymorphic. Polymorphic child-objects keep track of which parent-model to target for matching primary and foreign keys. For instance, since we interact with a `comment` exclusively within the context of its parent, upon creation its `comment_id` is set with the value of its parent's `id`, and `comment_type` is set with a value of either "topic" or "post".

#### Shared Tags for Topics & Posts
Like `comments` we wanted to provide authenticated users with the ability to tag both `topics` and `posts` with `labels`. The difference being, while any given `comment` may reference at most a single parent-object (whether it be a single `topic` or single `post`), we wanted to allow `labels` to be shared between multiple `posts` and `topics`. After all, a tagging feature loses much of its organizational utility if each tag is unique.

Unlike the `comment` example, a `label` doesn't directly reference its parents. Instead a join-table (`labelings`) stores a reference to both sides of the relationship. We can think of a join-table as having a left and right side. The **left-side** performs two functions: (1) it references the parent-object by storing its `id` in `labelable_id`, and (2) it references the corresponding parent-model in `labelable_type`. The **right-side** references the child-object by storing its `id` in `label_id`.

## Results

Test Driven Development (TDD) is a best practice for building apps, but was especially crucial when building my first app. To test our models and controllers, we used the RSpec framework. To make writing our tests more efficient, we used the Shoulda gem for testing model associations, and FactoryGirl for building objects used in both our model and controller tests.

For interacting with our app in the development environment, we initially seeded our data using an IRB shell in the console. But creating data manually in the console is time consuming -- especially for associated objects. So for efficiency sake we decided to build a module to seed our app with random data.

## Conclusion

Bloccit has become a reference point for my subsequent projects. One technique I continue to draw upon is the way Bloccit handles requests for polymorphic objects using modules to set the parent object before the underlying controller handles the request.

>I found this technique on Railscasts, but I plan to experiment with another technique which forgoes modules and relies upon a before action filter that fetches the parent object by parsing the path.

One notable difference in my subsequent apps is that I rely more on custom solutions for building widely used features like authentication, authorization or even for testing in terms of seeding data. It doesn't make sense to reinvent the wheel, but when using frameworks I anchor my understanding of them through the lens of the features I built from scratch.
