# Submission: Michał Sternik

## Time Spent

Total time: **3h 50 min** (approximate)

## Ticket Triage

### Tickets I Addressed

List the ticket numbers you worked on, in the order you addressed them:

1. **CFG-142**: Fixed the calculatePrice function to return proper timestamp dimension - fixing the timestamp comparison inside usePriceCalculation.ts hook.
2. **CFG-147**: Fixed shared configuration links being overwritten by the initial default configuration - that was broken because of race the condition caused by insufficient timestamp check inside usePriceCalculation.ts hook
3. **CFG-145**: Added a Ctrl+Enter keyboard shortcut for the Quick Add feature using useEffect hook. It's about Accessibility requirements from this specific client. This was also meant inside the task. Not a lot of work - and quick win.
4. **CFG-152**: Improved keyboard accessibility for color switches. Very important as the Accessibility features are the key ones that client cares about (context file).
5. **CFG-148**: Prevented crashes when disabling Include Premium Packaging by clearing dependent add-ons and removing the non null assertion "!" where the addOn is actually null.

### Tickets I Deprioritized

List tickets you intentionally skipped and why:

| Ticket  | Reason                                                                                                                                                                                                                                                                                                         |
| ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| CFG-143 | Hard to reproduce, the source of ticket is not clearly stated. Will take time to investigate. The workaround during demo - just reopen/refresh the window.                                                                                                                                                     |
| CFG-144 | That's the opposite task to CFG-145, which our client cares about. If we do a demo to a client which 'loves' this feature, there's no point of removing it before the demo.                                                                                                                                    |
| CFG-155 | Dark mode support is not critical and would be time consuming. The styles are coded inside style.css file, so the dark theme implementation will involve identification where the change should take place (styles.css and couple places of ProductConfigurator.tsx) and hardcoding new values for dark theme. |
| CFG-154 | The problem applies for specific scenario and the decision shouldn't belong to the developer. The business analyst should agree with the client for one proper version and pass the requirements to the developers.                                                                                            |
| CFG-153 | That's full feature that would involve a lot of time to complete. And a lot of questions for business analyst about the design. Too much for the given timespan.                                                                                                                                               |

### Tickets That Need Clarification

List any tickets where you couldn't proceed due to ambiguity:

| Ticket  | Question                                                                                                                                                                                                                                               |
| ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| CFG-151 | We should have a predefined list of possible errors with the user-friendly texts from business analyst to proceed. We can take this step by ourselves but the client may not like the end result. Better to agree with client about the text contents. |
| CFG-153 | More details about the design and about the use case flow.                                                                                                                                                                                             |

### Tickets With a Misleading Priority Label

Our board labels aren't always right. List tickets where you disagreed with the label (e.g. a Low that's actually High, a High that's actually Low, a Medium that's actually Unclear) and explain why. If you didn't disagree with any, say so.

| Ticket  | Label on board | Your read | Why                                                                                                                                                           |
| ------- | -------------- | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| CFG-152 | ⚪ Low         | 🟠 High   | The task was a quick fix with quick reward. And it should be high priority as the client clearly states that he cares about WCGA 2.1 Accessibility features.  |
| CFG-144 | 🟠 High        | ⚪ Low    | If something works properly but is not used, we should focus on the more critical tasks where something really influences the proper application functioning. |

### Tickets Where You Stopped to Ask Instead of Implementing

If a ticket was ambiguous enough that the right move was to escalate / ask PM / gather more context rather than ship a fix, list it here. We value this signal — picking one interpretation blindly and shipping it can cost more than stopping to ask.

| Ticket  | Who would you ask | What you'd ask                                                                                            |
| ------- | ----------------- | --------------------------------------------------------------------------------------------------------- |
| CFG-151 | Business Analyst  | What are the scenarios of the errors? Do we have a list of texts for error messages that client accepted? |

---

## Technical Write-Up

### Critical Issues Found

Describe the most important bugs you identified:

#### Issue 1: Price doesn't update. Stuck at 0.00$

**Ticket(s):** CFG-142

**What was the bug?**

The price calculation hook was comparing the response timestamp with latestRequestRef`timestamp, but the request tracking value was not a real timestamp. It was just a number (int). Because of that the comparision always failed and the price was not updated.

**How did you find it?**

I followed the price calculation flow from the configuration changes to the usePriceCalculation.ts hook. Used console log to see the return value of calculatePrice function. Noticed, the timestamp is (weirdly) int.

**How did you fix it?**

I changed the request tracking to use Date.now() so the comparision between the current response and the latest request could be done properly (two values of the same dimension).

**Why this approach?**

It seemed like a clear bug. It doesn't seem okay to return 1 or 2 in the place where we should return a timestamp.

---

#### Issue 2: Shared configuration link loads the wrong configuration

**Ticket(s):** CFG-147

**What was the bug?**

When the configurator was opened with a shared configuration link, the default configuration and the configuration decoded from the link were loaded at the same time. The mocked API uses a random response delay between 150 and 800 ms, so the request started first could finish after the other request. The older response could then overwrite the correct configuration loaded from the link.

**How did you find it?**

I followed the configuration loading flow and checked the asynchronous effects responsible for loading the default configuration and the configuration from the URL. The price on the UI was changing weirdly (sometimes from proper value to the wrong one, sometimes in reverse order) The race condition was caused by the random response time from await randomDelay(150, 800). There was no check that the response belonged to the latest configuration request before applying it.

**How did you fix it?**

I added an additional check that compares requestTime with latestRequestRef.current. This verifies that the response belongs to the most recently started request before updating the price state. The existing response timestamp check is also kept, so an older response cannot overwrite the result of the latest request.

**Why this approach?**

The random response delay is intentional and represents the type of timing problem that can happen with real asynchronous requests. Checking the latest request (which is the one that carries the configuration from the link) directly fixes the race condition in the environment with random network delay.

---

### Other Changes Made

Brief description of any other modifications:

Used a useEffect hook with a global keydown listener to handle Ctrl+Enter and Cmd+Enter, call the existing Quick Add handler, and remove the listener when the component unmounts.

Used native button keyboard behavior and tabIndex to make the color switches and quantity controls reachable with the Tab key, and stored the previously focused element in a useRef so focus could be restored after closing a modal.

---

## Code Quality Notes

### Things I Noticed But Didn't Fix

List any issues you noticed but intentionally left:

| Issue                                                                                                                                      | Why I Left It                                                                                                                                                        |
| ------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ProductConfigurator is a large monolithic component that handles rendering, state management, useEffects, and everything else in one file. | Refactoring it into smaller components would improve readability, but it would involve much more time. Not needed before the demo, while there are other priorities. |

### Potential Improvements for the Future

If you had more time, what would you improve?

Extract the helper functions to utils file, extract useEffect invocations to custom hooks. Split the render part into smaller components. Use other library for css management.

---

## Questions for the Team

Questions you would ask in a real scenario:

1. How long do we assume the demo to last?
2. Will we deploy and share the link to the client after the demo?

---

## Assumptions Made

List any assumptions you made to proceed:

1. We have a business analyst. So we can move the responsibility to him in case of any misunderstandings in the website content.
2. "CFG-158: Performance - memoize props in ProductConfigurator dependencies". The task doesn't need code change. The fix involved checking if the proposed solution is applied. And it was. So my assumption - the task was finished beforehand.

---

## Self-Assessment

### What went well?

I was able to understand the main parts of the codebase, identify the most important priorities for the demo, and also spot the tasks with lower priority that should be done first (knowing the context of how our client is).

### What was challenging?

The most challenging part was finding the right solution for the race condition. Another difficult part was deciding which medium-priority tickets should be done first, because there was not enough time to complete everything.

### What would you do differently with more time?

I would add focused tests for the async configuration and pricing flows before making further changes, so the timing-related fixes could be verified without relying mainly on manual reproduction.

---

## AI Assistance Notes (optional)

If you used AI tools (Copilot, Claude, Cursor, ChatGPT, etc.), we'd love a short note on how. **We don't score the fact that you used AI — we score the quality of your reasoning and verification.**

Useful things to share:

- What kinds of prompts you sent (paraphrase is fine, no need to paste logs)
- Where the AI helped and where you overrode it
- Anything the AI suggested that you rejected — and why
- Anything you verified manually that the AI got wrong

---

## Additional Notes

Anything else you want us to know:

[Your notes]
