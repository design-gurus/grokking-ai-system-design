# Contributing

Thanks for helping improve this free AI system design interview guide. Contributions of all sizes are welcome: fixing a typo, clarifying an explanation, adding a building block, or writing a new question walkthrough.

## Ground rules

- Keep it original. Do not paste content from paid courses (including the DesignGurus courses) or other copyrighted sources. Write explanations in your own words.
- Keep it concise and practical. Favor clear summaries, tables, and bullet points over long essays.
- Stay vendor-neutral in the technical explanations. Link to the course as the "go deeper" option, not as the only answer.
- No em dashes. Use commas, colons, parentheses, or periods instead. (House style.)
- No year references in the content. They date the page and the material outlives them.
- Prices and model names age fast. Prefer the shape of a calculation over a quoted price, and prefer a category over a product name.

## How to add a building block

1. Copy `building-blocks/_template.md` to `building-blocks/your-block.md`.
2. Fill in each section.
3. Add a row to the right table in `building-blocks/README.md`, and to the root `README.md` if it is one of the core blocks.

## How to add a question walkthrough

1. Copy `questions/_template.md` to `questions/design-your-system.md`.
2. Follow the seven steps. Do not substitute the classic system design framework, because the data, evaluation, and cost steps are the point of this repository.
3. Keep the solution at the "approach and trade-offs" level, not a full paid-grade solution.
4. Add a row to the correct family table in `questions/README.md`.

## How to add a deep dive

1. Copy `deep-dives/_template.md`.
2. Keep it at interview altitude: the idea, the trade-off, and the sentence to say. Not a paper summary.
3. Add a row to the table in `deep-dives/README.md`.

## Style

- One concept per file.
- Define every term the first time it appears.
- Short sentences. If a sentence runs past about 25 words, split it.
- Write the number, not the adjective. "About 30 GB" beats "a lot of memory".
- Use relative links between files so they work on GitHub.
- Prefer simple ASCII diagrams that render everywhere.

## Pull requests

Open a pull request with a short description of what you changed and why. Small, focused pull requests are easier to review and merge. The link checker runs on every pull request and fails on a broken relative link.
