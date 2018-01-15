We're going to do some mental gynastics here, and it's value won't be immediately visible, but bear with me :)

1. let’s start by firing 1,2,3 successively every few seconds
  ```
   Rx.Observable.create(observer => {
    setTimeout(() => observer.next(1), 1000)
    setTimeout(() => observer.next(2), 2000)
    setTimeout(() => observer.next(3), 3000)
  })
  ```
2. Now let’s do a map, doubling them => 2,4,6
  ```
   Rx.Observable.create(observer => {
    setTimeout(() => observer.next(1), 1000)
    setTimeout(() => observer.next(2), 2000)
    setTimeout(() => observer.next(3), 3000)
  }).map( n => n * 2)
  ```
3. now let’s do map them, but return another observable(these are called higher order observables)
  ```
    Rx.Observable.create(observer => {
    setTimeout(() => observer.next(1), 2000)
    setTimeout(() => observer.next(2), 4000)
    setTimeout(() => observer.next(3), 6000)
  }).map( n => {
    return Rx.Observable.create(observer => {
      setTimeout(() => observer.next(n), 0)     
      })
  })
  ```

4. Now let's make things complicated by adding more events
```
 Rx.Observable.create(observer => {
    setTimeout(() => observer.next(1), 2000)
    setTimeout(() => observer.next(2), 4000)
    setTimeout(() => observer.next(3), 6000)
  }).map( n => {
    return Rx.Observable.create(observer => {
      setTimeout(() => observer.next(n), 0)
      setTimeout(() => observer.next(n * 2), 750)
      setTimeout(() => observer.next(n * 3), 1500)      
      })
  })
```

4. now let’s flatmap
  ```
  Rx.Observable.create(observer => {
    setTimeout(() => observer.next(1), 2000)
    setTimeout(() => observer.next(2), 4000)
    setTimeout(() => observer.next(3), 6000)
  }).flatMap( n => {
    return Rx.Observable.create(observer => {
      setTimeout(() => observer.next(n), 0)
      setTimeout(() => observer.next(n * 2), 750)
      setTimeout(() => observer.next(n * 3), 1500)      
      })
  })
  ```

5. Okay so flatmap was cool. What about flatmapLatest?. Let's go back to the map for a second
```
  Rx.Observable.create(observer => {
    setTimeout(() => observer.next(1), 2000)
    setTimeout(() => observer.next(2), 4000)
    setTimeout(() => observer.next(3), 6000)
  }).map( n => {
    return Rx.Observable.create(observer => {
      setTimeout(() => observer.next(n), 0)
      setTimeout(() => observer.next(n * 2), 750)
      setTimeout(() => observer.next(n * 3), 1500)      
      })
  })
```

  What flatmaplatest does, is, whichever observable is returned last, it only listens to events from that and discards any previous obsrvables. 

7.In order to understand this, Let’s move the 2 and 3 back a little
  ```
   Rx.Observable.create(observer => {
    setTimeout(() => observer.next(1), 2000)
    setTimeout(() => observer.next(2), 3000)
    setTimeout(() => observer.next(3), 4000)
  }).map( n => {
    return Rx.Observable.create(observer => {
      setTimeout(() => observer.next(n), 0)
      setTimeout(() => observer.next(n * 2), 750)
      setTimeout(() => observer.next(n * 3), 1500)      
      })
  })
  ```

8.```
     Rx.Observable.create(observer => {
    setTimeout(() => observer.next(1), 2000)
    setTimeout(() => observer.next(2), 3000)
    setTimeout(() => observer.next(3), 4000)
  }).switchMap( n => {
    return Rx.Observable.create(observer => {
      setTimeout(() => observer.next(n), 0)
      setTimeout(() => observer.next(n * 2), 750)
      setTimeout(() => observer.next(n * 3), 1500)      
      })
  })
  ```
```
Did everyone get this?

Now, why is this important? This was really cool menal gymnastics but why is this important?

Now, requests are basically `Observable<Whatever Data you're getting>`. So, if you have an observable that fires a request ( a button or a pull to refresh or a text field or whatever), and you fire a request whenever the user taps, you only want to listen to the last request. Why? Cos there's no guarantee that if you fire request1 at somt time and fire request 2 half a second later, that you'll receive responses that order(your server is a concurrent system and order isnt guaranteed. some requests take longer to process and some take less time). Sooo, flatmaplatest automagically makes sure you only receive events from the latest observable(aka request), while discarding the last one(which btw also cancels them automatically. How cool is that?)
