class: center, middle

# Introduction to Gatling

---

# Introduction to Gatling

## Who am I?
**Emiliano Menéndez**, backend services developer at Tienda Nube.

## What is Gatling?
An automated load testing tool for HTTP services.

---

# Agenda

* General definitions
* Introduction to Gatling
* Setup
* Main concepts
* Live Demo
* Conclusions

---

# General definitions

**Load test**: A process that checks how a software component behaves under heavy usage.

**DSL**: Domain-Specific Language, a high-level language specialized to a particular application domain.

**Scala**: An object oriented, functional programming language.

**SBT**: Scala Build Tool, similar to Maven, Gradle, Gulp.

---

# Introduction to Gatling

**Gatling** (https://gatling.io) is an automated load testing tool which provides a Scala DSL for describing tests scenarios, known to Gatling as simulations.

## Common uses

Load test HTTP services performance on heavy load.

In addition to HTTP, Gatling supports JMS and MQTT protocols as well.

## Where did I use it?

As part of the image upload service development, to load test the implementation.

---

# Setup

Gatling can be executed:

* stand-alone using a CLI
* as part of ScalaTest tests
* with an SBT plugin

---

# Setup

## SBT plugin
In this presentation I'll show the setup of the SBT plugin.

`plugins.sbt`:
```scala
addSbtPlugin("io.gatling" % "gatling-sbt" % "3.0.0")
```

`build.sbt`:
```scala
lazy val root = (project in file(".")).enablePlugins(PlayScala, GatlingPlugin)

libraryDependencies += "io.gatling.highcharts" % "gatling-charts-highcharts" % "3.0.0" % "test,it"
libraryDependencies += "io.gatling"            % "gatling-test-framework"    % "3.0.0" % "test,it"
```

**IMPORTANT:** Simulation files must live under **`test/scala`** directory in order to Gatling SBT plugin to be able to find them.

---

# Main concepts

.center[
## Simulations
## Checks
## Assertions
]

---

# Simulations

## A concrete test case
I want to test how an image repository service responds to mass uploads.

## How do I describe the test with Gatling?
Writing a **Simulation**.

In Gatling, a simulation is the load test of a use case, written in Scala.

A simulation might be thought as a script defining sequences of HTTP requests and the expected results.

Simulation code is stored in regular .scala files.

---

# Simulations

## Simulation code

```scala
class ImageImportFromUrlSimulation {
 // HTTP client configuration
 val httpConf = http
    .baseUrl("http://localhost:9000")
    .acceptEncodingHeader("gzip, deflate")

 val imageUrl = "http://localhost:9000/photo/no_photo/no_photo.jpeg"

 // The test scenario
 val scn = scenario("Image import from URL")
   .exec(http("import_from_url")
     .post("/import-from-url") // execute POST to the /import-from-url endpoint
     .header("Content-Type", "application/json")
     .body(StringBody(s"""{ "url": "$imageUrl" }""")).asJson
     .check(status.is(201)))

 setUp(
   scn.inject(rampUsers(300) during (10 seconds))
 ).protocols(httpConf)
   .assertions(
     global.successfulRequests.percent.is(100) // Al requests should succeed
   )
}
```

---

# Simulations

## Generating requests

```scala
setUp(
  scn.inject(rampUsers(300) during (10 seconds))
)
```

Other injection expressions:

.remark-code[
* atOnceUsers(numberOfUsers)
* constantUsersPerSec(rate) during (duration)
* rampUsersPerSec(rate1) to (rate2) during(duration)
]

---

# Simulations

## Checks
Verify the structure of HTTP responses.

```scala
 val scn = scenario("Image import from URL")
   .exec(http("import_from_url")
     .post("/import-from-url") // execute POST to the /import-from-url endpoint
     .header("Content-Type", "application/json")
     .body(StringBody(s"""{ "url": "$imageUrl" }""")).asJson
     .check(status.is(201)))
```

**HTTP status**: `check(status.is(201)))`, `check(status.not(500))`

**CSS selector**: `check(css("article.more a", "href"))`

More on: https://gatling.io/docs/current/http/http_check

---

# Simulations

## Capture
Checks also allows to capture data and reuse it in the simulation:

```scala
.exec(http("Search")
  .get("/computers?f=mac")
  .check(css("a:contains('${searchComputerName}')", "href").saveAs("computerURL")))
.pause(1)
.exec(http("Select")
  .get("${computerURL}"))
```

---

# Simulations

## Assertions
Assertions verify Service Level Agreements for the HTTP services being tested.

They are composed of: metric + condition:

```scala
setUp(scn).assertions(
  global.responseTime.max.lt(50), // time in milliseconds
  global.successfulRequests.percent.gt(95)
)
```

---

# Simulations

## Metrics

.remark-code[
* responseTime: target the response time in milliseconds.
* allRequests: target the number of requests.
* failedRequests: target the number of failed requests.
* successfulRequests: target the number of successful requests.
* requestsPerSec: target the rate of requests per second
]

## Conditions

Conditions compare **the value** of the metric and the **threshold**:


.remark-code[
* lt(threshold): the value is less than the threshold.
* lte(threshold): the value is less than or equal to the threshold.
* gt(threshold): the value is greater than the threshold.
* gte(threshold): the value is greater than or equal to the threshold.
* between(thresholdMin, thresholdMax): the value is between two thresholds.
* between(thresholdMin, thresholdMax, inclusive = false): same as above but doesn’t include bounds
* is(value): the value is equal to the given value.
* in(sequence): the value of metric is in a sequence.
]

---

# Misc notes

* Simulation class files must live under **test/scala** directory (otherwise Gatling doesn't find them)

* We can pass parameters to the simulation using system properties

```scala
class ImageImportFromUrlSimulation {
 val serviceBaseUrl = System.getProperty("serviceBaseUrl", "http://localhost:9000")

 val httpConf = http.baseUrl(serviceBaseUrl).acceptEncodingHeader("gzip, deflate")
```

---

class: center, middle

# Live Demo

---

# Demo

## Run all Simulations

```bash
sbt gatling:test
```

## Run a single Simulation

```bash
$ sbt
```
and then inside SBT:
```bash
> gatling:testOnly computerdatabase.BasicSimulation
```

---

# Demo

## Reports
`sbt gatling:test` should output something like:

```bash
Reports generated in 0s.
Please open the following file:
target/gatling/computerworld-20181211194330455/index.html
```

Here `computerworld-20181211194330455/index.html` contains the simulation result report.

There are many `index.html` report files as simulations defined in the project, and the time of their execution.

---

![screenshot](gatling-report-requests-per-second.png)

---
# Conclusions

## In the real world, unit tests are not enough.

## But we can do something about it!

.center[![](one-does-not-simply-write-unit-tests.png)]

.footnote[(Picture credit: https://automationpanda.com/2017/05/18/can-performance-tests-be-unit-test/)]

---

class: center, middle

# Thanks!
