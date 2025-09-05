import json
import os

class Student:
    def __init__(self, name, age, gender, height_cm, weight_kg, activity_level, goal):
        self.name = name
        self.age = age
        self.gender = gender
        self.height_cm = height_cm
        self.weight_kg = weight_kg
        self.activity_level = activity_level
        self.goal = goal

    def calculate_bmi(self):
        height_m = self.height_cm / 100
        bmi = self.weight_kg / (height_m ** 2)
        return round(bmi, 2)

    def calculate_bmr(self):
        if self.gender.lower() == 'male':
            return 10 * self.weight_kg + 6.25 * self.height_cm - 5 * self.age + 5
        else:
            return 10 * self.weight_kg + 6.25 * self.height_cm - 5 * self.age - 161

    def get_activity_factor(self):
        levels = {
            'sedentary': 1.2,
            'light': 1.375,
            'moderate': 1.55,
            'active': 1.725,
            'very active': 1.9
        }
        return levels.get(self.activity_level.lower(), 1.2)

    def calculate_tdee(self):
        return round(self.calculate_bmr() * self.get_activity_factor(), 2)

    def recommend_nutrients(self):
        tdee = self.calculate_tdee()

        if self.goal.lower() == "gain":
            tdee += 300
        elif self.goal.lower() == "lose":
            tdee -= 300

        protein = round((0.3 * tdee) / 4, 1)
        carbs = round((0.5 * tdee) / 4, 1)
        fats = round((0.2 * tdee) / 9, 1)

        return {
            "Calories": round(tdee, 2),
            "Protein (g)": protein,
            "Carbohydrates (g)": carbs,
            "Fats (g)": fats,
            "Water (L/day)": round(self.weight_kg * 0.035, 2),
            "Vitamins": "Take multivitamins or eat fruits/veggies daily"
        }

    def meal_time_recommendation(self):
        return {
            "Morning": "High protein + complex carbs (eggs, oats, banana)",
            "Lunch": "Balanced meal (rice, vegetables, protein source)",
            "Evening": "Light meal (soup, salad, yogurt)",
            "Snacks": "Fruits, nuts, protein bars"
        }

    def exercise_recommendation(self):
        if self.goal.lower() == "gain":
            return "Strength training 4x/week + protein-rich diet"
        elif self.goal.lower() == "lose":
            return "Cardio + calorie deficit + light strength training"
        else:
            return "3x/week full-body workout + balanced diet"


# ==============================
# Data Handling + Main Program
# ==============================

DATA_FILE = "data/records.json"

def save_student_record(student, nutrients):
    if not os.path.exists("data"):
        os.makedirs("data")
    if not os.path.exists(DATA_FILE):
        with open(DATA_FILE, "w") as f:
            json.dump([], f)

    with open(DATA_FILE, "r") as f:
        records = json.load(f)

    student_data = {
        "name": student.name,
        "age": student.age,
        "gender": student.gender,
        "height_cm": student.height_cm,
        "weight_kg": student.weight_kg,
        "bmi": student.calculate_bmi(),
        "tdee": student.calculate_tdee(),
        "nutrients": nutrients,
        "exercise": student.exercise_recommendation()
    }

    records.append(student_data)

    with open(DATA_FILE, "w") as f:
        json.dump(records, f, indent=4)

def show_records():
    if not os.path.exists(DATA_FILE):
        print("No records found.")
        return
    with open(DATA_FILE, "r") as f:
        records = json.load(f)
        for i, rec in enumerate(records, start=1):
            print(f"\n--- Student {i} ---")
            for k, v in rec.items():
                print(f"{k}: {v}")

def main():
    print("📘 Student Diet Management System")
    while True:
        print("\nMenu:")
        print("1. Add New Student")
        print("2. Show All Records")
        print("3. Exit")
        choice = input("Enter choice: ")

        if choice == "1":
            name = input("Enter name: ")
            age = int(input("Enter age: "))
            gender = input("Enter gender (Male/Female): ")
            height_cm = float(input("Enter height (cm): "))
            weight_kg = float(input("Enter weight (kg): "))
            activity_level = input("Activity level (sedentary, light, moderate, active, very active): ")
            goal = input("Goal (Gain / Lose / Maintain): ")

            student = Student(name, age, gender, height_cm, weight_kg, activity_level, goal)
            nutrients = student.recommend_nutrients()

            print("\n📊 Nutrient Recommendations:")
            for k, v in nutrients.items():
                print(f"{k}: {v}")

            print("\n🍽 Meal Time Suggestions:")
            for k, v in student.meal_time_recommendation().items():
                print(f"{k}: {v}")

            print("\n🏋 Exercise Recommendation:")
            print(student.exercise_recommendation())

            save_student_record(student, nutrients)
            print("\n✅ Student record saved.")

        elif choice == "2":
            show_records()
        elif choice == "3":
            print("Goodbye!")
            break
        else:
            print("Invalid choice. Try again.")

if __name__ == "__main__":
    main()
