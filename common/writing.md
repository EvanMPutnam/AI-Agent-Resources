# 1. Writing

## 1.1 Orwell's Six Rules
All prose — documentation, comments, design docs, commit messages, and chat responses — should hold to George Orwell's six rules from "Politics and the English Language":

1. Never use a metaphor, simile, or other figure of speech which you are used to seeing in print.
2. Never use a long word where a short one will do.
3. If it is possible to cut a word out, always cut it out.
4. Never use the passive where you can use the active.
5. Never use a foreign phrase, a scientific word, or a jargon word if you can think of an everyday English equivalent.
6. Break any of these rules sooner than say anything outright barbarous.

### Applying Them
- Rule 1 rules out worn-out phrasing: "at the end of the day", "moving the needle", "low-hanging fruit". It also rules out AI-isms — "delve into", "it's worth noting", "what matters".
- Rule 2 favors "use" over "utilize", "start" over "commence", "help" over "facilitate".
- Rule 3 is the strongest one. Cut hedges ("basically", "essentially", "quite"), throat-clearing openers, and restatements of the heading.
- Rule 4 means "the parser reads the file", not "the file is read by the parser". Passive is allowed when the actor is unknown or irrelevant.
- Rule 5 permits genuine technical terms — "mutex", "idempotent", "TLS" — where the everyday equivalent would be longer or vaguer. It rules out jargon used as decoration.
- Rule 6 is the escape hatch. Clarity wins over the other five rules; never mangle a sentence to obey them.

## 1.2 AI Writing Habits to Avoid
LLM prose has recognizable tics. Strip them out.

- **Filler words.** "Basically", "essentially", "simply", "quite", "very", "really", "actually", "just". Delete them; the sentence rarely changes meaning.
- **Throat-clearing.** "It's worth noting that", "it's important to understand", "let's dive into", "in this section we will". Start with the point.
- **Rule of three.** Not every list needs three items. Don't pad a pair to a triple or trim a genuine list of five down to three for rhythm. List what exists.
- **Significance inflation.** "Critical", "crucial", "vital", "essential", "key", "fundamental", "game-changing". If everything is critical, nothing is. Reserve these for things that actually break when ignored.
- **Promotional adjectives.** "Powerful", "robust", "seamless", "elegant", "comprehensive", "rich", "intuitive". Describe behavior instead: say what the thing does, not how good it is.
- **Copula avoidance.** "X serves as a cache" / "X acts as a wrapper" / "X functions as the entry point" — write "X is a cache". Don't dodge the verb "to be".
- **Negative parallelism.** "It's not just X, it's Y." "This isn't about X — it's about Y." Say Y.
- **Summary restatement.** Closing paragraphs that repeat what was just said. If the reader needed a summary, the section is too long.
- **Hedged conclusions.** "This may potentially help in some cases." Commit or leave it out.
- **Em-dash pileups and tricolon rhythm.** Vary sentence structure; don't fall into the same cadence every paragraph.
- **Emoji and decorative headers.** No emoji in docs, comments, or commit messages unless the project already uses them.
- **Trailing participial clauses.** "Refactored the parser, ensuring maintainability and improving performance." The clause carries no information. Cut it, or make it a real sentence with a real claim.
- **Over-structuring.** Bulleting prose that should be a paragraph, a header for every two sentences, a nested list one item deep. Structure should reflect structure that exists.
- **Analogy reflex.** "Think of it like a post office." Explain the mechanism instead. Use an analogy only when the mechanism is genuinely unfamiliar to the reader.
- **Question headers.** "Why does this matter?" "What's next?" Usually a sign the section has nothing to say.
- **Fake precision.** Invented percentages, "studies show", "significantly faster" with nothing measured. Give the number and where it came from, or drop the claim.
- **Uniform rhythm.** Every bullet the same length, every paragraph three sentences. Let the length follow the content.
