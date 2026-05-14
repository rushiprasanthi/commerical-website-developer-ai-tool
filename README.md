
# Commercial Website Developer — Static Blog / Landing Page Generator

Project overview
----------------
What it does
- Interactive CLI tool that accepts article metadata, contact info, service descriptions, and Markdown content to render a single static HTML page using a Jinja2 template.
- Outputs the generated file to Blog/Pages/<slug>.html ready to be served as a static landing page or blog post.

Real-world problem it solves
- Rapidly create SEO-friendly, shareable, static landing pages or blog posts without a full CMS.
- Useful for solo developers, agencies, or consultants who want to generate marketing pages from Markdown and share them quickly (e.g., GitHub Pages, CDN, or local file).

Why it matters
- Removes friction of hand-coding marketing pages.
- Keeps content in Markdown while enabling consistent branding and layout through templating.
- Provides a reproducible, automatable foundation that can be integrated into CI/CD or used as a lightweight SSG for microsites.

Key engineering goals
- Simplicity and predictability: deterministic static HTML output from structured input.
- Maintainable template-driven rendering with clear extension points (new templates, partials).
- Portable: single-file application with minimal dependencies.
- Extensible: intended to be upgraded into a batch generator, REST wrapper, or GitOps pipeline.

Core features
-------------
Detailed feature breakdown
- Interactive CLI (blog.py main): prompts for title, slug, meta description, contact details, services, and Markdown content (terminated with line "END").
- Markdown processing: uses markdown2 to convert Markdown into HTML safely.
- Template rendering: Jinja2 template (template.html) to compose the blog content into a branded landing page.
- Output management: writes generated HTML files to Blog/Pages/, creating directories as needed.

Intelligent workflows & automation potential
- Template-driven content → separation of content vs presentation enables batch generation and automated deployments.
- Can be integrated into a Git-based workflow: generate HTML then commit to a `gh-pages` branch via CI for instant publication.
- Batch ingestion: adapted to read markdown files from a directory and metadata YAML/JSON for non-interactive generation.
- CI automation: generate pages on PR merge or when content changes, then deploy to GitHub Pages or an S3-backed website.

Operational capabilities & engineering highlights
- Deterministic generation and idempotent outputs (same inputs → same HTML output).
- Ease of localization by providing template slots for dynamic content.
- Lightweight dependency footprint (Jinja2 + markdown2).

System architecture
-------------------
High-level overview
- Single-process static site generator using:
  - Input layer (CLI)
  - Processing layer (Markdown → HTML conversion)
  - Presentation layer (Jinja2 template)
  - Output layer (filesystem under Blog/Pages)

ASCII diagram — logical flow
```
+----------------+     +--------------------+     +------------------+     +------------------+
|   User / CI    | --> |  blog.py (CLI/API) | --> |  markdown2 engine | --> |  Jinja2 template |
| (interactive)  |     |  (ingest metadata) |     |  (md -> html)     |     |  rendering HTML  |
+----------------+     +--------------------+     +------------------+     +------------------+
                                                                  |
                                                                  v
                                                         Blog/Pages/<slug>.html
```

Request lifecycle (single generation)
1. User runs blog.py (interactive or could be programmatic).
2. CLI collects metadata (title, slug, meta description), contact info, services (list), and markdown body.
3. markdown2 converts the Markdown body to HTML.
4. Jinja2 template is rendered with the converted HTML and metadata.
5. File is written to Blog/Pages/<slug>.html. Logging prints success.

Processing pipeline & dataflow
- Input: standard input prompts → structured Python objects (strings, lists of service dicts).
- Validation: minimal (basic int parsing for number of services) — recommended to add stricter validation.
- Transform: markdown2.markdown(md_content) → HTML string.
- Render: template.render(mapping) → final HTML.
- Persist: write to disk.

Service/module interactions
- blog.py imports:
  - markdown2.markdown: Markdown -> HTML
  - jinja2.Template: render UI
  - pathlib.Path, os: filesystem operations
- template.html is pure presentation with Jinja placeholders.

Technical stack & rationale
--------------------------
Table: primary technologies

| Layer | Technology | Version (locked) | Why chosen |
|---|---:|---:|---|
| Templating | Jinja2 | 3.1.6 | Mature, expressive templating; widely used in Python web and SSG contexts. |
| Markdown | markdown2 | 2.5.1 | Lightweight, predictable Markdown -> HTML conversion for simple content workflows. |
| Language | Python | 3.8+ recommended | Ubiquitous scripting language, excellent ecosystem for automation & CI. |
| Packaging | pip / requirements.txt | — | Minimal footprint to ensure easy installation across environments. |

Why these choices
- Small dependency surface reduces security and maintenance overhead.
- Jinja2 provides straightforward extensibility to support multiple themes and partials.
- Python-based generator is easily integrated into build pipelines or wrapped into services (FastAPI/Flask) for multi-tenant generation.

Folder structure
----------------
Inferred/proposed top-level tree (professional view):

/
├─ Blog/                    # Output/usage folder (created by generator)
│  └─ Pages/                # Generated HTML pages (Blog/Pages/<slug>.html)
├─ blog.py                  # CLI generator: main program
├─ template.html            # Jinja2 template used to render article + contact + services
├─ requirements.txt         # Python dependency pins
├─ README.md                # <— you are editing this file
└─ (optional) Dockerfile, .github/workflows/ — recommended additions

Explaination of important files/folders
- blog.py — core CLI; orchestrates input, markdown conversion, template render, and file write.
- template.html — HTML layout, controls branding and presentation. Replace or add new templates to skin output.
- Blog/Pages — generated outputs (commit these to a deploy branch for hosting).
- requirements.txt — reproducible environment.

Workflow / Execution pipeline (step-by-step)
--------------------------------------------
1. Environment setup:
   - Create a virtualenv and install dependencies.
2. Run generator:
   - Start blog.py, answer prompts.
3. Markdown ingestion:
   - Type or paste Markdown, finish with a line containing only END.
4. Post-processing:
   - markdown2 converts content; Jinja2 renders.
5. Output:
   - Final HTML is written to Blog/Pages/<slug>.html.
6. (Optional) Publish:
   - Add/commit generated files to repository and push to a `gh-pages` branch or push to a static hosting target (S3/CloudFront).

Module / API documentation
--------------------------
Primary module: blog.py

Public functions
- load_template() -> jinja2.Template
  - Reads `template.html` from repo root and returns a Template instance.

- generate_article(title, slug, meta_desc, md_content,
                   name, company, phone, email, services)
  - Parameters:
    - title (str) — page title
    - slug (str) — filename (without `.html`)
    - meta_desc (str) — meta description content
    - md_content (str) — raw Markdown body
    - name/company/phone/email (str) — contact fields
    - services (list[dict]) — service items; expected dict keys: title, description
  - Behavior:
    - Converts md_content -> HTML via markdown2.
    - Calls Jinja2 template.render with mapping to produce full HTML.
    - Writes to Blog/Pages/<slug>.html (creates the output dir if needed).
  - Output:
    - Creates/overwrites file Blog/Pages/<slug>.html and prints a success message.

- main() — entrypoint for CLI
  - Prompts user for all required fields, collects services in a loop, reads Markdown until "END", and calls generate_article.

Important internal notes / gotchas
- Template expects service objects with `service.title` and `service.description`. In blog.py, services are appended as {"title": s_title, "desc": s_desc}. This key mismatch will cause services to render empty descriptions. Fix is simple (see "Bugfix & small improvements" below).

Installation guide
------------------
Prerequisites
- Python 3.8+ (recommend 3.10+)
- Git (optional for cloning)

Steps (Unix/macOS)
```bash
# 1. Clone the repo
git clone https://github.com/rushiprasanthi/commerical-website-developer-ai-tool.git
cd commerical-website-developer-ai-tool

# 2. Create and activate venv
python -m venv .venv
source .venv/bin/activate

# 3. Install dependencies
pip install --upgrade pip
pip install -r requirements.txt
```

Windows (PowerShell)
```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

Configuration
- There are no external API keys required for the core functionality.
- To adapt the generator to read from files or a metadata store, create a JSON/YAML reader and pass structured objects to generate_article().

Running the project
-------------------
Development (interactive)
```bash
python blog.py
```
Follow the prompts:
- Title, slug, meta description
- Contact info
- Number of services → enter each title and description
- Paste Markdown content; finish with a line: END

Example session (concise)
```
Enter article title: How to Scale Microservices
Enter slug (filename without .html): scale-microservices
Enter meta description: Practical guide to scaling microservices
Your Name: Rushi
Company Name: Acme Inc
Phone Number: +1-555-1234
Gmail ID: rushi@example.com
How many services do you want to list? 2
Service 1 Title: Architecture Review
Service 1 Description: We provide architecture reviews.
Service 2 Title: DevOps Consulting
Service 2 Description: CI/CD and infra as code.
Paste your Markdown content. End with a single line: END
# Heading
Content goes here
END
```

Output
- Generated file: Blog/Pages/scale-microservices.html
- Open in browser or serve with a static server:
```bash
python -m http.server --directory Blog/Pages 8000
# visit http://localhost:8000/scale-microservices.html
```

Production mode / Automation
- Run non-interactively by adapting blog.py to accept CLI flags or JSON/YAML input.
- Example non-interactive call pattern (recommended extension):
  - `python blog.py --metadata post.yaml --content post.md --template template.html --out Blog/Pages/`

Scalability & engineering considerations
---------------------------------------
Despite being a simple generator, the architecture can scale to support production workloads.

Performance optimizations
- I/O bound: writing files is fast; for many pages generate concurrently using a worker pool (multiprocessing or async if network I/O involved).
- Caching: if converting identical Markdown repeatedly, cache results keyed by content hash.
- Template compilation: compile Jinja2 templates once and reuse compiled Template objects rather than reloading for every page in batch mode.

Scalable hosting & CDN
- Generated static sites are perfect for CDNs (CloudFront, Fastly, Netlify, GitHub Pages) — scale horizontally with near-zero server cost.

Fault tolerance & retries
- For batch generation that writes to remote stores (S3), implement exponential backoff and idempotent writes (write to temp key then rename).
- For CI commits/pushes, implement pre-commit checks and retry transient git HTTP errors.

Queueing & orchestration
- For large-scale generation (thousands of pages), add a queue (Redis, RabbitMQ) and workers that consume page-generation jobs.
- Orchestrate with Airflow or GitHub Actions for scheduled updates or periodic content rebuilds.

Database & search
- Store metadata in a light DB (SQLite → Postgres) when needing indexing, tags, and search.
- Use an indexer (Algolia, ElasticSearch) for full-text search across many articles.

Horizontal scaling opportunities
- Convert generator to a stateless service (containerized) behind a job-queue — easy horizontal scale by adding more worker containers.

Challenges & engineering decisions
---------------------------------
Design tradeoffs
- Simplicity vs features: The project favors a small surface area and minimal dependencies to maximize portability and reduce security risk.
- Templating choice: Jinja2 is powerful and familiar; supporting multiple theme engines is deferred to future work.
- Interactivity vs automation: Current flow is interactive for quick manual content creation. To support automation, a JSON/YAML ingestion mode is recommended.

Key technical challenges addressed
- Deterministic static output ensures consistent deployment artifact for pipeline usage.
- UX: interactive CLI removes context switching for non-technical users to create blog posts.

Known issues & small bugfix
- Template expects `service.description` but blog.py sets `desc` key:

  - Problematic line (in blog.py):
    ```python
    services.append({"title": s_title, "desc": s_desc})
    ```

  - Template references:
    ```jinja
    <p>{{ service.description }}</p>
    ```

  - Quick fix (update blog.py):
    ```python
    # change from:
    services.append({"title": s_title, "desc": s_desc})

    # to:
    services.append({"title": s_title, "description": s_desc})
    ```

  - Alternatively, change template to `{{ service.desc }}` — but keeping `description` is clearer and explicit.

Future improvements (roadmap)
---------------------------
Short-term (low-effort, high-impact)
- Add CLI flags and file-based input (JSON/YAML) for batch generation.
- Fix the services key mismatch and add unit tests for template rendering.
- Add a small test harness (pytest) to validate render outputs.

Medium-term
- Add a REST API wrapper (FastAPI) to accept POST requests with metadata and return generated HTML (or store it to a hosting target).
- Add a sample Dockerfile & GitHub Actions workflow to build, test, and optionally publish generated pages to `gh-pages` or an S3 bucket.
- Add multiple templates (themes) and partial includes to support customization.

Long-term / Advanced
- Multi-tenant generation with per-tenant templates and branding assets.
- Integrate with an ML-powered content assistant:
  - Suggest SEO meta description using a simple language model prompt.
  - Auto-generate summary bullets for services via an LLM (careful with API key handling).
- Real-time preview service with hot reload (development server).
- Full content pipeline: ingest Markdown → run accessibility linter → convert → run HTML minification → deploy.

Operationalizing: Docker + Example GitHub Actions
------------------------------------------------
Example Dockerfile (production generation container):
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt /app/
RUN pip install --no-cache-dir -r requirements.txt
COPY . /app
CMD ["python", "blog.py"]
```

Example GitHub Actions skeleton (generate and deploy to gh-pages)
- Add `.github/workflows/generate-and-deploy.yml` to:
  - Run tests
  - Generate pages (non-interactive mode)
  - Commit generated files to `gh-pages` branch
  - Push and publish via GitHub Pages

Contribution
------------
Contributions welcome — suggested contribution flow:
1. Fork the repository.
2. Create a feature branch (e.g., `feature/cli-args`).
3. Add tests where applicable.
4. Open a PR with a descriptive title and rationale.

Please follow these guidelines:
- Keep changes atomic and well-documented.
- Include unit tests for core logic (markdown conversion + template mapping).
- Ensure templates are backward compatible when changing keys used by blog.py.

License
-------
This repository does not currently include a license file. For maximum adoption and to demonstrate professionalism, consider adding an OSI-approved license such as MIT.

Suggested LICENSE (MIT):
```text
MIT License

Copyright (c) 2026 rushiprasanthi

Permission is hereby granted, free of charge, to any person obtaining a copy...
```
(Replace the ellipsis with the full MIT text and commit as LICENSE.)

Author
------
- Rushi Prasanthi (GitHub: rushiprasanthi)
- Project: Commercial Website Developer — Static Blog Generator

