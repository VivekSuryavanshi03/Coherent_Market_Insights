# n8n Google Maps Scraper and Data Storage Workflow

This project contains two connected **n8n workflows**. Together, they allow you to scrape information from Google Maps, process the data, and automatically store it in Google Sheets in a structured format.  

The setup has been divided into two parts:  

1. **Independent Workflow** – This is the entry point. It listens for a message or trigger, then connects to a Google Maps scraping tool.  
2. **Dependent Workflow** – This is executed only after the first workflow. It takes the scraped data, processes it step by step, and finally saves it in Google Sheets.  

---

## 📌 Workflow 1: Google Maps Scraper (Independent)

This is the workflow that starts the whole process.  

![Independent Workflow](./images/workflow1.png)

**Trigger Node:**  
- The workflow begins with **When Chat Message Received** (or any other trigger you prefer). This node simply waits until a user input or signal comes in.  

**Flow of Nodes:**  
1. **Trigger** – Captures the user request (for example, "Find all dermatologists in Pune").  
2. **Google Maps Scraper Tool** – Connects with an external service that can scrape Google Maps data such as names, addresses, contact numbers, and ratings of businesses.  
3. **Pass Data** – Once the data is collected, it is passed to the second workflow for further processing and storage.  

This workflow’s main responsibility is only to get the raw data. It does not store or format the results.  

---

## 📌 Workflow 2: Data Processing and Storage (Dependent)

This workflow runs only when triggered by the first workflow. Its job is to take the raw data, clean it up, and save it into Google Sheets.  

![Dependent Workflow](./images/workflow2.png)

**Step-by-Step Explanation of Nodes:**  

1. **When Executed by Another Workflow**  
   - This is the starting point. It waits for Workflow 1 to finish and provide the scraped data.  

2. **HTTP Request**  
   - Sends a request to Apify (or any other service you have connected) to confirm or fetch the latest scraping job results.  

3. **HTTP Request 1**  
   - A follow-up request to pull all the detailed data that came from the scraper.  
   - This is where the actual Google Maps business information is retrieved in raw JSON format.  

4. **Code Node**  
   - The raw JSON data is often messy and contains more details than needed.  
   - This node extracts only the useful fields (like address, name, contact, rating, etc.) and reshapes the data into a cleaner format.  

5. **HTTP Request 2**  
   - If additional data from an external service is required, this node handles it.  
   - For example, it can enrich the scraped records or validate details before storing them.  

6. **Code 1**  
   - This is another cleanup or transformation step.  
   - It makes sure that the data is properly structured into rows and columns, matching the format of the Google Sheet where it will be stored.  

7. **Prompt (Manual Mapping)**  
   - A simple step where the data is aligned with the exact column names of the Google Sheet.  
   - Think of it as preparing the final version of the record before saving it.  

8. **Append or Update Row in Google Sheets**  
   - This node takes the cleaned data and directly adds it into a Google Sheet.  
   - If a row already exists, it can update it; otherwise, it will append a new row.  

9. **Aggregate Node**  
   - If multiple results are coming in (for example, dozens of doctors or clinics), this node gathers them into a proper list so everything can be written neatly into the sheet.  

---

## 🚀 How the Two Workflows Work Together

1. You send a request or trigger the first workflow with a query (for example, “Find neurologists in Pune”).  
2. The first workflow runs the Google Maps scraper and collects the raw data.  
3. The scraped data is handed over to the second workflow.  
4. The second workflow:  
   - Fetches the scraping results,  
   - Cleans and transforms the data,  
   - Maps it to the correct format, and  
   - Saves it into Google Sheets.  
5. The final sheet contains all the useful business information in a structured format.  

---

## ✅ Example Output in Google Sheets

Here’s an example of what the final result looks like inside the Google Sheet after running both workflows:

| Complete Address | Doctor Name | Specialty | Clinic/Hospital | City | Pin Code | State | Contact Number | Ratings | Reviews | Reviews Count | Rank | Map Location |
|------------------|-------------|-----------|-----------------|------|----------|-------|----------------|---------|---------|---------------|------|--------------|
| 502, Deron Hills, S.No.6/1/1, Amar Apex Lane, Baner Road, Pune, Maharashtra 411069, India | DOC-AT-HOME | Health consultant | DOC-AT-HOME | Pune | 411069 | Maharashtra | 91 72198 73600 | 3.5 | """DOC-AT-HOME has been a game changer for me, their health consulting services are top notch and very convenient. Highly recommended!"" - Rohan Sharma<br>""I've been using DOC-AT-HOME for a while now and I must say, their team is very professional and knowledgeable. Great experience!"" - Priya Jain<br>""DOC-AT-HOME's health consulting services are very personalized and effective, I've seen significant improvement in my overall health. Thank you!"" - Karan Singh" | 152 | 3 | [Link](https://www.google.com/maps/search/?api=1&query=DOC-AT-HOME&query_place_id=ChIJU4kI89i-wjsRPxtEEvz5zMU) |
| OPD no.1, Unnamed Road, Ward No. 8, Aundh Gaon, Aundh, Pune, Maharashtra 411027, India | Dr. Satish Gupta Clinic \| General Physician in Pune, Aundh \| Diabetologist in Pune, Aundh | General practitioner, Allergist, Diabetologist, Doctor, Gastroenterologist, Infectious disease physician, Medical Center, Medical clinic, Surgeon | Dr. Satish Gupta Clinic \| General Physician in Pune, Aundh \| Diabetologist in Pune, Aundh | Pune | 411027 | Maharashtra | 91 78210 94354 | 4.9 | """Dr. Satish Gupta is an excellent doctor, very polite and explains everything clearly. Highly recommended for diabetes treatment!"" - Rohan S.<br>""I visited Dr. Gupta for stomach issues and he diagnosed the problem quickly. His clinic is well-maintained and staff is very cooperative."" - Priya J.<br>""Dr. Satish Gupta is a very experienced and knowledgeable doctor. He treated my allergic reaction effectively and I'm completely satisfied with his service."" - Amit K." | 152 | 12 | [Link](https://www.google.com/maps/search/?api=1&query=Dr.%20Satish%20Gupta%20Clinic%20%7C%20General%20Physician%20in%20Pune%2C%20Aundh%20%7C%20Diabetologist%20in%20Pune%2C%20Aundh&query_place_id=ChIJy0eLMiS_wjsRKJpXPNi5T8A) |

---

## ⚙️ Setup Instructions

1. Import both workflows into n8n using the *Import from File* option.  
2. Configure the credentials for:  
   - **Apify (or any scraping API you are using)**  
   - **Google Sheets** (via Google credentials in n8n)  
3. Start Workflow 1 and send a query.  
4. Workflow 2 will automatically execute after scraping and update your Google Sheet.  

---

## 📌 Notes

- Workflow 1 is **independent** and only handles scraping.  
- Workflow 2 is **dependent** and only runs when Workflow 1 provides data.  
- This setup is useful for **automating lead collection, market research, or local business listings**.  
- The Google Sheet becomes a central place to manage and analyze all collected data.  
