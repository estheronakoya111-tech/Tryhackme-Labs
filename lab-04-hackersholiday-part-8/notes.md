# TryHackMe — Hackers Holiday Part 8

## Topic

**Race Conditions — Burp Suite**

## What I Did

I worked on Part 8 of the **Hackers Holiday** room on TryHackMe. The lab demonstrated a **race condition**, where multiple requests are processed at nearly the same time and can cause unexpected behavior when the application does not properly handle concurrent requests.

I used **Burp Suite** to investigate and reproduce the issue.

### Approach

* Intercepted the relevant request using Burp Suite.
* Sent the request to **Repeater**.
* Created multiple requests from the same request.
* Sent the requests concurrently rather than waiting for each request to finish before sending the next one.
* Observed that the application's state changed unexpectedly.
* The balance/reward increased to around **$400**, allowing me to complete the required condition and retrieve the flag.

## What I Learned

A race condition can occur when an application performs a check and an action separately without properly protecting the operation from simultaneous requests.

Instead of:

```text
Check → Update → Finish
```

multiple requests can interact with the same state:

```text
Request A ─┐
Request B ─┤
Request C ─┼──→ Application
Request D ─┤
Request E ─┘
```

If the application does not handle this safely, requests may interfere with one another and produce an unintended result.

## Tool Used

* Burp Suite

  * Repeater
  * Multiple/concurrent requests

## Key Takeaway

The important lesson from this lab was that **sending the same request repeatedly is not necessarily the same as exploiting a race condition**. The timing and concurrency of the requests are important.

I also learned how Burp Suite can be used to test how an application's state-changing operations behave when several requests are sent at nearly the same time.

## Lab Result

**Completed the lab and retrieved the flag.**

> Note: I followed a tutorial while solving the lab, so I reviewed the technique afterward to understand why the race condition worked rather than treating the steps as something I discovered independently.
