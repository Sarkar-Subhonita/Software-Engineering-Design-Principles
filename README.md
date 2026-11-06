# Smart Waste Collection Management System

An IoT-based system that automates and optimizes municipal solid-waste collection. Ultrasonic sensors on waste bins report fill levels to a cloud backend, which raises alerts and generates optimized collection routes. Citizens can report overflowing bins from a mobile app.

**Status:** requirements and design phase (SRS v1.0, design analysis report). Implementation details below describe the planned stack.

**Course project:** KIET Deemed to be University, Computer Science Engineering

## Why this project

Manual collection sends trucks to bins that are not full and misses bins that overflow. SWCMS aims to:

- reduce unnecessary collection trips
- prevent bin overflow
- lower fuel and manpower cost
- improve urban sanitation, in line with Smart City objectives

## Features

| Feature | What it does |
| --- | --- |
| Bin fill-level monitoring | Measures fill % at a configurable interval (default 15 min) and sends it with bin ID and timestamp |
| Threshold alerts | Notifies admin and driver apps when a bin reaches a configurable threshold; alerts are logged |
| Route optimization | Builds a route over all bins at or above the threshold (target: 10 s for 200 bins); admin can add or remove bins |
| Admin dashboard | Colour-coded bin map, historical fill trends, collection reports; admin login required |
| Citizen complaints | Submit a complaint with location and optional photo, get a tracking ID, see Pending or Resolved status |

## Architecture

Three tiers:

1. **Device tier:** HC-SR04 ultrasonic sensor + ESP32/Arduino + Wi-Fi/GSM module (e.g. SIM800L), battery or solar powered. Sends data over HTTP or MQTT.
2. **Backend tier:** cloud-hosted Node.js or Django server behind a REST API gateway, with modules for ingestion, authentication, analytics, complaints, routing and alerting. Database: MongoDB or PostgreSQL. Firebase Cloud Messaging for push alerts; OpenStreetMap or Google Maps for routing.
3. **Client tier:** admin web dashboard (Chrome, Firefox, Edge), plus driver and citizen mobile apps (Android 8.0+, iOS 13+) built with Flutter or React Native. All clients use HTTPS REST with JSON.

```
Sensor node --HTTP/MQTT--> Backend (REST API) --HTTPS/JSON--> Admin dashboard
                               |                          \--> Driver app
                               |                           \-> Citizen app
                      Database, FCM, Maps API
```

## Actors

| Actor | Role |
| --- | --- |
| Municipal administrator | Manages the system, configures thresholds, views the dashboard and reports, reviews complaints, generates and adjusts routes |
| Collection staff / driver | Receives alerts, views the assigned route in the mobile app, marks bins as collected |
| Citizen | Reports overflowing bins or requests pickup, tracks complaint status |
| Sensor node (system actor) | Measures fill level and sends readings to the backend at fixed intervals |
| System administrator | Maintains servers, sensors and connectivity |

## Use cases

Admin login is a precondition for all admin use cases (REQ-DB-3).

| # | Use case (actor) | Function | Precondition | Postcondition |
| --- | --- | --- | --- | --- |
| 1 | Report fill level (sensor node) | Measure distance, compute fill %, send with bin ID and timestamp | Node is powered, has connectivity, bin is registered | Reading stored; dashboard status updated; failed reads retried and logged |
| 2 | Receive alert (admin, driver) | Get notified when a bin reaches the threshold | Threshold is set; a reading at or above it has arrived | Alert delivered to dashboard and driver app; alert logged with bin ID and timestamp |
| 3 | View assigned route (driver) | See the day's optimized route as list or map | Driver is logged in; a route has been generated and pushed | Driver sees the ordered bins to collect |
| 4 | Mark bin collected (driver) | Record that a bin has been emptied | Bin is on the driver's route; pickup is done | Bin recorded as collected; related complaints become Resolved |
| 5 | Configure alert threshold (admin) | Set threshold per bin or globally | Admin is logged in | New threshold saved and used for later checks |
| 6 | View bin map and reports (admin) | See colour-coded map, fill trends, collection report for a date range | Admin is logged in; readings exist | Map or report shown; no data changed |
| 7 | Review complaints (admin) | See citizen complaints forwarded to the dashboard | Admin is logged in; at least one complaint exists | Complaint reviewed; its bin can be added to a route |
| 8 | Generate optimized route (admin) | Compute a route over bins at or above the threshold | Admin is logged in; at least one bin is at or above threshold; mapping service available | Route shown on dashboard and pushed to driver app |
| 9 | Adjust route (admin), extends 8 | Manually add or remove a bin | A route has been generated | Updated route saved and pushed; a removed bin returns to Alert state |
| 10 | Submit complaint (citizen) | Report an overflowing bin with location and optional photo | App or web form is open; location is detected or chosen | Complaint logged with unique tracking ID, status Pending, forwarded to dashboard |
| 11 | Track complaint status (citizen) | View Pending or Resolved status | Complaint submitted; tracking ID known | Current status shown |
| 12 | Maintain system (system administrator) | Keep servers, sensors and connectivity running | Has technical access to servers and devices | Services and sensor nodes operating; faults fixed |

## Requirements at a glance

Functional requirements use the ID format `REQ-<Feature>-<Number>`:

- `REQ-BM-1..3` bin monitoring
- `REQ-AL-1..3` alerts
- `REQ-RO-1..3` route optimization
- `REQ-DB-1..3` dashboard and analytics
- `REQ-CF-1..3` citizen complaints

Key nonfunctional targets: 500 concurrent sensor nodes, dashboard loads in 3 s, route in 10 s for 200 bins, 95% transmission success, HTTPS/TLS everywhere, authenticated admin access.

## Open decisions

| ID | Decision | Options |
| --- | --- | --- |
| TBD-1 | Route algorithm | Nearest-neighbor heuristic or Vehicle Routing Problem solver |
| TBD-2 | Mapping provider | Google Maps or OpenStreetMap |
| TBD-3 | Default alert threshold | Set after pilot testing (SRS example: 80%) |

## Documentation

- Software Requirements Specification v1.0
- Design Analysis Report
- UML diagrams: architecture, activity, sequence, collaboration, state transition, component, use case

## Getting started

Implementation is not started yet. This section will cover setup once the code is in place:

- [ ] Backend setup and environment variables
- [ ] Database setup
- [ ] Flashing the sensor firmware
- [ ] Running the dashboard and mobile apps

## License

To be decided.
