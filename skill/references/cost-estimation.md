# Cost Estimation Reference

Read this file only when you reach the cost-estimate step of a review (context gathered and saved, API call not yet made). It is not needed for routing or argument parsing.

## OpenRouter Pricing (per 1M tokens, approximate)

| Model | Input | Output | Typical Review Cost |
|-------|-------|--------|---------------------|
| DeepSeek V4 Pro (default) | $0.435 | $0.87 | ~$0.002-0.008 |
| GPT 5.5 (council) | $5.00 | $30.00 | ~$0.10-0.40 |
| Gemini 3.1 Pro (council) | $2.00 | $12.00 | ~$0.05-0.18 |
| Grok 4.2 (council) | $2.00 | $6.00 | ~$0.04-0.12 |

**These prices are external vendor claims and they rot.** Treat them as approximations good enough for an order-of-magnitude estimate. If a price matters (big council run, or the user questions a number), verify against https://openrouter.ai/models before quoting it.

## Estimation Formula

```
input_tokens = total_context_chars / 4
output_tokens = ~2500 (typical review length)

# Single model mode (DeepSeek V4 Pro)
single_cost = (input_tokens * 0.435 + output_tokens * 0.87) / 1_000_000

# Council mode (all 3 models in parallel: GPT 5.5 + Gemini 3.1 Pro + Grok 4.2)
council_cost = (input_tokens * (5.00 + 2.00 + 2.00) + output_tokens * (30 + 12 + 6)) / 1_000_000
             ≈ input_tokens * 9.00/M + output_tokens * 48/M
```

## Examples

- Small review (10K chars / 2.5K tokens): Single ~$0.003, Council ~$0.14
- Medium review (50K chars / 12.5K tokens): Single ~$0.008, Council ~$0.23
- Large review (200K chars / 50K tokens): Single ~$0.024, Council ~$0.57

## Reminder: When to Show the Estimate

The estimate is computed from the ACTUAL size of the saved context file, after all gathering is done and before the review script runs. The display template and the confirm/decline flow live in the Cost Estimation section of SKILL.md.
