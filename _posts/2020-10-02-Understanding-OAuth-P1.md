---
title: "Understanding OAuth from a penetration tester’s perspective — Part 1"
date: 2020-10-02 00:00:00 +0530
categories: [Authentication]
tags: [auth,OAuth] 
image: "assets/img/Posts/uoauthp1/thumb.png"
---

Here, we are going to understand OAuth and related security concerns.

<!-- ![](https://cdn-images-1.medium.com/max/2000/1*uvBZtaz3DsBtbsz4JCtNGA.png) -->

## Target Audience

So, if you are following my Machine Learning articles, you might be thinking “what is OAuth?”. Or if you know OAuth, you might be thinking “why a Machine Learning Engineer would write on OAuth?”. Yes, your question is legit, and here’s the answer.

Apart from Machine Learning, I have an interest in Cyber Security. And OAuth is something that web applications use widely nowadays. So, when writing this article I ain’t an ML Engineer but a Penetration Tester (Am I? Nope, I am a still learning).

So, the target audience here is:

1. Someone who is developing web applications

1. Someone who is trying to break web-based applications (obviously, the ones using OAuth)

And the prerequisites are:

Oops.. there are no prerequisites. I’ll cover everything needed.

So, let’s start with the very basics. As this is my first article into Cyber Sec, you might not know about my writing style. The best I can tell you about my writing style is, “I always try to cover everything and in a way as organized as possible”.

So, let’s first answer this “Why another article on OAuth?” Obviously, there are so many resources out there. But those are incomplete. A lot of you might be having a hard time understanding OAuth just because the information provided in the article you read was incomplete. Now, you have to look for answers elsewhere. When I was reading about OAuth, I had to refer about 13 blogs, 2 YouTube videos and 2 papers (RFC). This is a lot and it might have been better if one can find all these at a single place. Also, after reading all these, the information I gained seemed incomplete. I had many questions in my mind which weren’t answered on these references and I was bored looking out for information. So, I started creating answers myself.

## Why OAuth? / OAuth History

“Wait.. what? I still don’t know what OAuth is and you are diving straight into the why part!” — yeah, when knowing “why”, I will make you know “what”. *Also, penetration testers may skip this part if they feel that it’s very boring.*

So, the journey of OAuth looks something like this:

OpenID → OAuth → Oauth 2 → OpenID Connect

Now, what are all these? What is the difference between them? All such questions will be answered here while going through why they are needed.

So initially, there was a need for businesses to **authenticate** users across multiple websites. And at this point, instead of having several accounts for each website, why not have a single account that can work over these multiple sites?

For example, we all use StackOverflow, StackExchange, and other stack related websites. You might have noticed that once you register on one of these stack sites, you will be able to reuse the same account on stack related websites. This mechanism is called single sign-on (SSO) i.e. registering once, using everywhere. When I say everywhere, I mean all the sites that trust each other. For example, you won’t be able to use StackOverflow’s account to login in GitHub. Because they don’t trust each other and they haven’t agreed to share their users’ identity.

A similar example of SSO is commenting on blogs. In earlier times (about 5 years back), if you read blogs and wanted to leave a comment on any blog, you might have got multiple options to **comment as**:

* Anonymous

* Google account

* OpenID

* Name and Email

*Have a look at the comment section of [The Hacker’s Library](https://libraryofhacks.blogspot.com/2018/12/how-i-hacked-gtus-website-admin-panel.html)* 

As you can see, here too the bloggers used single sign-on i.e. if you are signed in with OpenID in your web browser (ex: Chrome), then you may use OpenID account to leave a comment on that blog. This is because the blog supports/trusts OpenID’s website data, and OpenID as well agreed to share the user’s identity with that blogging platform.

So, **Single Sign On is, creating an account at some website and reusing that account for authentication at other sites (which trust that website)**. Single Sign On is an authentication mechanism. So, what is OpenID? **OpenID is a way to implement this SSO mechanism**. As mentioned above, in the early days, if you had an OpenID account, you could use that account to authenticate yourself at multiple other websites/blogs. Hence, the name “Open + ID”. Your virtual identity (a single account) is open to several applications.

Now, as days passed, there was an increase in the needs of businesses. Now, along with the above mentioned feature, they also needed something else. Businesses wanted to access users’ data (not just their identity) from other websites.

For example, I own a website “abc.com” and when a user uses my service, I want to access that user’s data (maybe, photos?) from “pqr.com”. Now, accessing data is something related to authorization and not authentication. Hence, OpenID won’t help. OpenID serves the purpose of Authentication only.

### Authentication vs. Authorization
> In simple terms, authentication is the process of verifying who a user is, while authorization is the process of verifying what they have access to.

In the above example, the website “abc.com” needs to access the data of a user on another site “pqr.com”. Hence, **the site “pqr.com” needs to authenticate the user** in order to know who is he/she. Also, **the site “pqr.com” needs to authorize the site “abc.com”** in order to know, what data of the authenticated user, does “abc.com” have access too. Read this paragraph another time to make the requirement clear.

Here, the site “abc.com” will know who the user is by the user’s OpenID (user account on the “pqr.com” website). But still, we need “Authorization”. Look carefully, we need to **authorize a website with another website and not a user with a website**. Hence, this was something new for the businesses and hence, the WWW needed a new mechanism. OAuth was then found as a way to implement this mechanism.

Now, you might be still confused about where are businesses using OAuth. So, here’s a simple example. You might have seen “Login with Google” or “Login with Facebook” on several websites. These websites can log you in using Google/Facebook because Google and Facebook have implemented OAuth. Let’s see what happens.

So, when you log in to a website using Google, the website needs your email address, your name, your birth date, your gender and maybe your profile picture. Are you submitting all this information to the website? No.. The website gets this information from your Google account. Here, this information i.e. email address, name, birth date, gender and profile picture is nothing but a user’s data. And the website needs to access that data from the user’s Google account. Hence, the website needs to Authorize itself to Google. Remember, authorization is the process of verifying what they have access to. And here, the Google server will verify if the website has access to the user’s email, name, birth date, gender  and profile picture.

![**Why OAuth?** ](https://cdn-images-1.medium.com/max/2000/1*EAdjGgcUbRaCJ7C6isEA8A.jpeg)***Why OAuth?** *

## What is OAuth and How it works?

So, that now you know why OAuth is needed, let’s dive into know how it works and also look at it’s definition.
> # OAuth is an open standard for access delegation, commonly used as a way for Internet users to grant websites or applications access to their information on other websites but without giving them the passwords. — Wikipedia

Did you focus on the part “without giving them the passwords”? So, why is it necessary to include this phrase in the definition? The problem we are trying to solve is, a website (for ex: website A) needs to access user’s data on other website (for ex: Google). So, how developers might have approached this problem?

The website A might ask users for their username and password of their Google account. Then, the website A will log in to the user’s Google account using the provided username and password. After login, the site will access user’s data (for ex: profile picture). But then, who would guarantee that the website will only access the profile picture of user and not his/her contacts, google drive files, google photos, etc. “Google will guarantee it. Simple!” —No, it’s not that easy for Google.

Yaa.. so if we consider that Google will verify the access rights of website A, then the first question which rises in my mind is “How will Google know if the one who is asking for data is the website A and not the user himself/herself?” I mean, website A might fire up a web browser on it’s side and then use browser to login using the user’s credentials. What would Google do in such a case? You simply can’t trust website A with your credentials, right? And that’s the reason why OAuth is secure (well.. it’s not so secure as we will see next), as it does not require the user to provide his/her credentials to website A.

Now, suppose the website you use is trusted. You trust it with your credentials. Suppose that it is built by you and hence, you trust it. You know that this website will not impersonate (pretend to be) any user when asking information from Google. You know that this website will only access what it wants and nothing extra. But still, you should not provide your password to this website. What if the website got hacked? What if the hacker got access to website’s database and hence, your password? This is also one of the reason why Google too doesn’t know your Google account’s password. **They save the hash of your password and not your plaintext (original) password.**

So, if not this way, then how does this OAuth thing work?? We will see this now..

### OAuth Flow

The fun begins..

Remember the OAuth history I mentioned above? We traveled the path from OpenID to OAuth but now, we have two versions of OAuth — OAuth1 and OAuth2. There are too many differences between these two versions. And hence, we need to go through them.

The first question you should get in your mind is
> “which version are we going to study? Both?”
> No.. we are only going to look at one version which is OAuth2.
> “Why?”

So, here comes the very first difference. **OAuth1 is deprecated now and all of the websites have shifted to OAuth2 (except Twitter, and a few others).**
> So, what are the differences between these two?

In order to understand the difference between two, you must know at-least one of them, right? So, let’s understand OAuth2 and then we will get back to the difference.

### OAuth Roles

In order to understand how OAuth works, you first need to understand the roles. Suppose, you give a child a few dollars and he goes to the shop to buy a chocolate with that money. In this case, you are the “money lender”, the child is a “buyer” and the shopkeeper is a “seller”. This is what I mean by “roles”.

In OAuth, you already know that there is a user, a website and some another website. And one of the website wants to access the user’s information from another website. Now, when some website asks Google for your profile picture, Google does not directly give the profile picture to it. It will need to verify if that website is authorized i.e. has access rights to read your profile picture.Here, we are going to understand OAuth and related security concerns.

If the website is not authorized, it will ask you. It will tell you that this website here is asking for your profile picture and you haven’t authorized this website before. Do you want to authorize it now? If you click say “Yes”, then Google will add that application to your authorization list and will send your profile picture to that website. If you say “No”, then Google will deny the website and it will not provide the profile picture. Here, what/who are you?, what is Google? what do we call that another website which is asking for profile picture? Let’s see..

OAuth2 defines 4 roles :

* **Resource Owner**: generally yourself.

* **Resource Server**: server hosting protected data. For example, Google hosting your profile and personal information.

* **Client**: application requesting access to a resource server (it can be any PHP (, .Net, etc.)  website, a Javascript (, Angular, etc.) application or a mobile application). For example, website accessing your profile picture.

* **Authorization Server**: server issuing access token to the client. This token will be used by the client to request the resource server. This server can be the same as the authorization server (same physical server and same application), and it is often the case.

So, the confusing part here is this newly added “Authorization Server” that came into the process. Let’s see what it is and why it is needed.

### Steps in OAuth

Revisiting OAuth flow with the above mentioned roles.

1. Client wants to access resources (photos) of resource owner (you).

1. Client asks the owner to grant it the access to his/her photos.

1. The owner accesses the authorization server (Google) and adds client as a trusted party. The owner also allows the client to access his/her photos.

1. The authorization server gives the owner an authorization code. It asks the owner to pass on this authorization code to the client. Only if the client provides this authorization code, the authorization server will consider it authorized by the user.

1. Now, the client has been authorized as the owner gave it the authorization code.

1. The client will now send the code to authorization server and ask it for an access token to access the owner’s resources (photos).

1. The authorization server will check if the client is trusted or not. It will also check if the client is granted access to the owner’s photos. If the check is successful, the authorization server will send an access token to the client. Else, it will deny the access.

1. Now, the client has the access token. It will send the access token to the resource server (Google Photos) and ask it for user’s resources (photos). The resource server will verify the access token and send user’s photos to the client.

This is the eight step process which has many bugs, right? No.. **the process is neat and secure but the way developer’s implement these eight steps is insecure**. By the way, do developers develop the OAuth framework by themselves or just search GitHub for openly available code? Pentesters/hackers know the advantage when developers use openly available code.

![Simplest flow diagram.. Source: Digital Ocean](https://cdn-images-1.medium.com/max/2000/1*fyfmxv1QjNyo9uan1YZroQ.png)*Simplest flow diagram.. Source: Digital Ocean*

For pentesters: What do you think? Can it be hacked!

At this stage, I was trying to answer the above question. See, steps 1–4 needs user intervention. And as we say, **humans are always vulnerable**. Hence, we can attack. As the owner is performing all tasks on his/her browser, we will need to hack using JS (inject malicious code into the browser by some means) or maybe do some social engineering.
> What are the ways to inject malicious code in a user’s web browser? I need your comments on this. The comments will help others (as well as me) to know all the possible ways to inject code in web browser (One might know a few but not all.. a discussion/listing would help).

Now, in the steps after 4, the client (some website) is directly communicating with the authorization (and resource) server. Now, as there is no user, it will be hard to hack. The only possible way to hack at this stage is, if any of these websites has some kind of flaw. What are the flaws that we are looking for? This question will be answered later in this (or next) article.

Also, there is another thing to note of. The complete  process is based on two codes —authorization code and  access code. In order to break the process, an attacker needs to get any of these codes. And in order to hack it completely, the attacker needs the final access code. How to get it? That too will be answered later. For now, you may think about it for a while and let me know your thoughts in the comment section. The thinking process is interesting.

Also, two more questions, for both, developers as well as pentesters.

1. Why do the initial steps require user intervention? Is there a way to remove user from the process?

1. Why does OAuth use two codes? Why not only the access code? What happens if we refine the process by just using a single code?

The answer to these questions are already in the paragraphs above it. Try solving these mysteries and leave your thoughts..

Looking at the current size of this article, I don’t think that I can fit all the information in here. Let’s just end it at this point. I will write the next part in a few days and update links here.

Also, I have given you too many questions to think about. Think for a while, share your thoughts, and get replies (from me and other readers). Discussing your ideas is the only way to get knowledge from others.

Remember,
> # Curiosity is always the “main” key to learning

Also, just a reminder for myself — we still have the second half of history of OAuth, difference between two OAuth versions remaining and OpenID Connect. I’ll cover them in next articles, as following a sequence is the best way to learn; and reading for too long is not possible for everyone. I’ll also try to cover SAML in upcoming articles.

Till then, think think think… and share your thoughts :)
> In the next article, we will dive into the technical details of OAuth. This will be much more interesting as you will then be able to guess more confidently about the kind of possible attacks on OAuth.

### By Kadam Parikh