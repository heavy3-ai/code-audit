# Cost Estimation Reference

Read this file only when you reach the cost-estimate step of a review (context gathered and saved, API call not yet made). It is not needed for routing or argument parsing.

## OpenRouter Pricing (per 1M tokens, approximate)

| Model | Input | Output | Typical Review Cost |
|-------|-------|--------|---------------------|
| DeepSeek V4 Pro (default) | $0.435 | $0.87 | ~$0.002-0.008 |
| GPT 6 Sol (council) | $2.00 | $10.00 | ~$0.03-0.13 |
| Gemini 3.1 Pro (council) | $2.00 | $12.00 | ~$0.05-0.18 |
| Grok 4.7 (council) | $1.60 | $4.80 | ~$0.03-0.10 |

**These prices are external vendor claims and they rot.** Treat them as approximations good enough for an order-of-magnitude estimate. If a price matters (big council run, or the user questions a number), verify against https://openrouter.ai/models before quoting it.

## Estimation Formula

```
input_tokens = total_context_chars / 4
output_tokens = ~2500 (typical review length)

# Single model mode (DeepSeek V4 Pro)
single_cost = (input_tokens * 0.435 + output_tokens * 0.87) / 1_000_000

# Council mode (all 3 models in parallel: GPT 6 Sol + Gemini 3.1 Pro + Grok 4.7)
council_cost = (input_tokens * (2.00 + 2.00 + 1.60) + output_tokens * (10 + 12 + 4.80)) / 1_000_000
             = input_tokens * 5.60/M + output_tokens * 26.80/M
```

## Examples

- Small review (10K chars / 2.5K tokens): Single ~$0.003, Council ~$0.08
- Medium review (50K chars / 12.5K tokens): Single ~$0.008, Council ~$0.14
- Large review (200K chars / 50K tokens): Single ~$0.024, Council ~$0.35

## Reminder: When to Show the Estimate

The estimate is computed from the ACTUAL size of the saved context file, after all gathering is done and before the review script runs. The display template and the confirm/decline flow live in the Cost Estimation section of SKILL.md.
