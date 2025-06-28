# 🧠 Commercial Website Developer AI Tool

This project is an AI-powered tool that helps automate the creation of **commercial websites** using Python. Designed to streamline the process of content posting and static site generation, it's perfect for developers, bloggers, and businesses looking to speed up web development.

---

## 🚀 Features

- ✍️ Simple input-based blog creation
- 🧱 Static HTML page generation
- 🎨 Custom templates using `template.html`
- 📦 Automatically saves content with filename and metadata
- 📃 Markdown-style input supported
- ⚙️ Easy to modify, deploy, and expand

---

## 🛠️ Technologies Used

- Python 3.x
- Jinja2 (for HTML templating)
- markdown2 (for Markdown conversion)
- Basic HTML & CSS (template.html)

---

## 📦 Installation

1. **Clone the repo**

```bash
git clone https://github.com/rushiprasanthi/commerical-website-developer-ai-tool.git
cd commerical-website-developer-ai-tool
pip install -r requirements.txt
python blog.py
You enter:

Title

Slug (filename)

Meta description

Markdown content
you name
you contact
your gmail
your comapany name
The script uses a pre-defined template.html and generates a clean static HTML blog page inside the Blog/Pages/ folder
├── blog.py                # Main script
├── template.html          # HTML layout template
├── requirements.txt       # Required Python libraries
├── README.md              # Project documentation
└── Blog/
    └── Pages/             # Output HTML pages
