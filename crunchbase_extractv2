import streamlit as st
import re
import pandas as pd
import io

st.title("🛠️ Company Text Formatter")

st.write("""
Paste your messy company text below (each 'Logo' line starts a new company).
The app will clean it, preview it, and let you download it as a CSV!
""")

# Text input
raw_text = st.text_area("Paste your text here:", height=300)

# Buttons
col1, col2 = st.columns([1, 1])

with col1:
    format_clicked = st.button("Format Text")
with col2:
    clear_clicked = st.button("Clear Text")

# Clear text action
if clear_clicked:
    st.experimental_rerun()

# Process text
if format_clicked:
    if not raw_text.strip():
        st.warning("⚠️ Please paste some content first.")
    else:
        # Step 1: Preprocess lines
        lines = [line.strip() for line in raw_text.split("\n") if line.strip()]

        # Step 2: Group companies by 'Logo'
        companies = []
        current_company = []

        for line in lines:
            if "Logo" in line and current_company:
                companies.append(current_company)
                current_company = []
            current_company.append(line)
        if current_company:
            companies.append(current_company)

        # Step 3: Parse companies
        parsed_companies = []

        for company in companies:
            parsed = {}

            # Company Name (remove 'Logo' from line)
            company_name_line = company[0] if len(company) > 0 else '—'
            clean_name = company_name_line.replace("Logo", "").strip()
            parsed['Company Name'] = clean_name

            for info in company[1:]:
                if re.search(r'@\w+\.', info):
                    parsed['Email'] = info
                elif re.search(r'(http|www\.)', info) or re.search(r'\.(com|org|net|ai|io|bio|health)', info):
                    parsed['Website'] = info
                elif re.search(r'[$€]|CA\$', info):
                    parsed['Funding Amount'] = info
                elif re.search(r'\b\d{1,2}[/-]\d{1,2}[/-]\d{2,4}\b|\b\w+\s+\d{1,2},\s*\d{4}\b', info):
                    parsed['Date'] = info
                else:
                    # Assign to Extra Fields
                    field_name = f"Extra Field {len([k for k in parsed.keys() if 'Extra Field' in k]) + 1}"
                    parsed[field_name] = info

            parsed_companies.append(parsed)

        # Step 4: Create DataFrame (NO reordering)
        df = pd.DataFrame(parsed_companies)

        # Step 5: Show Preview
        st.success(f"✅ Parsed {len(df)} companies!")
        st.dataframe(df)

        # Step 6: Download CSV
        csv_buffer = io.StringIO()
        df.to_csv(csv_buffer, index=False)
        csv_data = csv_buffer.getvalue()

        st.download_button(
            label="📄 Download CSV",
            data=csv_data,
            file_name="formatted_companies.csv",
            mime="text/csv"
        )
