# PM API Literacy Lab — Umbrella Weather Check

A hands-on **n8n workflow literacy lab for product managers and product leaders**.

The workflow answers a deliberately simple product question:

> **Should I carry an umbrella tomorrow?**

The point is not the weather recommendation itself. The point is to make the plumbing behind a small product capability visible:

**human input → API call → structured response → data mapping → second API call → product rule → branch → result**

It is designed as a companion exercise for learning how APIs and workflow orchestration fit together from a product perspective.

---

## What This Lab Teaches

The workflow demonstrates several concepts that can remain surprisingly abstract when they are only described in requirements or architecture diagrams:

- A user-facing value such as a **city name** may not be what a downstream service requires.
- One API can translate that value into the input contract required by another API.
- Data returned by one service can be mapped directly into the next service call.
- A technically successful API response is not necessarily a usable **product** result.
- Product rules often live between systems rather than inside either system.
- Failure paths are part of product behavior, not merely engineering exceptions.
- One workflow can be invoked manually for learning or exposed as a callable interface.

The n8n canvas effectively becomes an **executable architecture diagram**.

---

## Scenario

The workflow starts with a city, geocodes it, retrieves weather data, selects tomorrow's forecast, and applies a simple product rule:

> If tomorrow's maximum precipitation probability is greater than **60%**, recommend bringing an umbrella.

```text
City
  ↓
Open-Meteo Geocoding API
  ↓
Latitude + Longitude
  ↓
Open-Meteo Forecast API
  ↓
Tomorrow's Forecast
  ↓
Precipitation Probability > 60%?
  ↙                         ↘
Yes                          No
 ↓                            ↓
Bring an umbrella     Probably fine without one
```

---

## Why Two APIs?

The user thinks in a human-friendly value such as `Stamford`. The forecast API expects coordinates.

So the workflow first calls a geocoding API to translate the city into latitude and longitude. Those values then become inputs to the forecast API.

> **The output contract of one interface becomes the input contract of another.**

---

## APIs Used

This lab uses [Open-Meteo](https://open-meteo.com/).

### Geocoding

```http
GET https://geocoding-api.open-meteo.com/v1/search
```

Parameters:

```text
name = <city>
count = 1
language = en
format = json
```

### Forecast

```http
GET https://api.open-meteo.com/v1/forecast
```

The workflow requests:

**Current conditions**

```text
temperature_2m
apparent_temperature
precipitation
```

**Daily forecast**

```text
temperature_2m_max
temperature_2m_min
precipitation_probability_max
```

and `timezone=auto`.

The Open-Meteo endpoints used in this learning exercise do not require an API key or paid subscription.

---

## Two Ways to Start the Workflow

### 1. Manual Trigger — Learning Mode

This is the easiest way to explore the workflow. The default city is `Stamford`.

Click **Execute workflow** in n8n, then click backward through the executed nodes to inspect:

1. the city input,
2. the geocoding request,
3. the geocoding JSON response,
4. extracted latitude and longitude,
5. the forecast request,
6. the forecast response,
7. tomorrow's selected values,
8. the IF decision,
9. the final recommendation.

Change the city in **Manual Input — Stamford** to experiment.

### 2. Webhook Trigger — Optional Postman Caller

The same workflow can also be called externally.

```http
GET <n8n-webhook-url>?city=Stamford
```

Open **Webhook Trigger — Optional Postman Caller** in n8n and copy its Test or Production URL. Use the Test URL while developing. The Production URL is appropriate once the workflow is active/published for your n8n environment.

The final JSON result is returned through **Respond to Webhook — Return JSON to Postman**.

---

## Where Product Logic Enters

The two external APIs do not decide whether you should carry an umbrella. The workflow does.

The rule is implemented in:

**IF — Rain Probability > 60%?**

Try changing the threshold to 40% or 80%. Nothing about either API changes. Only the **product rule** changes.

> The IF node is implementation. The threshold is a product decision.

A real weather product could also consider rainfall amount, duration, time of day, user preference, wind, activity type, or forecast uncertainty.

---

## Why the Workflow Selects Index `1`

Open-Meteo returns daily forecast fields as parallel arrays.

For this exercise:

```text
index 0 = today
index 1 = tomorrow
```

The **Code — Select Tomorrow + Shape Product Data** node makes that hidden transformation explicit and reduces the larger provider response to fields the product actually needs.

---

## Failure Paths

The workflow intentionally includes more than the happy path. It handles examples such as:

- missing or blank city input,
- a city that does not return usable coordinates,
- geocoding service failure,
- forecast service failure,
- missing tomorrow forecast data.

Those paths create readable structured errors with fields such as `error_code`, `message`, and `http_status`.

That creates useful product questions:

- What should the user see?
- What should be logged?
- Should the system retry?
- Should several location matches be offered?
- Should another provider be used as a fallback?
- Which failures are technical errors versus normal product states?

---

## A Useful Ambiguity Exercise

Change the city to `Springfield`, then temporarily change the geocoding request from `count=1` to `count=5`.

Several legitimate locations may appear. A supposedly simple requirement such as “let the user enter a city” now creates real product questions about disambiguation, defaults, location context, and user choice.

---

## This Workflow Intentionally Stops at Node Output

The manual learning path ends at **Manual Result — Inspect Output**. The result is simply visible as structured JSON inside n8n.

The optional webhook path returns the same JSON to Postman.

This repository intentionally does **not** build:

- a webpage,
- a mobile interface,
- a dashboard,
- a notification,
- a smart-device action,
- persistent storage,
- or an AI/model handoff.

In a real application, one of those downstream actions would often be the actual purpose of the workflow. The result might later render in a UI, trigger a notification or device, be stored, feed another workflow, or become context for an AI model.

Those are deliberately out of scope so the API and orchestration mechanics remain easy to inspect.

---

## Requirements

You need:

- an n8n instance,
- internet access from n8n to the Open-Meteo APIs.

No Open-Meteo API key is required for this learning workflow, and no paid third-party integration is required.

---

## Importing the Workflow

1. Download the workflow JSON from this repository.
2. Open n8n.
3. Create a new workflow or use n8n's workflow import option.
4. Import the JSON file.
5. Review the sticky notes on the canvas.
6. Click **Execute workflow** to run the manual learning path.

The imported canvas includes instructional sticky notes and node annotations explaining what each area is meant to demonstrate.

---

## Suggested Walkthrough

1. Run the default Stamford example.
2. Inspect the geocoding response.
3. Find latitude and longitude.
4. Inspect how those fields feed the forecast request.
5. Look at Open-Meteo's daily arrays.
6. Inspect the node that selects tomorrow.
7. Change the rain threshold.
8. Change the city.
9. Try a location that does not resolve.
10. Try the webhook version from Postman.
11. Change Springfield to return multiple geocoding results and think through the product implications.

The goal is not to become an n8n expert. The goal is to become more comfortable reasoning about what actually happens when one product capability crosses several interfaces.

---

## Relationship to Postman

Postman is useful for examining an individual API conversation:

```text
request → response
```

n8n makes the next layer visible:

```text
request
  ↓
response
  ↓
data mapping
  ↓
another request
  ↓
business logic
  ↓
product outcome
```

That is the shift from **API literacy** toward **workflow and interface literacy**.

---

## Product Questions to Ask While Exploring

- Who initiates each interaction?
- What does each system require?
- Which fields actually matter to the product?
- Where is data transformed?
- Where does product logic enter?
- What counts as success?
- What happens when an upstream system fails?
- What happens when it succeeds technically but returns unusable data?
- How many external calls does one customer action create?
- Where would authentication and credentials live in a production version?
- What downstream system would ultimately consume the result?

Those questions are more important than memorizing the location of any particular n8n button.

---

## Cost and External Services

A normal successful run makes:

```text
1 geocoding API request
1 forecast API request
```

A production feature might add an AI/model call, notification service, database, analytics, identity service, or additional enrichment APIs.

One visible customer action can therefore fan out into many downstream operations. That is both an architecture concern and potentially a product-economics concern.

---

## Disclaimer

This is an educational workflow. The umbrella recommendation is intentionally simplistic and should not be treated as meteorological or safety advice.

The workflow is intended to teach API contracts, orchestration, mapping, branching, errors, and product decision points.

---

## License

Add the license appropriate for how you want others to reuse the workflow and accompanying material. If no license is included, normal copyright rules apply.
