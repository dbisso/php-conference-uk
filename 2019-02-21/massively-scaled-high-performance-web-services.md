#Massively Scaled High Performance Web Services with PHP

Demin Yin

Slides: https://www.slideshare.net/DeminYin/massively-scaled-high-performance-web-services-with-php-132696547

---

## Background

Design Home - an iOS and Android interior design simulation game. 
- 1M DAU. 40M votes daily.
- 100,000 API calls / minute ~ 56ms response time

### Architecture

All clients connect through main gateway which talks to database and microservices

Talk focused on how the 'Inbox' microservice was optimised
- Average 14k rpm, peak 34k, response < 9ms
- PHP7, opcache, APCu, distributed cache, chained cache

## Improvements
### Web Server

- Decouple microservices. Easier to upgrade dependencies (esp. PHP), smaller API surface to test and refactor
- Dockerise the microservices. Make them self-contained and easier to manage. Multiple, hand-built base images: HTTP (FPM), Worker (CLI), Async (Swoole), immutable tags. Easier to debug
- Tooling: monitoring - NewRelic, BugSnag, CloudWatch, security - SonarQube,SensioLabs Security Checker

### HTTP

Improve background processing (error, notifications, reporting etc)

Strategies:
- External binary exec
- Register shutdown functions
- Job/Queue server
- fastcgi_finish_request(): works under FPM to flush output to client whilst allowing script to finish execution. Useful for end-of-request activities but still blocks FPM worker and potentially locks resources and sessions

Shutdown functions are run first in order and then destruct methods in non-deterministic order

Important considerations: 
- Validate data before triggering
- Back-off and retry
- Reporting for potentially silent failures

Resources
- https://github.com/Crowdstar/background-processing
- https://github.com/deminy/background-processing-in-php

Compression

- Gzip in nginx is good but needs correct config.
- Make sure we honour proxies
- Don't compress small responses - counterproductive. `gzip_min_length 1280`. Fit response within MTU
- Always set Content-Length otherwise Nginx will always compress which may be undesirable
- Last-Modified, If-Modified-Since, If-None-Match, Cache-Control, Etag etc and return 304
 
### Data Layer

Redis (transient) + Couchbase (persistent)

NoSQL considerations
- Limitations on types of queries
- Indices are difficult
- Serialization and compression have to be considered to get best performance and storage
  - save compressed messages, templated messages, short field names, message TTL

### Hardware

Previously - sync and switch.
- Failed/interrupted requests
- Hard to isolate bugs without reverting and redeploying

Switched to AWS ECS for quicker, deployments, request draining and rollback
- Multiple environments for test, dev, RC, staging and prod
- Keep instances close (eg API + database in same AZ), use internal networking

## Future

- Async / Swoole
- HTTP2 / Tars (https://github.com/TarsCloud/Tars/blob/master/Introduction.en.md)