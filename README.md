# Architecture and other Decision Records for Hugo

This Hugo module provides a simple way to publish architecture (and other kinds of) decision records (DRs) as web pages with stable, human-accessible permalinks along other documentation of a project.

This makes **Decision Records more accessible for non tec-savvy stakeholders**.

It can be used for classic Architectural Decision Logs, general Decision Logs supporting different kinds of decision records (such as  requirements, topics, IDTs) as well as for aggregating and managing DRs from different projects.

This hugo module provides templates and partials (for rendering HTML as well as for consistency rules on the front matter..)

## Requirements

Most important requirement:

- Have a stable `human-accessible permalink` to all kinds of decisions.

Other functional requirements:

- The implementation approach should feel “Hugo native”
- Work as an extension to any Hugo theme (no theme lock-in)
- Keep content close to source (Markdown files), readable, and easy to reference
- Support single-project sites with a simple Decision Log
- Support a single site aggregating many groups/projects for broader Architectural Knowledge Management
- Decisions are categorized e.g. as adr, requirement, topic, idt... (decision-types).
- Each decision has a status (e.g. decided, proposed, superseded, deprecated).
- Provide overview pages for:
  - All decisions of a group across projects
  - All decisions of a project
  - Decisions filtered by status

What we want from published `dr`s:

- Permalink of the form `dr/{group}/{project}/{adr|requirement|topic|theme}/make-decision`
- With a simple form `dr/make-decision` for single project with classic Decision Log

## A Decision Record (ADR)

All kinds of decisions records should have the following data items:

```text
name/title
status such as proposed, accepted, rejected, deprecated, superseded, etc. OR  pending, decided, or approved.
style: 
Related decisions:
Related requirements: 
Deciders: [list everyone involved in the decision]
Date: [YYYY-MM-DD when the decision was last updated] 
```

### Decision types

Many aspects of standard- and software development can be expressed as decisions.

Hence several types of decisions are supported

`{adr|requirement|topic|theme|idt}`

- adr: Architecture Decision Record
- requirement: A classical software engineering requirement. It is a decision which requirements to consider. Can also be seen as ASR: architecturally-significant requirement
- topic: 
- theme: More of guiding principles for making decisions.
- idt: Important Technical Decision. Many decisions that are critical in practice are technical but not strictly architectural see https://ignaciolarranaga.medium.com/itds-a-lean-adr-for-executive-technical-decision-making-at-scale-e18bb3f6a563. Main feature: the decision is in the title

### As Markdown with Front Matter

- Applies to any system with Front Matter support (hugo, Jekyll...) 
- Is anyway nice to separate out full structured data from unstructured human text

Rules of thumb:

- Use the front matter variable of your content system and map the Decision Record data accordingly
- consider usability in other systems like your IDEs and coding platforms like Git{Lab,Hub}. However, be aware that your decision records most likely will not leave your system. Hence portability is of minor concern.

Below example is for Hugo static site generator.

The common `dr` specific part is in `params`. 

Example:

```yaml
---
title: "Use Architecture Decision Records"
date: 2024-02-02T04:14:54-08:00 #creation date
description: "adr of..."  # the description is typically rendered within a meta element within the head element of the published HTML file
draft: false # Whether to disable rendering unless you pass the --buildDrafts flag to the hugo command
keywords: ["java"] # An array of keywords, typically rendered within a meta element within the head element of the published HTML file, or used as a taxonomy to classify content 
lastmod: 2024-02-02T04:14:54-08:00
publishDate: 2024-02-02 #The page publication date. Before the publication date, the page will not be rendered unless you pass the --buildFuture flag to the hugo command.
summary: "the summary either summarizes the content or serves as a teaser to encourage readers to visit the page"
params:
  dr: # generell decision record
    type: adr
    # will be used as page content title, if not present page.title will be used 
    name: "Use Architecture Decision Records"
    # mandatory: id without any leading zeros  
    status: "decided"
    related-decisions: 'dr:/adr/use-postgresql'
    related-requirements: 'dr:/requirement/use-postgresql'
---
```

In detail:

title: is the page display title which might have rules common to the website and being determined by an external editor. In order to have a stable decision name we use `params.dr.name`

## Decision Log (ADL)

The normal way of git based ADRs is having folder with ADRs <https://github.com/architecture-decision-record/architecture-decision-record/tree/main#how-to-start-using-adrs-with-git> in the project repository. Usually: `docs/adr`.

In hugo it might be `content/dr`

### Suggested markdown file name convention

- The name has a present tense imperative verb phrase. This helps readability and matches our commit message format.

- The name uses lowercase and dashes (same as this repo). This is a balance of readability and system usability.

examples:

- choose-database.md
- format-timestamps.md
- manage-passwords.md
- handle-exceptions.md

### Within a Hugo site

As simple as having the Decision Record folder within the Hugo `content` folder. Suggested: `content/dr`

Then add an additional `_index.md` to render an overview with a list of `DR`s.

```shell
content/
└── dr/
    ├── _index.md                         (lists all decisions)
    └── use-decision-records.md
```

### Hugo path and DRs as RESTful resources

Given above directory structure hugo generates a logical tree with logical paths <https://gohugo.io/methods/page/path/#logical-tree>

This is the basis for navigating and referencing DRs within hugo.

Additionally, the logical path of some `DR` is the hugo internal and unique name of the `DR` and the basis for the `human-accessible permalink` which fulfils all criteria of a `resource` in terms of RESTFul API design.

With `relative permalink` the host-relative URL of a published resource or a rendered page.

### Maintaining all DRs from different projects in a single Hugo project

In order to have a common pre- path segment for collecting all decisions we choose: `dr` for decision records (of any kind).

In order to keep public and content as close as possible together, the content folder should have the following structure:

```shell
content/
└── dr/
    ├── _index.md                         (lists all organizations/groups)
    ├── org-1/
    │   ├── _index.md                     (lists all projects of group)
    │   ├── project-1/
    │   │   ├── _index.md                 (lists all decisions by type)
    │   │   ├── requirement/
    │   │   │   ├── _index.md
    │   │   │   └── document-req-page.md  (page)
    │   │   ├── adr/                      (architecture decisions)
    │   │   └── topic/                    (topics, any other topic with a conclusion)
    │   └── project-2/                    (same as for project-1)
    ├── org-2/
    └── etc..
```

If all different projects maintain all DRs within a single Hugo project, the link structure is simply reflecting the directory structure:

```bash 
`dr/{group}/{project}/{adr|requirement|topic|theme|...}/
```

Linking and relating each DR is always consistent.

## Towards Architecture Knowledge Management (AKM)

Aggregating DRs from different projects is a first step in AKM.

A more advanced setup would allow each group and project an own Hugo site for its own documentation purpose.

Suggestion: Each project follows the above directory structure.

### Aggregating from different project via e.g. a build system 

The way depends on the build system. However, this approach might allow deviations from the common directory structure.

### Importing DRs using Hugo modules

Here the best way is to have a common directory structure in all projects.

### Keep Links between DR consistent across projects

A challenge is to allow an easy way of linking DRs in Markdown text as well as in front-matter params such as `related-decisions`.

To achieve this allow for a specific "pseudo" protocol in an URI:

Pseudo Links within a project:

dr:/

Pseudo Links within to another project:

dr:/{group}/{project}/adr/make-us-of

e.g.

dr:/another-group/another-project/adr/make-us-of

...

### Consistency Rules

### Editing support

- todo e.g. sveltia

## Contributing

All kinds of contribution are welcome :)

## Authors and acknowledgment

Show your appreciation to those who have contributed to the project.

## License

Apache2