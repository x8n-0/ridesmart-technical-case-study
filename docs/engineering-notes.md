# Engineering notes

Eight problems from building and running RideSmart, each with what went wrong or could have, what I did, and what it cost.

## 1. Two coordinators, one ride

**Problem.** Planning a ride is slow: read the answers, call the optimizer, let a person look at the result. Meanwhile a member can withdraw, a driver can drop a seat, or a second coordinator can move someone. A plan built on the old picture could seat a rider who said no, overfill a car, or put one person in two cars.

**Approach.** Split planning from committing. All slow work happens with no lock held. Committing opens a short transaction, takes a PostgreSQL advisory lock keyed to the ride, re-reads everything that matters, and re-validates the plan with the same function used for a hand-made plan. Every write that can seat or unseat someone, including the scheduled job, goes through that one boundary.

**Proof.** Race tests run the real route handlers against an in-memory database that interleaves operations at chosen points. Each has a control with the lock disabled that must fail, so the test is known to reproduce the race.

**Cost.** One ride's changes are serialized, which is fine at church scale. The rule "one active car per rider per ride" lives in service code, not a single database constraint, so the shared boundary must never be bypassed.

## 2. The robot and the human must agree

**Problem.** The scheduled job closes replies, plans and finalizes rides on its own. Early on it checked "is everyone seated?" before taking the lock, while the coordinator's Finalize button checked inside it. The two could disagree about the same ride, and a coordinator's cancel could be overwritten by the job closing the ride a moment later.

**Approach.** The job now plans outside the lock, then inside one transaction re-reads the ride, cars and answers and validates with the same function as the assign route. It finalizes behind the same unassigned-rider count as the button. Every status change is conditional on the status it was decided from, so a cancel and an automatic close cannot both succeed. A ride the job cannot finalize gets one retry and then one alert to a coordinator, rather than retrying forever or failing silently.

## 3. Time zone data that lagged the law

**Problem.** Alberta stopped changing its clocks in November 2026, but the time zone database shipped with Node still moved Edmonton back an hour on 1 November. Server times for Alberta churches would have been an hour off, and "on the day of the ride" checks would have refused the last hour of each day.

**Approach.** Use a fixed-offset zone (UTC-6) for Alberta, offered in Settings as "Alberta Time, no clock change", and make it the fallback. A dry-run-by-default script moved the affected organization's setting, checked first for unfinished rides. The mobile app was changed to show ride times in the church's zone, as the web already did. The decision record says to revisit when the time zone database catches up.

**Related.** Weekly rides step their dates from the first occurrence in the church's own zone and carry the time of day forward, so a 9:00 service stays at 9:00 across daylight saving changes.

## 4. Round trips without a migration

**Problem.** One church needed a ride there and a ride home. The obvious design adds direction columns to answers, cars and notifications, which is a large migration on live data.

**Approach.** Before writing code I wrote a contract against the existing schema. An event already grouped rides, with at most one ride per direction. Treating each direction as its own ride meant answers, seat offers, cars and finalization were already per direction. What was missing was behaviour: one form for both directions, one deadline, one invitation, planning the ride home in reverse, and direction labels on every screen and message. The feature shipped with **no new tables or columns**, in small steps, each with tests for the invariants the contract listed.

## 5. "Reminder sent to 20 people"

**Problem.** In the first live run-through on the web, the Remind button said it had reached 20 people. It had reached nobody: web members have no push token and do not open in-app notices.

**Approach.** Invitations and reminders now also go by email, once per person for invitations. Remind is limited to once per ride every six hours, shows the deadline in the church's time, and reports how many people were actually emailed. The lesson stuck: a message on screen must describe what happened, not what was attempted.

## 6. The sign-in page that ad blockers hid

**Problem.** During the same run-through the sign-in page was blank for one tester. The page's CSS classes used an `ad-` prefix, and their ad blocker hid every element matching it.

**Approach.** The classes were renamed. A second issue found that day, a redirect loop after a password change, came from the edge proxy deciding where a signed-in user should go; the page now makes that decision itself, with the session in hand.

## 7. Pickups in the wrong place

**Problem.** Pickup addresses were typed as free text and split on the last comma. A typo or missing city meant a failed save, or worse, a pin in the wrong place for the driver. An edited address could also keep the coordinates of its old text.

**Approach.** Address autocomplete in every place a pickup is entered, with street, city and coordinates submitted together. Editing the text by hand clears the old pin, and a slow autocomplete result cannot overwrite a later choice. On the server, geocoding is strict: partial, coarse or ambiguous matches are refused, and a failed lookup keeps the text with no coordinates rather than a guess. When a car has a pickup with no pin, the planner names that person, and the driver's leave-by time is withheld instead of being computed from an incomplete route.

## 8. A faster member portal

**Problem.** Member pages ran their database reads one after another, and the session and coordinator checks ran more than once per request.

**Approach.** Each page streams a loading skeleton first, the session check is one cached query per request, and the home, ride and pickup-run pages read their data in one parallel batch. Measured on a development server against the hosted database (median of seven loads): home 769 ms to 517 ms, ride page 1,085 ms to 507 ms, with the skeleton on screen in about 240 ms.
