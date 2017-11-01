1. let’s start by firing 1,2,3 successively every few seconds
  ```
   Rx.Observable.create(observer => {
    setTimeout(() => observer.next(1), 1000)
    setTimeout(() => observer.next(2), 2000)
    setTimeout(() => observer.next(3), 3000)
  }).subscribe()
  ```
2. Now let’s do a map, doubling them => 2,4,6
  ```
   Rx.Observable.create(observer => {
    setTimeout(() => observer.next(1), 1000)
    setTimeout(() => observer.next(2), 2000)
    setTimeout(() => observer.next(3), 3000)
  }).map( n => n * 2).subscribe()
  ```
3. now let’s do map them to an observable
  ```
   Rx.Observable.create(observer => {
    setTimeout(() => observer.next(1), 1000)
    setTimeout(() => observer.next(2), 2000)
    setTimeout(() => observer.next(3), 3000)
  }).map( n => {
  	return Rx.Observable.create(observer => {
  		observer.next(n * 2)
      })
  }).subscribe()
  ```
4. now let’s flatmap
  ```
   Rx.Observable.create(observer => {
    setTimeout(() => observer.next(1), 1000)
    setTimeout(() => observer.next(2), 2000)
    setTimeout(() => observer.next(3), 3000)
  }).flatMap( n => {
  	return Rx.Observable.create(observer => {
        setTimeout(() => observer.next(n), 300)
  	  setTimeout(() => observer.next(n * 2), 600)
  	  setTimeout(() => observer.next(n * 3), 900)      
      })
  }).subscribe()
  ```
5. now let’s do a flatmapLatest. Not much difference, right? . 
  ```
   Rx.Observable.create(observer => {
    setTimeout(() => observer.next(1), 1000)
    setTimeout(() => observer.next(2), 2000)
    setTimeout(() => observer.next(3), 3000)
  }).flatMapLatest( n => {
  	return Rx.Observable.create(observer => {
        setTimeout(() => observer.next(n), 300)
  	  setTimeout(() => observer.next(n * 2), 600)
  	  setTimeout(() => observer.next(n * 3), 900)      
      })
  }).subscribe()
  ```
6. Let’s move the 2 back a little
  ```
   Rx.Observable.create(observer => {
    setTimeout(() => observer.next(1), 1000)
    setTimeout(() => observer.next(2), 1500)
    setTimeout(() => observer.next(3), 2000)
  }).flatMapLatest( n => {
  	return Rx.Observable.create(observer => {
        setTimeout(() => observer.next(n), 300)
  	  setTimeout(() => observer.next(n * 2), 600)
  	  setTimeout(() => observer.next(n * 3), 900)      
      })
  }).subscribe()
  ```


Notice how we ignore values from "1" after we receive "2", and ignore values from "2" once we receive "3". Now, requests are basically `Observable<Whatever Data you're getting>`. So, if you have something that returns a request, and an Observable that acts as a trigger (text field for search or a timer), you basically only make 1 request at a time (the latest one) and only get responses from that one. Pretty neat, right?
