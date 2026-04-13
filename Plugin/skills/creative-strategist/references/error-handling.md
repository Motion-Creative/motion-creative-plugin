# Error Handling

## Degraded Mode

If any tool call fails (timeout, auth error) or returns empty results:
- Note the gap explicitly (e.g., "Demographic data unavailable — analysis based on creative-level data only")
- Never fabricate performance data or fill gaps with assumptions
- If a core tool (get_creative_insights) fails entirely, say so and suggest the user verify their Motion workspace connection
- If all tools fail, stop and report the failure
- Proceed with whatever data is available — a partial analysis with clear caveats is better than a fabricated complete one
- For empty results: suggest broadening the datePreset or checking spend thresholds

## Auth Failures
- If get_auth_context fails: stop and ask user to check their Motion connection
- If a specific tool returns 401/403: note which tool failed and suggest reconnecting

## Thin Data
- If fewer than 3 creatives return: note the limited data, suggest broadening datePreset or checking spendThreshold
- If demographics return empty: skip demographic analysis, note the gap
- If glossary returns empty: proceed without taxonomy insights, note the gap
