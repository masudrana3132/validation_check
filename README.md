# validation_check

import re
import streamlit as st

st.set_page_config(
    page_title="Text File Validator", page_icon="📋", layout="centered"
)

st.title("📋 Text File Validation System")
st.write(
    "Upload your comma-separated text file below to check against all 47"
    " validation rules."
)

# ফাইল আপলোড করার ফর্ম
uploaded_file = st.file_uploader("Attach your text file (.txt)", type=["txt"])

if uploaded_file is not None:
  if st.button("Validate File"):
    # utf-8-sig ব্যবহার করা হয়েছে যাতে ফাইলের অদৃশ্য BOM (Byte Order Mark) সমস্যা না করে
    lines = uploaded_file.getvalue().decode("utf-8-sig").splitlines()

    total_errors = 0
    error_details = []

    # রেগুলার এক্সপ্রেশন প্যাটার্নসমূহ
    col1_pattern = re.compile(r"^EM.{8}$")  # Col 1: EM দিয়ে শুরু এবং মোট ১০ ক্যারেক্টার
    col2_pattern = re.compile(
        r"^[A-Za-z\s\.\-]+$"
    )  # Col 2: অক্ষর, স্পেস, ডট (.) এবং ড্যাশ (-) অনুমোদিত, কোনো সংখ্যা নেই
    col3_pattern = re.compile(r"^\d{4}$")  # Col 3: ঠিক ৪ ডিজিটের সংখ্যা
    col5_pattern = re.compile(r"^.{7}$")  # Col 5: ঠিক ৭ ক্যারেক্টার ডেটা
    col6_pattern = re.compile(
        r"^(0[0-9]|[1-9][0-9])$"
    )  # Col 6: ২ ডিজিট, একক সংখ্যা হলে প্রথমে ০ থাকবে (যেমন 05 বা 12)
    col7_pattern = re.compile(
        r"^[1-9]\d*$|^0$"
    )  # Col 7: নিউমেরিক ডেটা, সামনে কোনো অপ্রয়োজনীয় শূন্য (যেমন '01') থাকবে না
    decimal_pattern = re.compile(
        r"^\d+\.\d{2}$"
    )  # Col 8-46: দশমিকের পর ঠিক দুই ঘর থাকবে (যেমন 999.99)
    col47_pattern = re.compile(
        r"^\d{4}$"
    )  # Col 47: যেকোনো ৪ ডিজিটের বছর (যেমন 2021, 2022, 2026 ইত্যাদি)

    for line_num, line in enumerate(lines, 1):
      # প্রতিটি লাইন কমা (,) দিয়ে স্প্লিট করা
      cols = line.split(",")

      # শর্ত ১: মোট কলাম সংখ্যা ৪৭ হতে হবে
      if len(cols) != 47:
        total_errors += 1
        error_details.append(
            f"Line {line_num}: Expected 47 columns, but found {len(cols)} columns."
        )
        continue

      # ভ্যালিডেশনের সুবিধার্থে প্রতিটি কলাম ট্রিম (Whitespace রিমুভ) করে নেওয়া
      cols = [c.strip() for c in cols]

      line_errors = []

      # কলাম ১: Must be 10 characters starting with 'EM'
      if not col1_pattern.match(cols[0]):
        line_errors.append(
            f"Col 1 ('{cols[0]}'): Must be exactly 10 characters starting with"
            " 'EM'."
        )

      # কলাম ২: Name field, letters, dots, dashes, and spaces allowed, no numbers
      if not col2_pattern.match(cols[1]):
        line_errors.append(
            f"Col 2 ('{cols[1]}'): Name must contain only characters, dots,"
            " dashes, and spaces."
        )

      # কলাম ৩: Must be 4 digit numeric data
      if not col3_pattern.match(cols[2]):
        line_errors.append(
            f"Col 3 ('{cols[2]}'): Must be a 4-digit numeric value."
        )

      # কলাম ৪: Address and District field with various characters
      if not cols[3]:
        line_errors.append("Col 4: Address/District cannot be empty.")

      # কলাম ৫: Must be 7 digit character data
      if not col5_pattern.match(cols[4]):
        line_errors.append(f"Col 5 ('{cols[4]}'): Must be exactly 7 characters.")

      # কলাম ৬: Must be 2 digits character data, if single digit start with Zero ('0')
      if not col6_pattern.match(cols[5]):
        line_errors.append(
            f"Col 6 ('{cols[5]}'): Must be 2 digits (single digit must start"
            " with '0')."
        )

      # কলাম ৭: Must be numeric data, no leading zero allowed (e.g., '01', '02' invalid)
      if not col7_pattern.match(cols[6]):
        line_errors.append(
            f"Col 7 ('{cols[6]}'): Must be numeric data without leading zeros"
            " (e.g., '1', '12' allowed, but not '01')."
        )

      # কলাম ৮ থেকে ৪৬ (ইনডেক্স ৭ থেকে ৪৫): Decimal data type with 2 digits after decimal
      for i in range(7, 46):
        val = cols[i]
        if not decimal_pattern.match(val):
          line_errors.append(
              f"Col {i+1} ('{val}'): Must be a decimal with exactly 2 digits"
              " after the dot."
          )

      # কলাম ৪৭ (ইনডেক্স ৪৬): Must be any 4-digit year data like 2021, 2022, 2026 etc.
      if not col47_pattern.match(cols[46]):
        line_errors.append(
            f"Col 47 ('{cols[46]}'): Must be a valid 4-digit year (e.g., 2021,"
            " 2026)."
        )

      # যদি এই লাইনে কোনো এরর পাওয়া যায়
      if line_errors:
        total_errors += len(line_errors)
        for err in line_errors:
          error_details.append(f"Line {line_num} -> {err}")

    # রেজাল্ট প্রদর্শন
    st.markdown("---")
    if total_errors == 0:
      st.success(
          "🎉 Validation Successful! No errors found in the uploaded text"
          " file."
      )
    else:
      st.error(
          f"❌ Validation Failed! Total Errors Found: **{total_errors}**"
      )

      with st.expander("🔍 View Detailed Error Report", expanded=True):
        for err in error_details:
          st.write(f"- {err}")
