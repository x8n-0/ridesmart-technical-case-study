# Product and pilot

## Who it is for

| Person | What they need | Where they use it |
| --- | --- | --- |
| **Coordinator** | Know who needs a seat, build a fair plan quickly, tell everyone, handle the Saturday-night change | Web dashboard; planner also on mobile |
| **Rider** | Say "I need a ride" in seconds, then know who is coming and when | Member portal on the web, or the app |
| **Driver** | Offer seats, see their riders and route, run the pickups without fumbling | Member portal pickup run, or the app's run screen |
| **Guest** | A visitor or family without an account who needs a seat to an event | Public event link, verified by email |

## Scope decisions

**Churches first.** The first market is church congregations in Western Canada. Other organizations with similar volunteer transport (community groups, schools) are deliberately deferred until the church workflow is proven.

**A free founding pilot.** The first churches use RideSmart free in exchange for feedback, with a paid continuation only once success measures are met. Pricing on the public site is labelled provisional.

**Web first, one-way first.** A readiness review before the first pilot found two large gaps: round trips were not built for members, and nothing had been proven on physical phones. Rather than delay, the pilot started on the web (coordinators on the dashboard, members and drivers in the browser portal) with one-way rides. Both gaps have since been worked on: round trips are built, and the Android app has been tested end to end on a phone.

**The coordinator decides.** The optimizer suggests and automation helps, but sending a plan that leaves someone without a seat requires the coordinator to accept that explicitly, on web and mobile alike, based on the same server count.

## What the pilot changed

Watching real coordinators and members use the product drove most of the work since September:

- **Email for everything that matters.** Web members never saw in-app notices, so invitations, reminders, plans, changes and cancellations are now emailed, respecting per-group preferences.
- **Live pickup run.** Drivers start the run, riders are told their estimated time, pickups are marked with an undo, and the coordinator watches progress on a live page.
- **A map-first planner.** Suggestions from the optimizer on a map, drag a person between cars or onto a stop, change one driver's seats for one ride, make a rider a driver, optimize a car's stop order, and edit a plan after it is sent with the affected people told.
- **Exports the way churches already work.** A plan PDF, large-type sheets for each driver, a CSV, and plain text for the church group chat.
- **Pickup addresses that land in the right place.** Autocomplete everywhere and strict geocoding. See [engineering notes](engineering-notes.md#7-pickups-in-the-wrong-place).
- **Rides home.** A church asked for them, so members can now answer both directions in one form.
- **Small honest fixes.** "Change my answer" became a visible button; the Remind button reports how many people were actually emailed; times show in the church's zone everywhere.

The mobile app followed: a run screen with one fixed action and slide to finish, the coordinator planner with drag and drop, Android push through Firebase Cloud Messaging, and a home screen led by the member's own next ride.

## How the work is run

RideSmart is built by one engineer, so the process exists to keep decisions and status honest rather than to coordinate a team.

- **Written decisions.** 96 dated decision records cover product scope, pricing, safety boundaries, privacy and architecture. Each states the question, context, decision and consequences. Large features begin as a design contract that is reviewed before code is written.
- **A state file that records failures.** After every change, what was done is recorded with the commands run and their results, including what failed and what was not checked. "Tested in Jest" and "tested on a phone" are never written as the same thing.
- **Findings go to an inbox.** Issues noticed outside the current task are written up as notes rather than fixed silently or forgotten.
- **Human approval gates.** Deploys, database migrations, credential changes and anything touching production data need explicit approval, and scripts that change hosted data run as a dry run by default.
- **Periodic reviews.** Specialist reviews of web, mobile, backend, database, quality and shipping, plus a review against general software-engineering principles, feed a single priority list.

## What is next

- A recorded hosted end-to-end run, plus backup-restore and monitoring checks, as the remaining pilot gate.
- iOS device testing and a store-ready build.
- Error tracking and mobile tests in CI.
