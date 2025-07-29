# Get Latest Refreshed Time of Power BI Report

## Requirement
We aim to display the latest refresh timestamp of a Power BI report that uses Import mode. Additionally, the timestamp should reflect the local time zone.

## Challenges
1. The Power BI Service does not respect local time zones in the same way Power BI Desktop does.
2. While the REST API can retrieve the refresh history of the semantic model, integrating the API results into the report is complex.

## Suggested Approach
Within the Power BI report, you can create a blank query and paste the following M code. This query will refresh each time the semantic model is refreshed, effectively capturing the latest refresh timestamp. Moreover, it converts the time to UTC+8, ensuring the timestamp reflects the local time zone when viewed in the Power BI Service.

```powerquery
let
    Source = DateTime.From(DateTimeZone.RemoveZone(DateTimeZone.SwitchZone(DateTimeZone.UtcNow(), 8)))
in
    Source
```
