# A/B Experiment Brief, RouteLogic (B2B)

## Parameters
| Parameter | Decision |
|---|---|
| Feature under test | Shift Handoff Wizard |
| Persona | The Frontline Driver, A working delivery driver who performs the same core actions dozens of times per shift under real-world physical constraints. |
| Expected outcome | Faster task completion |
| Primary success metric | Time on task reduced by 63% |
| Baseline rate | Current time on task of 11min |
| Guardrail metric | Task completion rate |
| Guardrail boundary | Task completion must not drop below 5% |
| Second guardrail | Error rates should not increase |
| Minimum Detectable Effect | Any size |
| Sample size per arm | 686 |
| Traffic split | 10/10 |
| Test duration | 20min |
| Significance threshold | p < 0.05 |

## Control vs. Variant
- **Control (A):** The current Shift Handoff Wizard. To mark a package delivered, the driver opens the wizard, selects the package, confirms the recipient, adds delivery details, reviews a summary, and submits. That is roughly six screens and taps (illustrative; confirm the exact count from the live app). Median time on task is 11 min. The friction is a physical one: the driver often has no spare hand, no spare time, and no patience left at the doorstep.
- **Variant (B):** dd a single "Mark delivered" button on the delivery screen. One tap confirms the handoff and logs it, replacing the multi-step confirmation path. The same delivery data is still recorded (timestamp, location, driver, package ID), and no fields are removed, so dispatcher reports and downstream systems are unchanged. The only difference between A and B is how the driver confirms delivery.
- **Held constant (isolation check):** App version and release timing, device types and OS versions, the shift log-in screen, route assignment and stop density, fleet mix (drivers randomized within each fleet), driver tenure (balanced across arms), and the screens before and after the handoff.

## Hypothesis
> I believe that Shift Handoff Wizard for The Frontline Driver, A working delivery driver who performs the same core actions dozens of times per shift under real-world physical constraints. will result in Faster task completion, as measured by a Any size change in Time on task reduced by 63% within 20min. We will protect Task completion rate throughout the test.

## Shipping criteria
> We will **ship** if Time on task reduced by 63% improves by ≥ Any size at p < 0.05 and Task completion rate does not reach Task completion must not drop below 5% after 20min.
> We will **iterate** if direction is positive but lift is below the MDE.
> We will **kill** if the primary metric shows no improvement or moves negatively.
> The read date is fixed at the end of 20min, no results reviewed before this date.
