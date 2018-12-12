class: center, middle

# Gatling Introduction

---

# Agenda

1. Introduction
2. Setup
3. Simulations

---

# Introduction

## What is Gatling
An automated load testing tool which provides a Scala DSL for describing tests scenarios.

Test scenarios are stored in regular .scala files.

https://gatling.io/docs/current

##Common uses

Testing HTTP services. It also supports JMS, MQTT protocols.

---

#Setup
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

##Caveats
Files storing simulations must live under `test/scala` directory in order to Gatling SBT plugin to find them.

---


#Simulations

## A complete example
```scala
import io.gatling.core.Predef._
import io.gatling.http.Predef._
import scala.concurrent.duration._

class ImageImportFromUrlSimulation {
 // We can pass parameters as system properties
 val serviceBaseUrl = System.getProperty("serviceBaseUrl", "http://localhost:9000")
 val imageUrl = System.getProperty("imageUrl", "http://localhost:9000/photo/no_photo/no_photo.jpeg")

 // HTTP client configuration
 val httpConf = http
   .baseUrl(serviceBaseUrl)
   .acceptEncodingHeader("gzip, deflate")
```

continued...

---

```scala
 // The test scenario
 val scn = scenario("Image import from URL")
   .exec(http("import_from_url")
     .post("/import-from-url") // execute POST to the /import-from-url endpoint
     .header("Content-Type", "application/json")
     .body(StringBody(s"""{ "url": "$imageUrl" }""")).asJson // using this JSON body
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

## Checks
Verify structure of HTTP responses.

HTTP status: `check(status.is(201)))`, `check(status.not(500))`

CSS selector: `check(css("article.more a", "href"))`

More on: https://gatling.io/docs/current/http/http_check

### Capture
Checks also allows to capture data and reuse it in the simulation:

```scala
.exec(http("Search")
  .get("/computers?f=mac")
  .check(css("a:contains('${searchComputerName}')", "href").saveAs("computerURL")))
.pause(1)
.exec(http("Select")
  .get("${computerURL}"))
```

```scala
headerRegex("FOO", "foo(.*)bar(.*)baz").ofType[(String, String)]
```

---

## Assertions
Verify SLAs.

```scala
setUp(scn).assertions(
  global.responseTime.max.lt(50),
  global.successfulRequests.percent.gt(95)
)
```

---

## Run Simulations
```bash
sbt gatling:test
```

## Reports
`sbt gatling:test` should output something like:

```
Reports generated in 0s.
Please open the following file: target/gatling/computerworld-20181211194330455/index.html
```

Here `computerworld-20181211194330455/index.html` contains the reports for the simulation run.

There are many `index.html` report files as simulations defined in the project, and the time of their execution.

---

![screenshot](gatling-report-requests-per-second.png)

---
