---
layout: post
title:  "🧠 Lazy Loading HTTP Requests in Angular with RxJS and BehaviorSubject"
date:   2025-07-03 12:00:00 +1000
categories: [angular, frontend]
tags: [lazy loading, http, angular, performance]
permalink: /blog/2025/Lazy-Loading-HTTP-Requests-in-Angular/
---

When building modern Angular apps, it’s common to have services that fetch data from HTTP endpoints. But you don’t always want those calls to fire immediately, instead, you might want **lazy loading**: only fetch the data when it’s actually needed.

In this post, we’ll walk through how to **lazily fetch and cache data in Angular using RxJS**, and how to **extend the pattern across multiple endpoints** using a generic helper method.

---

## 🚀 The Problem

You want to:

* Trigger an HTTP request only when data is needed.
* Cache the result to avoid unnecessary network calls.
* Share the same data across multiple subscribers.
* Support more than one endpoint without duplicating logic.

---

## ✅ Solution: `BehaviorSubject` + RxJS + Generic Lazy Loader

We’ll use:

* `BehaviorSubject<T | null>` to hold the cached data.
* `switchMap()` and `tap()` to lazily fetch from HTTP.
* A **generic helper method** to apply this pattern to multiple endpoints.

---

## 🔧 The Service (with Two Endpoints)

```ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { BehaviorSubject, Observable, of } from 'rxjs';
import { switchMap, tap } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class MyDataService {

  private dataSubject = new BehaviorSubject<any | null>(null);
  private data2Subject = new BehaviorSubject<any | null>(null);

  constructor(private http: HttpClient) {}

  // Lazy-loads and caches /api/my-data
  getData(): Observable<any> {
    return this.lazyLoad(this.dataSubject, '/api/my-data');
  }

  // Lazy-loads and caches /api/another-data
  getData2(): Observable<any> {
    return this.lazyLoad(this.data2Subject, '/api/another-data');
  }

  // Generic lazy loading and caching method
  private lazyLoad<T>(subject: BehaviorSubject<T | null>, url: string): Observable<T> {
    return subject.pipe(
      switchMap((cached) => {
        if (cached !== null) {
          return of(cached); // Return cached value
        } else {
          return this.http.get<T>(url).pipe(
            tap(response => subject.next(response)) // Cache the response
          );
        }
      })
    );
  }
}
```

---

## 🧪 In a Component

```ts
@Component({
  selector: 'app-my-component',
  template: `
    <div *ngIf="data$ | async as data; else loading">
      {{ data | json }}
    </div>
    <ng-template #loading>Loading...</ng-template>
  `
})
export class MyComponent {
  data$ = this.dataService.getData();

  constructor(private dataService: MyDataService) {}
}
```

For a second component:

```ts
export class AnotherComponent {
  data2$ = this.dataService.getData2();
}
```

---

## 🧠 Why This Pattern Works

| Feature      | How it Works                                                |
| ------------ | ----------------------------------------------------------- |
| **Lazy**     | No HTTP call until `getData()` is first subscribed          |
| **Cached**   | `BehaviorSubject` holds the result for reuse                |
| **Reactive** | Multiple components can subscribe and receive the same data |
| **Reusable** | `lazyLoad()` can be used for any number of endpoints        |

---

## 🔁 Bonus: Add a Refresh Method

If you want to force a re-fetch:

```ts
refreshData(): void {
  this.http.get('/api/my-data').subscribe(data => this.dataSubject.next(data));
}
```

Or, make it generic:

```ts
private refresh<T>(subject: BehaviorSubject<T | null>, url: string): void {
  this.http.get<T>(url).subscribe(data => subject.next(data));
}
```

---

## 🧩 When to Use This Pattern

✅ Works great for:

* Caching static or rarely changing data
* Settings, metadata, or lookup tables
* Reusing data across multiple components

⚠️ Avoid for:

* Real-time or frequently updated data
* Parameterized calls where each param has different data
* Streaming APIs (use `Subject` or `WebSocket` instead)

---

