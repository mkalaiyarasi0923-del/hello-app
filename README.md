# hello-app


import streamlit as st
import pandas as pd
import random

def generate_timetable(periods_per_subject, four_consec_subjects, teacher_availability, lab_session_choice):
    days = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday']
    periods_per_day = 9
    times = [
        '9:10-10:00', '10:00-10:50', '11:00-11:50', '11:50-12:40',
        '12:40-1:40',
        '1:40-2:30', '2:30-3:15', '3:15-4:00', '4:00-4:45'
    ]
    break_idx = 4
    timetable = {day: [''] * periods_per_day for day in days}
    normal_subject_list = []
    for sub, count in periods_per_subject.items():
        normal_subject_list.extend([sub] * count)
    four_assignment = {}
    if four_consec_subjects:
        assigned_days = random.sample(days, k=len(four_consec_subjects))
        four_assignment = dict(zip(assigned_days, four_consec_subjects))
    for day in days:
        day_schedule = [''] * periods_per_day
        day_schedule[break_idx] = "Break"
        if day in four_assignment:
            lab = four_assignment[day]
            session = lab_session_choice[lab][day]
            if session == "Forenoon":
                lab_slots = [0, 1, 2, 3]
            else:
                lab_slots = [5, 6, 7, 8]
            if all(idx in teacher_availability[lab].get(day, []) for idx in lab_slots):
                for idx in lab_slots:
                    day_schedule[idx] = lab
                    if lab in normal_subject_list:
                        normal_subject_list.remove(lab)
        for idx in range(periods_per_day):
            if idx == break_idx or day_schedule[idx] != "":
                continue
            available_subjects = [
                sub for sub in normal_subject_list
                if day in teacher_availability.get(sub, {})
                and idx in teacher_availability[sub][day]
            ]
            if available_subjects:
                chosen = random.choice(available_subjects)
                day_schedule[idx] = chosen
                normal_subject_list.remove(chosen)
            else:
                day_schedule[idx] = "Library"
        timetable[day] = day_schedule
    return pd.DataFrame(timetable, index=times)

def main():
    st.title("Intelligent Timetable Generator")
    st.subheader("Created by CODE BLOODED")
    days = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday']
    st.header("Step 1: Subjects & Teacher Availability")
    num_subjects = st.number_input("Number of normal subjects", min_value=1, step=1)
    periods_per_subject = {}
    teacher_availability = {}
    for i in range(num_subjects):
        st.subheader(f"Subject {i + 1}")
        sub = st.text_input("Subject name", key=f"sub_{i}")
        if sub.strip() == "":
            continue
        periods_per_subject[sub] = st.number_input("Periods per week", min_value=1, step=1, key=f"periods_{i}")
        teacher_availability[sub] = {}
        for day in days:
            available = st.multiselect(
                f"{sub} availability on {day}",
                [0, 1, 2, 3, 5, 6, 7, 8],
                format_func=lambda x: f"Period {x + 1}",
                key=f"{sub}_{day}"
            )
            if available:
                teacher_availability[sub][day] = available
    st.header("Step 2: 4-Consecutive (Lab) Subjects")
    num_labs = st.number_input("Number of lab subjects", min_value=0, step=1)
    four_consec_subjects = []
    lab_session_choice = {}
    for i in range(num_labs):
        lab = st.text_input(f"Lab subject {i + 1}", key=f"lab_{i}")
        if lab.strip() == "":
            continue
        four_consec_subjects.append(lab)
        teacher_availability[lab] = {}
        lab_session_choice[lab] = {}
        st.write(f"Session choice for {lab}")
        for day in days:
            session = st.radio(
                f"{lab} on {day}",
                ["Forenoon", "Afternoon"],
                horizontal=True,
                key=f"{lab}_{day}_session"
            )
            lab_session_choice[lab][day] = session
            if session == "Forenoon":
                teacher_availability[lab][day] = [0, 1, 2, 3]
            else:
                teacher_availability[lab][day] = [5, 6, 7, 8]
    if st.button("Generate Timetable"):
        df = generate_timetable(
            periods_per_subject,
            four_consec_subjects,
            teacher_availability,
            lab_session_choice
        )
        st.header("Generated Timetable")
        st.dataframe(df)
        csv = df.to_csv().encode("utf-8")
        st.download_button("Download Timetable as CSV", csv, "timetable.csv", "text/csv")
        st.write("### Notes")
        st.write("- Download as CSV file.")
        st.write("- Subject availability is shown as Period 1, Period 2, …")
        st.write("- Labs occupy 4 continuous periods without intersecting the periods by break.")
        st.write("- Timetable strictly respects teacher availability.")
        st.write("- empty slots are filled with Library.")

if __name__ == "__main__":
    main()
