# PROMPTS.md: Living Prompt Pack

> Module 3 · Prompt Chaining. Re-architect the build with prompt chains; capture the reusable ones here.

## How to use this pack

_Each prompt is a reusable step. Chain them: the output of one becomes the input to the next._

## Prompt chain: [name your flow]

### Step 1: Expand, build new screens in a strict sequence
```
Build the next phase of this app in a strict sequence:
1. Add a screen for me to update my calendar.  Match the layout and spacing of the dashboard screen
2. Add a screen for me to visualize a specific day entry. Match the data density of the attached dashboard screen
3. Add my rating also on the sidebar
4. Navigation: write the logic so I have the possibility to update my calendar and to visualize my day bookings.

Build these in order so I open the dashboard and have on the menu a button so I can check my calendar and another  for the day bookings to be able to accept or cancel sessions.
```

### Step 2: Behavior, hard-code the states
```
Apply the following logic constraints to the {{screen}} flow:
- Use skeleton screens for the screens loading state. Add a loading delay of 1 second in order to visualize the loading happening
- If no data is present, on the dashboard show No data available. If it's on the calendar, show text with "Add new times for your availability." And if is day booking screen show "No bookings yet"  
- On fetch failure, trigger the error state: "Please try again later"

Maintain the same design language throughout and tether all behavior strictly to these rules.
```

### Step 3: Refine, one surgical polish
```
The calendar screen needs a professional polish.
Wrap the week days for a row or a grid in case there is not enough room to expand
Do some polishing on visuals. 

Don't change anything else in the project or touch the underlying logic.
```

## Reusable techniques learned

- _____
- _____

## What broke (and the fix)

_Where a single mega-prompt failed and chaining fixed it._

_____
