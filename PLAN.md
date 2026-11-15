# Complete Planner — Programming Plan

**Architecture:** MVC. The model is plain Java with no UI or file I/O. Build it and test it first, then add the view.
**Core idea:** an `Entry` = *what* it is (Event or Task) + *when* it happens (`Placement`). A Task holds a list of `TaskComponent`s.

## File tree

```
complete-planner/
├── pom.xml
└── src/
    ├── main/java/com/amanda/planner/
    │   ├── App.java                      # wires model + controller + view
    │   │
    │   ├── model/
    │   │   ├── Entry.java                # abstract: UUID id, title, Placement
    │   │   ├── Event.java                # class/meeting (attended, not completed)
    │   │   ├── Task.java                 # completed flag + List<TaskComponent>
    │   │   ├── Placement.java            # sealed interface: LocalDate date()
    │   │   ├── TimeSlot.java             # record(start, end) → fixed block
    │   │   ├── Floating.java             # record(date, approxTime) → "around 3pm"
    │   │   ├── DayOnly.java              # record(date) → "sometime today"
    │   │   ├── Planner.java              # interface: the model's public API
    │   │   ├── PlannerImpl.java          # stores entries, date-range queries
    │   │   └── component/
    │   │       ├── TaskComponent.java    # sealed interface
    │   │       ├── Checklist.java        # title + List<ChecklistItem>
    │   │       ├── ChecklistItem.java    # text, done, quantity?, timer?
    │   │       ├── TaskTimer.java        # countdown: duration + state
    │   │       ├── TimerState.java       # enum NOT_STARTED/RUNNING/PAUSED/FINISHED
    │   │       └── Note.java             # free text
    │   │
    │   ├── model/layout/
    │   │   ├── DayLayout.java            # segments + day-only task list
    │   │   ├── DaySegment.java           # sealed: BlockSegment | GapSegment
    │   │   ├── BlockSegment.java         # record(Entry)
    │   │   ├── GapSegment.java           # record(start, end, List<Task>)
    │   │   └── DayLayoutBuilder.java     # places Floating tasks into gaps
    │   │
    │   ├── service/
    │   │   ├── TimerService.java         # ticks timers, fires alerts (uses Clock)
    │   │   └── TimerListener.java        # observer: onTimerFinished(...)
    │   │
    │   ├── persistence/
    │   │   ├── PlannerRepository.java    # interface: save / load
    │   │   └── JsonPlannerRepository.java
    │   │
    │   ├── controller/
    │   │   ├── PlannerController.java    # user actions → model calls
    │   │   ├── ViewMode.java             # enum DAY/THREE_DAY/WEEK/MONTH
    │   │   └── DateRange.java            # record(from, to)
    │   │
    │   └── view/
    │       ├── PlannerView.java          # interface
    │       ├── TextView.java             # console view for testing
    │       └── fx/                       # JavaFX view (last)
    │
    └── test/java/com/amanda/planner/     # mirrors main/ — one test class per class
```

## Build order

| # | Build | Test that… |
|---|-------|------------|
| 1 | `Placement`, `TimeSlot`, `Floating`, `DayOnly` | `TimeSlot` rejects end ≤ start; `date()` is correct for each placement type |
| 2 | `Entry`, `Event`, `Task` | tasks complete/uncomplete; placement can be swapped |
| 3 | `TaskComponent`, `Checklist`, `ChecklistItem`, `Note` | add/remove/check items; progress like "3/5" |
| 4 | `TaskTimer`, `TimerState` (pass in a `Clock`) | start → pause → resume → finish, using a fixed clock |
| 5 | `Planner`, `PlannerImpl` | add/remove by id; `getEntries(from, to)` includes boundary dates |
| 6 | `DayLayoutBuilder` + layout classes | gaps between blocks; floating tasks land in the right gap; overlaps; empty day; tasks after the last block |
| 7 | `ViewMode`, `DateRange` | DAY/3-DAY/WEEK/MONTH ranges from a focus date |
| 8 | `PlannerController` + `TextView` | interact with your real model in the console |
| 9 | `TimerService`, `TimerListener` | listener fires when a timer runs out |
| 10 | `PlannerRepository`, `JsonPlannerRepository` (add Gson) | save → load gives back an equal planner |
| 11 | JavaFX view (add `javafx-controls` + `javafx-maven-plugin`) | manual testing |

**Later:** recurring events (`RecurrenceRule` on `Event`), more components (`TimeLog`, `Counter`, `Link`, `Reminder`), task templates.

**Run tests:** `.\mvnw.cmd test`
