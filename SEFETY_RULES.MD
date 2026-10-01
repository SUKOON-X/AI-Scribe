# SEFETY RULES

These are non-negotiable and apply to every contributor, human or AI agent.

1. **Never invent clinical facts.** If something was not said, the field stays blank.
2. **Flag, don't guess.** Missing units, ambiguous terms, unmatched drugs and conflicting doses get a visible flag.
3. **Keep the original transcript** as the source of truth.
4. **Every claim in a note links to a transcript span.**
5. **No note leaves the system unsigned** by a doctor.
6. **Signed notes are immutable.** Later changes create an addendum version.
7. **Never call an LLM or ASR provider directly.** Always go through the gateway, which redacts identifiers first.
8. **Never log patient identifiers or raw clinical text** in application logs or traces.
9. **Never store secrets in code.**
10. **Every data access is written to the audit log.**
11. **No diagnosis or treatment suggestions.** The system documents; it does not advise.
12. **All LLM output is validated against Pydantic schemas.** Invalid output is retried, then flagged.
