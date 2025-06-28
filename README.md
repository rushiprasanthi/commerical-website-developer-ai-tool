# commerical-website-developer-ai-tool
this is a AI tools this crestes a websites to the user usage: user crestes a websites without coding language 
import markdown2
import os
from pathlib import Path
from jinja2 import Template

# Paths
TEMPLATE_FILE = "template.html"
OUTPUT_DIR = Path("Blog/Pages")

# Ensure output folder exists
OUTPUT_DIR.mkdir(parents=True, exist_ok=True)

def load_template():
    with open(TEMPLATE_FILE, 'r', encoding='utf-8') as f:
        return Template(f.read())

def generate_article(title, slug, meta_desc, md_content, name, company, phone, email, services):
    template = load_template()
    html_body = markdown2.markdown(md_content)

    # Fill template with blog + contact + services data
    html_full = template.render(
        title=title,
        meta=meta_desc,
        content=html_body,
        contact_name=name,
        contact_company=company,
        contact_phone=phone,
        contact_email=email,
        services=services
    )

    output_file = OUTPUT_DIR / f"{slug}.html"
    with open(output_file, 'w', encoding='utf-8') as f:
        f.write(html_full)
    print(f"\n✅ Article saved as: {output_file}")

def main():
    print("\U0001F4C4 Blog Post Generator\n")

    title = input("Enter article title: ")
    slug = input("Enter slug (filename without .html): ")
    meta = input("Enter meta description: ")

    print("\n\U0001F464 Contact Information:")
    name = input("Your Name: ")
    company = input("Company Name: ")
    phone = input("Phone Number: ")
    email = input("Gmail ID: ")

    print("\n\U0001F6E0️ Services Offered:")
    try:
        service_count = int(input("How many services do you want to list? "))
    except ValueError:
        service_count = 0

    services = []
    for i in range(service_count):
        print(f"\n🔹 Service {i+1}")
        s_title = input("Service Title: ")
        s_desc = input("Service Description: ")
        services.append({"title": s_title, "desc": s_desc})

    print("\n📝 Paste your Markdown content. End with a single line: END")
    lines = []
    while True:
        line = input()
        if line.strip() == "END":
            break
        lines.append(line)
    md_content = "\n".join(lines)

    generate_article(title, slug, meta, md_content, name, company, phone, email, services)

if __name__ == "__main__":
    main()
