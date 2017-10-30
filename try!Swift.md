autoscale: true
theme: Poster,1
# Refactoring your app in Rx

---

# Prelude

^ Hello Everyone! It’s an honour and pleasure to speak here. Round of applause for the  speakers and organizers. Great job everyone .

---

# 👏👏👏

---

Show of hands ✋

1. How many people here’ve heard of Functional Reactive Programming?
2. _How many people here’ve heard of Reactive Swift/RxSwift?_
3. _How many people here’ve used Reactive Swift/RxSwift?_

^ Don’t forget to announce the percentages for video reasons

---

Show of hands ✋

1. _How many people here’ve heard of Functional Reactive Programming?_
2. How many people here’ve heard of Reactive Swift/RxSwift?
3. _How many people here’ve used Reactive Swift/RxSwift?_

^ Don’t forget to announce the percentages for video reasons

---

Show of hands ✋

1. _How many people here’ve heard of Functional Reactive Programming?_
2. _How many people here’ve heard of Reactive Swift/RxSwift?_
3. How many people here’ve used Reactive Swift/RxSwift?

^ Don’t forget to announce the percentages for video reasons

---

What this talk is _**NOT**_ about

---

- Introduction to RxSwift

---

- ~~Introduction to RxSwift~~

---

^ It isn’t an introduction to Rx, observables or reactive programming. It’s a complicated topic, you need to rewire your brain quite a bit to think reactively. Trust me, I tried.

![inline](tweet.png)

---

-  RxSwift with MVVM

---

- ~~RxSwift with MVVM~~

^ That’s a whole other ball game, and really hard to explain in a 20 minute session.

---

Stories
Examples
Experiences

^ What this talk **will** be, is a bunch of stories around how we refactored large parts of our app using Rx, and the stuff we found useful.

---

RxSwift is _**not**_ always the best way to solve a problem

^ Remember, RxSwift is not always the best way to solve a problem, but you kinda have to know when to use it to its full potential. I will try to present you some examples that you can use in your application.

---

I DON'T KNOW RxSwift 😰

^ Now, a lot of you might be going
^ And that's all right

---

>Conferences aren’t about learning things, they’re about learning *what* to learn, network, and meet great people.
-- Some guy on Twitter

^ No one’s gonna turn into an FP talk after watching 15 slides of maps/flatmaps. No one’s gonna turn into Peter Norvig after a 20 minute ML talk. Sorry Mugunth

---

So chill ❄

^ and lets get to work

---

# 2.0.0 🎉🎉🎉

^ We started refactoring large parts of our app using RxSwift for our 2.0.0 release. We had lots of fun and interesting challenges, shipping actual features and value to customers all while writing testable maintainable code.

---

^ Before covering these examples, I'll go over some bits on Networking that'll help you understand some of the stuff I'll be covering

# Networking

---

Rx ➡ streams of values over time.
Sockets ➡ streams of data packets over time on a wire
➡ Rx ❤️ Sockets

^ Thus, Rx translates really well for things like websockets, cos websockets are packets over a wire over time.  We’ve even written our own wrappers around existing websocket clients for our app, and things like disconnections, event subscriptions etc. are handled _really_ well. (link to repo)


---

## What about HTTP requests?

![inline](gps.png)

^ You know, where you make a request and you get a response. And there’s just one response. Not very stream-ey, is it? Is Rx still useful for them?

^ Here’s the thing. Requests don’t exist in isolation. Often they’re part of a bigger flow, like refreshing expired tokens, retrying with exponential backoff if you lose an internet connection. Which is why often you don’t deal with a single request by itself in your application flow. We’ll discuss this stuff in detail later, but I brought this up early because there’s this really cool function that has some interesting use cases in Rx. It’s called flatMapLatest

---

# `flatMapLatest()`

---

# Notification Preferences

^ We shipped notifications with our app in 2.0.0, and we needed to have preferences so the user can opt-in/out of certain notification types

---

![inline fill](noti-prefs-web.png)

^Pretty standard UI, but we weren't big fans of the save-cancel buttons. We wanted something that the users didn't want to put much thought towards and felt _automatic_

---

![inline](noti-prefs-ios.jpeg)![inline](noti-prefs-android.jpg)

^This is what they looked like on android and iOS

---

![inline](prefs_request.png)

---

## _Maximize_ responsiveness _Minimize_ requests

---

## First idea: a request queue 🤔

---

![inline autoplay mute](ios-optimistic.mp4) ![inline autoplay mute](android-optimistic.mp4)

^ Our first idea came from our android dev - implement a request queue. We’d used one recently for sending messages in our app, and this’d be a good technical fit for reuse. Problem is, this’d mean one request for every switch, which could mean if the user moved 5 UISwithces, there’d be 5 requests in serial order which would be very high latency

---

## Consistency 💯
## Latency 😞

---

# State

1. Dictionary of `[id: Bool]`
2. only send a dictionary if state has _actually_ changed

^ We decided we would do the ideal thing, which’d involve keeping track of the “state” - think of it as a `[“id”:  Bool]` dictionary, and only send a request every few seconds if the state actually changed. This way, if a user turned a switch on, then off, a new request wouldn’t be sent at all!

---

# Imperative approach

- It. Was. Hell.

---

```
let timer: Timer
let request: DataRequest

// every time state actually changed
{
	timer.invalidate()
	self.request = client.createNewRequest()
	// wait for that callback to complete
}
```

---

Imagine if we added desktop preferences

![inline](whatif.png)

---

```
let timer: Timer
let request: DataRequest

```

---

```
let mobileTimer: Timer
let mobileRequest: DataRequest
let desktopTimer: Timer
let desktopRequest: DataRequest

// and the checks would be worse too
```

---

This gets untenable _very_ quickly

---

# With Rx

![inline](throttledSwitch.png)

^ In contrast, the approach was delightfully simple. 

---

and if we wanted to cancel requests automagically ✨ , all we’d do was

![inline](throttledRequest.png)

^ So, now so much weird mutable state is now just handled all well for free!

---

# Online Indicators

---

![inline](POP.png)

^ Let's do POP like all good swift programmers who've seen Dave Abrahammson's WWDC talk. And in our actual code, we’d update from a network client. In our tests, we’d just broadcast events and test for the indicator appearing

^ In fact, there's nothing wrong with this approach. It's perfect, uses swift's tools and is also testable.

^ One Problem though

---

# 🤔

^ Problem with this approach is, you have to **think**. THINKING. IS. HARD. There’s a reason why most talks about testing in conferences have like 40 year old grizzled devs - it takes experience (and lots of it) to write testable code well.

---

# Subjects

^ We’re using an Rx data structure called a subject. These act as both observers **and** observables. So, you can send events *TO* them, and all their subscribers receive their events. They’re like radios. Lots of people can broadcast on the same frequency, and anyone listening in, gets everything they’re saying. (Let's not talk about threading just yet, cos it makes things complicated). The best part about them is that you can use _only_ the observable bits using the `asObservable()` function

![inline](RxAvatar.png)

^ For our tests, we can hook them up to an Observable we created ourselves. And for our app code, we can hook it up to our websocket connection

---

![inline](avatars.gif)

^ So now we get testability for free 🎉🎉🎉🎉🎉

---

## Background Images

### If you put text on top of an image, the image is _**filtered**_ so the text is always readable. 

---

# Isn’t that **great?**

![](http://deckset-assets.s3-website-us-east-1.amazonaws.com/colnago2.jpg)

---

# You can also turn the filter off.

![original](http://deckset-assets.s3-website-us-east-1.amazonaws.com/colnago2.jpg)