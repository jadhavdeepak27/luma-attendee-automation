import json
import time
from string import Template
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By

# -----------------------------
# Step 0: Configuration
# -----------------------------
CONFIG = {
    "luma_email": "YOUR_LUMA_EMAIL",
    "luma_password": "YOUR_LUMA_PASSWORD",  # Consider using environment variables
    "hubspot_api_key": "YOUR_HUBSPOT_API_KEY",
    "sendgrid_api_key": "YOUR_SENDGRID_API_KEY",
    "sendgrid_from_email": "you@example.com",
    "event_url": "https://luma.com/events/YOUR_EVENT_ID/attendees"
}

# -----------------------------
# Step 1: Scrape Luma Attendees
# -----------------------------
def scrape_luma_attendees():
    """Scrapes Luma attendee list using Selenium"""
    options = Options()
    options.headless = True
    driver = webdriver.Chrome(options=options)
    
    # Login
    driver.get("https://luma.com/login")
    driver.find_element(By.NAME, "email").send_keys(CONFIG["luma_email"])
    driver.find_element(By.NAME, "password").send_keys(CONFIG["luma_password"])
    driver.find_element(By.CSS_SELECTOR, "button[type='submit']").click()
    time.sleep(5)  # wait for login

    # Navigate to event attendee page
    driver.get(CONFIG["event_url"])
    time.sleep(5)
    
    attendees = []
    last_height = driver.execute_script("return document.body.scrollHeight")
    
    while True:
        elements = driver.find_elements(By.CSS_SELECTOR, ".attendee-card")
        for el in elements:
            name = el.find_element(By.CSS_SELECTOR, ".attendee-name").text
            title = el.find_element(By.CSS_SELECTOR, ".attendee-title").text
            company = el.find_element(By.CSS_SELECTOR, ".attendee-company").text
            email = el.get_attribute("data-email")  # if available
            profile_link = el.find_element(By.CSS_SELECTOR, "a.profile-link").get_attribute("href")
            
            attendee = {
                "name": name,
                "title": title,
                "company": company,
                "email": email,
                "profile_link": profile_link
            }
            attendees.append(attendee)
        
        # Scroll to bottom
        driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
        time.sleep(3)
        new_height = driver.execute_script("return document.body.scrollHeight")
        if new_height == last_height:
            break
        last_height = new_height
    
    driver.quit()
    
    # Remove duplicates
    unique_attendees = {a['email'] or a['profile_link']: a for a in attendees}.values()
    
    # Save to JSON
    with open("attendees.json", "w") as f:
        json.dump(list(unique_attendees), f, indent=2)
    
    print(f"[Scrape] Collected {len(unique_attendees)} attendees.")
    return list(unique_attendees)

# -----------------------------
# Step 2: Enrich Attendees (Mocked)
# -----------------------------
def enrich_attendee(attendee):
    """Mock enrichment. Replace with LinkedIn/Twitter/Clearbit APIs"""
    attendee["linkedin"] = "https://linkedin.com/in/" + attendee["name"].replace(" ", "").lower()
    attendee["twitter"] = "@" + attendee["name"].split()[0].lower()
    attendee["email"] = attendee.get("email") or attendee["name"].replace(" ", ".").lower() + "@example.com"
    return attendee

# -----------------------------
# Step 3: Push to CRM (Mocked)
# -----------------------------
def push_to_crm(attendee):
    """Mock CRM push"""
    data = {
        "properties": {
            "email": attendee["email"],
            "firstname": attendee["name"].split()[0],
            "lastname": attendee["name"].split()[-1],
            "company": attendee["company"],
            "title": attendee["title"],
            "linkedin_url": attendee["linkedin"],
            "twitter_handle": attendee["twitter"]
        }
    }
    print(f"[CRM] Pushing to HubSpot: {json.dumps(data, indent=2)}")
    # Actual API call commented out
    # import requests
    # response = requests.post(f"https://api.hubapi.com/crm/v3/objects/contacts?hapikey={CONFIG['hubspot_api_key']}", json=data)

# -----------------------------
# Step 4: Personalized Outreach (Mocked)
# -----------------------------
def send_email(attendee):
    template = Template("""
Hi ${first_name},

It was great seeing you at the event. I loved learning about ${company} and your role as ${title}.

Connect with me on LinkedIn: ${linkedin}

Best,
Your Name
""")
    message = template.substitute(
        first_name=attendee["name"].split()[0],
        company=attendee["company"],
        title=attendee["title"],
        linkedin=attendee["linkedin"]
    )
    print(f"[Email] Would send:\n{message}")
    # Use SendGrid API to send for real

# -----------------------------
# Main Workflow
# -----------------------------
if __name__ == "__main__":
    attendees = scrape_luma_attendees()
    for a in attendees:
        enriched = enrich_attendee(a)
        push_to_crm(enriched)
        send_email(enriched)
