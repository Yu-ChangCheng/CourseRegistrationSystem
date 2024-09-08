def solution(queries):
    class CourseRegistrationSystem:
        def __init__(self):
            self.courses = {}  # key: courseId, value: (name, credits, gradingType)
            self.student_courses = {}  # key: studentId, value: set(courseIds)
            self.grades = {}  # key: (studentId, courseId), value: {component: grade}

        def create_course_ext(self, course_id, name, credits, grading_type="Standard"):
            if any(course_id == c_id or name == c_name for c_id, (c_name, _, _) in self.courses.items()):
                return "false"
            self.courses[course_id] = (name, int(credits), grading_type)
            return "true"

        def register_for_course(self, student_id, course_id):
            if course_id not in self.courses:
                return "false"
            if student_id in self.student_courses and course_id in self.student_courses[student_id]:
                return "false"
            self.student_courses.setdefault(student_id, set()).add(course_id)
            return "true"

        def set_component_grade(self, student_id, course_id, component, grade):
            if course_id not in self.courses or student_id not in self.student_courses or course_id not in \
                    self.student_courses[student_id]:
                return "invalid"
            key = (student_id, course_id)
            if key not in self.grades:
                self.grades[key] = {}
            if component in self.grades[key]:
                self.grades[key][component] = int(grade)
                return "updated"
            else:
                self.grades[key][component] = int(grade)
                return "set"

        def get_gpa(self, student_id):
            if student_id not in self.student_courses:
                return ""
            total_weighted_grades = 0
            total_credits = 0
            pass_count = 0
            fail_count = 0

            for course_id in self.student_courses[student_id]:
                key = (student_id, course_id)
                if key not in self.grades or len(self.grades[key]) != 3:
                    return ""
                total_grade = sum(self.grades[key].values())
                name, credits, grading_type = self.courses[course_id]

                if grading_type == "Standard":
                    total_weighted_grades += total_grade * credits
                    total_credits += credits
                elif grading_type == "Pass/Fail":
                    if total_grade >= 66:
                        pass_count += 1
                    else:
                        fail_count += 1

            if total_credits == 0:  # Handle edge case where there are no standard courses
                return ""
            gpa = total_weighted_grades // total_credits
            return f"{gpa}, {pass_count}, {fail_count}"

        def get_paired_students(self):
            paired_students = set()
            course_to_students = {}

            for student, courses in self.student_courses.items():
                for course in courses:
                    if self.courses[course][2] == "Standard":
                        if course not in course_to_students:
                            course_to_students[course] = set()
                        course_to_students[course].add(student)

            for students in course_to_students.values():
                student_list = sorted(students)
                for i in range(len(student_list)):
                    for j in range(i + 1, len(student_list)):
                        pair = (student_list[i], student_list[j])
                        paired_students.add(pair)

            if not paired_students:
                return ""
            formatted_pairs = "[[" + "], [".join(f"{a}, {b}" for a, b in sorted(paired_students)) + "]]"
            return formatted_pairs

    def run_queries(queries):
        system = CourseRegistrationSystem()
        results = []
        for query in queries:
            if query[0] == "CREATE_COURSE_EXT":
                result = system.create_course_ext(query[1], query[2], query[3], query[4])
            elif query[0] == "REGISTER_FOR_COURSE":
                result = system.register_for_course(query[1], query[2])
            elif query[0] == "SET_COMPONENT_GRADE":
                result = system.set_component_grade(query[1], query[2], query[3], query[4])
            elif query[0] == "GET_GPA":
                result = system.get_gpa(query[1])
            elif query[0] == "GET_PAIRED_STUDENTS":
                result = system.get_paired_students()
            results.append(result)
        return results

    return run_queries(queries)

queries = [
        ["CREATE_COURSE_EXT", "CSE220", "Data Structures", "3", "Standard"],
        ["CREATE_COURSE_EXT", "CSE300", "Operating Systems", "4", "Standard"],
        ["CREATE_COURSE_EXT", "CSE330", "Computer Architecture", "3", "Pass/Fail"],
        ["REGISTER_FOR_COURSE", "st001", "CSE220"],
        ["REGISTER_FOR_COURSE", "st001", "CSE330"],
        ["REGISTER_FOR_COURSE", "st002", "CSE330"],
        ["GET_PAIRED_STUDENTS"],
        ["REGISTER_FOR_COURSE", "st002", "CSE220"],
        ["GET_PAIRED_STUDENTS"],
        ["SET_COMPONENT_GRADE", "st002", "CSE220", "homeworks", "20"],
        ["SET_COMPONENT_GRADE", "st002", "CSE220", "homeworks", "25"],
        ["SET_COMPONENT_GRADE", "st002", "CSE300", "homeworks", "25"],
        ["SET_COMPONENT_GRADE", "st002", "BIO777", "homeworks", "25"],
        ["SET_COMPONENT_GRADE", "st002", "CSE220", "midterm", "23"],
        ["SET_COMPONENT_GRADE", "st002", "CSE220", "final", "33"],
        ["GET_GPA", "st002"],
        ["SET_COMPONENT_GRADE", "st002", "CSE330", "homeworks", "25"],
        ["SET_COMPONENT_GRADE", "st002", "CSE330", "midterm", "25"],
        ["SET_COMPONENT_GRADE", "st002", "CSE330", "final", "15"],
        ["GET_GPA", "st002"],
        ["REGISTER_FOR_COURSE", "st002", "CSE300"],
        ["GET_GPA", "st002"],
        ["SET_COMPONENT_GRADE", "st002", "CSE300", "homeworks", "20"],
        ["SET_COMPONENT_GRADE", "st002", "CSE300", "midterm", "20"],
        ["SET_COMPONENT_GRADE", "st002", "CSE300", "final", "35"],
        ["GET_GPA", "st002"]
    ]
print(["true", "true", "true", "true", "true", "true", "", "true", "[[st001, st002]]", "set", "updated", "invalid",
       "invalid", "set", "set", "", "set", "set", "set", "81, 0, 1", "true", "", "set", "set", "set", "77, 0, 1"])
print(solution(queries))

import unittest

class TestCourseRegistrationSystem(unittest.TestCase):

    def test_solution(self):
        queries = [
            ["CREATE_COURSE_EXT", "CSE220", "Data Structures", "3", "Standard"],
            ["CREATE_COURSE_EXT", "CSE300", "Operating Systems", "4", "Standard"],
            ["CREATE_COURSE_EXT", "CSE330", "Computer Architecture", "3", "Pass/Fail"],
            ["REGISTER_FOR_COURSE", "st001", "CSE220"],
            ["REGISTER_FOR_COURSE", "st001", "CSE330"],
            ["REGISTER_FOR_COURSE", "st002", "CSE330"],
            ["GET_PAIRED_STUDENTS"],
            ["REGISTER_FOR_COURSE", "st002", "CSE220"],
            ["GET_PAIRED_STUDENTS"],
            ["SET_COMPONENT_GRADE", "st002", "CSE220", "homeworks", "20"],
            ["SET_COMPONENT_GRADE", "st002", "CSE220", "homeworks", "25"],
            ["SET_COMPONENT_GRADE", "st002", "CSE300", "homeworks", "25"],
            ["SET_COMPONENT_GRADE", "st002", "BIO777", "homeworks", "25"],
            ["SET_COMPONENT_GRADE", "st002", "CSE220", "midterm", "23"],
            ["SET_COMPONENT_GRADE", "st002", "CSE220", "final", "33"],
            ["GET_GPA", "st002"],
            ["SET_COMPONENT_GRADE", "st002", "CSE330", "homeworks", "25"],
            ["SET_COMPONENT_GRADE", "st002", "CSE330", "midterm", "25"],
            ["SET_COMPONENT_GRADE", "st002", "CSE330", "final", "15"],
            ["GET_GPA", "st002"],
            ["REGISTER_FOR_COURSE", "st002", "CSE300"],
            ["GET_GPA", "st002"],
            ["SET_COMPONENT_GRADE", "st002", "CSE300", "homeworks", "20"],
            ["SET_COMPONENT_GRADE", "st002", "CSE300", "midterm", "20"],
            ["SET_COMPONENT_GRADE", "st002", "CSE300", "final", "35"],
            ["GET_GPA", "st002"]
        ]



        expected_results = ["true", "true", "true", "true", "true", "true", "", "true", "[[st001, st002]]", "set", "updated", "invalid", "invalid", "set", "set", "", "set", "set", "set", "81, 0, 1", "true", "", "set", "set", "set", "77, 0, 1"]
        actual_results = solution(queries)
        self.assertEqual(actual_results, expected_results, "Failed to match expected results.")

if __name__ == '__main__':
    unittest.main()
