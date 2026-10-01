# AGENT INSTRUCTIONS
These are common instrcution of an agent across all scenarios

## PROJECT OVERVIEW

A web application that listens to a doctor's consultation or dictation in Hindi, English, or Hinglish, converts it into standard medical English and produces a structured clinical note and prescription that the doctor reviews, edits, and signs.

## Build and Test Command

## GENERAL GUILDLINES
1. Never use the em dash "—". Use plain dash "-" instead
2. Never writing commit messages, NEVER auto-add your agent name as co-author
3. When writing or substantialy editing long markdown files, put each pull sentence on its own line. Preserve normal markdown structure, but avoid wrapping multiple sentences onto one physical line. 
4. When making technical decisions, do not give much weight to development cost. Instead prefer quality, simplicity, robustness, scalability and long term maintainability.
5. When doing bug fixes, always start with reproducing the bug in an E2E setting as closely aligned with how an end user would experience it as possible. This make sure you find the real problem so your fix will actually solve it.
6. When end-to-end testing a product, be picky about the UI you see and be obsessed with pixel perfection.
If something clearly looks off, even if it is not directly related to what you are doing, try to get it fixed along the way.
7. Apply that same high standard to engineering excellence: lint, test failures, and test flakiness.
If you see one, even if it is not caused by what yoou are working right now, still get it fixed.

## SEFETY RULES
@SEFETY_RULES.MD

## Code Style Guidelines

## Testing Instruction

## Security Considerations

## Extra Instruction
- DO NOT: Dowload any LLM model locally
