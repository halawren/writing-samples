# Documentation writing samples

Hannah Lawrence-Brown

Four documents covering three content types. Three of them document Pennsylvania's supplier registration and procurement systems (a project I most recently worked on with United States Digital Response). The fourth is written against Rippling's published App Shop integration documentation.

**Live pages:** https://halawren.github.io/writing-samples/

## The documents

### Quick start guide — tutorial

[pa-quickstart-guide.html](https://halawren.github.io/writing-samples/pa-quickstart-guide.html)

Takes a small business from unregistered to registered, certified, and finding contract opportunities. Conditional paths by business type (Small Business, Small Diverse Business, Veteran Business Enterprise), prerequisites stated before each step, time estimates per stage, inline definitions for procurement jargon, and a full Spanish translation.

### Systems and identifier reference — reference

[pa-systems-reference.html](https://halawren.github.io/writing-samples/pa-systems-reference.html)

The structural counterpart to the guide. Which system owns what, which identifiers move between them, what states a supplier record can hold, and what each certification status permits. Built around tables and a state model rather than prose.

### Setting up E-Alerts for your NAICS codes — how-to guide

[pa-ealerts-howto.html](https://halawren.github.io/writing-samples/pa-ealerts-howto.html)

One task, end to end, including the part most procedural guides leave out: confirming it actually worked, and diagnosing it when it hasn't. Expected result stated after every step, plus a troubleshooting reference.

### Credential lifecycle and troubleshooting — reference

[rippling/03-credential-lifecycle.md](rippling/03-credential-lifecycle.md)

A reference page drafted for Rippling's App Shop Integrations documentation, consolidating credential lifetimes, reuse rules and renewal paths into a single lifecycle table, with a troubleshooting reference for the installation flow. Written in Markdown with frontmatter to match their existing docs-as-code setup, so it reads as source rather than as a finished page.

## On the content types

A tutorial, a reference and a how-to guide answer different questions and fail in different ways. A tutorial that stops to explain architecture loses the beginner it was written for. A reference page that tries to teach becomes unsearchable. A how-to that assumes nothing turns into a tutorial and stops being useful to the person who already knows the basics.

These were written as a set, with that separation held deliberately, and they cross-reference each other rather than repeating.

## Formats

The three Pennsylvania documents are standalone HTML because they were standalone deliverables: responsive, printable, and self-contained. The Rippling page is Markdown because it was written to drop into an existing documentation pipeline. Reading it on GitHub, as source, is the right way to see it.
