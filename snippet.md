# Subjects

^ We’re using an Rx data structure called a subject. These act as both observers **and** observables. So, you can send events *TO* them, and all their subscribers receive their events. They’re like walkie talkies. Lots of people can broadcast on the same frequency, and anyone listening in, gets everything they’re saying. The best part about them is that you can use _only_ the observable bits using the `asObservable()` function

^ (Let's not talk about threading just yet, cos it makes things complicated) in case someone points out threading