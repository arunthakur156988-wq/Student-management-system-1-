* ============================================================
 *  PROJECT : Student Result Management System
 *  LANGUAGE: C++ (OOPs)
 *  CONCEPTS: Class, Object, Encapsulation, Abstraction,
 *            Inheritance, Polymorphism, Constructor/Destructor,
 *            Operator Overloading, Friend Function,
 *            Static Members, Exception Handling, File Handling,
 *            STL (vector), Templates
 * ============================================================
 */
#include <iostream>
#include <fstream>
#include <vector>
#include <string>
#include <iomanip>
#include <limits>
#include <algorithm>
#include <sstream>
using namespace std;
// ------------------------------------------------------------
// Custom Exception class (Exception Handling)
// ------------------------------------------------------------
class ResultException {
    string message;
public:
    ResultException(const string &msg) : message(msg) {}
    string what() const { return message; }
};
// ------------------------------------------------------------
// Abstract base class: Person (Abstraction + Inheritance)
// ------------------------------------------------------------
class Person {
protected:
    string name;
    int age;
public:
    Person() : name(""), age(0) {}
    Person(const string &n, int a) : name(n), age(a) {}
    virtual ~Person() {}
    // Pure virtual function -> makes the class abstract
    virtual void display() const = 0;
    string getName() const { return name; }
    int getAge() const { return age; }
};
// ------------------------------------------------------------
// Subject class (Composition: Student HAS-A many Subjects)
// ------------------------------------------------------------
class Subject {
    string subjectName;
    float marks;
    float maxMarks;
public:
    Subject() : subjectName(""), marks(0), maxMarks(100) {}
    Subject(const string &n, float m, float mx = 100)
        : subjectName(n), marks(m), maxMarks(mx) {
        if (m < 0 || m > mx)
            throw ResultException("Invalid marks for subject " + n);
    }
    string getName() const { return subjectName; }
    float getMarks() const { return marks; }
    float getMaxMarks() const { return maxMarks; }
    bool isPass() const { return marks >= 0.35f * maxMarks; }
};
// ------------------------------------------------------------
// Student class (derived from Person)
// ------------------------------------------------------------
class Student : public Person {
    int rollNo;
    string course;
    vector<Subject> subjects;
    static int totalStudents;      // static member
public:
    Student() : Person(), rollNo(0), course("") { totalStudents++; }
    Student(int r, const string &n, int a, const string &c)
        : Person(n, a), rollNo(r), course(c) { totalStudents++; }
    Student(const Student &s) : Person(s), rollNo(s.rollNo),
        course(s.course), subjects(s.subjects) { totalStudents++; }
    ~Student() { totalStudents--; }
    // ----- getters -----
    int getRollNo() const { return rollNo; }
    string getCourse() const { return course; }
    const vector<Subject>& getSubjects() const { return subjects; }
    static int getTotalStudents() { return totalStudents; }
    // ----- setters -----
    void setName(const string &n) { name = n; }
    void setAge(int a) { age = a; }
    void setCourse(const string &c) { course = c; }
    void addSubject(const Subject &s) { subjects.push_back(s); }
    void clearSubjects() { subjects.clear(); }
    float totalMarks() const {
        float t = 0;
        for (size_t i = 0; i < subjects.size(); i++) t += subjects[i].getMarks();
        return t;
    }
    float maxTotal() const {
        float t = 0;
        for (size_t i = 0; i < subjects.size(); i++) t += subjects[i].getMaxMarks();
        return t;
    }
    float percentage() const {
        float mt = maxTotal();
        return mt == 0 ? 0 : (totalMarks() / mt) * 100.0f;
    }
    bool isPass() const {
        if (subjects.empty()) return false;
        for (size_t i = 0; i < subjects.size(); i++)
            if (!subjects[i].isPass()) return false;
        return true;
    }
    char grade() const {
        if (!isPass()) return 'F';
        float p = percentage();
        if (p >= 90) return 'A';
        if (p >= 75) return 'B';
        if (p >= 60) return 'C';
        if (p >= 45) return 'D';
        return 'E';
    }
    string division() const {
        if (!isPass()) return "Fail";
        float p = percentage();
        if (p >= 75) return "Distinction";
        if (p >= 60) return "First Division";
        if (p >= 45) return "Second Division";
        return "Third Division";
    }
    // Overriding the pure virtual function (Polymorphism)
    void display() const {
        cout << "\n==================== STUDENT RESULT ====================\n";
        cout << "Roll No : " << rollNo << "\n";
        cout << "Name    : " << name << "\n";
        cout << "Age     : " << age << "\n";
        cout << "Course  : " << course << "\n";
        cout << "--------------------------------------------------------\n";
        cout << left << setw(25) << "Subject" << setw(12) << "Marks"
             << setw(12) << "Max" << "Status\n";
        cout << "--------------------------------------------------------\n";
        for (size_t i = 0; i < subjects.size(); i++) {
            cout << left << setw(25) << subjects[i].getName()
                 << setw(12) << subjects[i].getMarks()
                 << setw(12) << subjects[i].getMaxMarks()
                 << (subjects[i].isPass() ? "Pass" : "Fail") << "\n";
        }
        cout << "--------------------------------------------------------\n";
        cout << fixed << setprecision(2);
        cout << "Total      : " << totalMarks() << " / " << maxTotal() << "\n";
        cout << "Percentage : " << percentage() << " %\n";
        cout << "Grade      : " << grade() << "\n";
        cout << "Result     : " << division() << "\n";
        cout << "========================================================\n";
    }
    // Operator overloading: compare students by percentage
    bool operator>(const Student &o) const { return percentage() > o.percentage(); }
    bool operator<(const Student &o) const { return percentage() < o.percentage(); }
    // Friend function for file output
    friend ostream& operator<<(ostream &os, const Student &s);
    friend istream& operator>>(istream &is, Student &s);
};
int Student::totalStudents = 0;
// ----- Serialisation: one student per line, fields separated by '|' -----
ostream& operator<<(ostream &os, const Student &s) {
    os << s.rollNo << "|" << s.name << "|" << s.age << "|" << s.course << "|"
       << s.subjects.size();
    for (size_t i = 0; i < s.subjects.size(); i++)
        os << "|" << s.subjects[i].getName() << "|" << s.subjects[i].getMarks()
           << "|" << s.subjects[i].getMaxMarks();
    os << "\n";
    return os;
}
istream& operator>>(istream &is, Student &s) {
    string line;
    if (!getline(is, line) || line.empty()) { is.setstate(ios::failbit); return is; }
    vector<string> parts;
    string tok;
    stringstream ss(line);
    while (getline(ss, tok, '|')) parts.push_back(tok);
    if (parts.size() < 5) { is.setstate(ios::failbit); return is; }
    s.rollNo = atoi(parts[0].c_str());
    s.name = parts[1];
    s.age = atoi(parts[2].c_str());
    s.course = parts[3];
    int n = atoi(parts[4].c_str());
    s.subjects.clear();
    for (int i = 0; i < n; i++) {
        size_t base = 5 + i * 3;
        if (base + 2 >= parts.size()) break;
        s.subjects.push_back(Subject(parts[base],
                             (float)atof(parts[base + 1].c_str()),
                             (float)atof(parts[base + 2].c_str())));
    }
    return is;
}
// ------------------------------------------------------------
// Template function (Generic Programming)
// ------------------------------------------------------------
template <typename T>
T readValue(const string &prompt) {
    T value;
    while (true) {
        cout << prompt;
        if (cin >> value) {
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
            return value;
        }
        cin.clear();
        cin.ignore(numeric_limits<streamsize>::max(), '\n');
        cout << "Invalid input! Please try again.\n";
    }
}
string readLine(const string &prompt) {
    string s;
    cout << prompt;
    getline(cin, s);
    return s;
}
// ------------------------------------------------------------
// ResultManager: handles all operations (CRUD + File Handling)
// ------------------------------------------------------------
class ResultManager {
    vector<Student> students;
    string fileName;
    int findIndex(int roll) const {
        for (size_t i = 0; i < students.size(); i++)
            if (students[i].getRollNo() == roll) return (int)i;
        return -1;
    }
public:
    ResultManager(const string &f = "students.dat") : fileName(f) { load(); }
    ~ResultManager() { save(); }
    void load() {
        ifstream in(fileName.c_str());
        if (!in) return;
        Student s;
        while (in >> s) students.push_back(s);
        in.close();
    }
    void save() const {
        ofstream out(fileName.c_str());
        for (size_t i = 0; i < students.size(); i++) out << students[i];
        out.close();
    }
    void addStudent() {
        int roll = readValue<int>("Enter Roll No: ");
        if (findIndex(roll) != -1)
            throw ResultException("Roll number already exists!");
        string name = readLine("Enter Name: ");
        int age = readValue<int>("Enter Age: ");
        string course = readLine("Enter Course: ");
        Student s(roll, name, age, course);
        int n = readValue<int>("Number of subjects: ");
        for (int i = 0; i < n; i++) {
            cout << "--- Subject " << (i + 1) << " ---\n";
            string sn = readLine("Subject name: ");
            float m = readValue<float>("Marks obtained: ");
            float mx = readValue<float>("Maximum marks: ");
            s.addSubject(Subject(sn, m, mx));
        }
        students.push_back(s);
        save();
        cout << "\nStudent added successfully!\n";
    }
    void viewAll() const {
        if (students.empty()) { cout << "\nNo records found.\n"; return; }
        cout << "\n" << left << setw(8) << "Roll" << setw(22) << "Name"
             << setw(14) << "Course" << setw(10) << "Total"
             << setw(8) << "%" << setw(7) << "Grade" << "Result\n";
        cout << string(78, '-') << "\n";
        cout << fixed << setprecision(2);
        for (size_t i = 0; i < students.size(); i++) {
            const Student &s = students[i];
            cout << left << setw(8) << s.getRollNo() << setw(22) << s.getName()
                 << setw(14) << s.getCourse() << setw(10) << s.totalMarks()
                 << setw(8) << s.percentage() << setw(7) << s.grade()
                 << s.division() << "\n";
        }
        cout << "\nTotal students: " << Student::getTotalStudents() << "\n";
    }
    void searchStudent() const {
        int roll = readValue<int>("Enter Roll No to search: ");
        int idx = findIndex(roll);
        if (idx == -1) throw ResultException("Student not found!");
        const Person *p = &students[idx];   // base pointer -> runtime polymorphism
        p->display();
    }
    void updateStudent() {
        int roll = readValue<int>("Enter Roll No to update: ");
        int idx = findIndex(roll);
        if (idx == -1) throw ResultException("Student not found!");
        Student &s = students[idx];
        s.setName(readLine("New Name: "));
        s.setAge(readValue<int>("New Age: "));
        s.setCourse(readLine("New Course: "));
        s.clearSubjects();
        int n = readValue<int>("Number of subjects: ");
        for (int i = 0; i < n; i++) {
            string sn = readLine("Subject name: ");
            float m = readValue<float>("Marks obtained: ");
            float mx = readValue<float>("Maximum marks: ");
            s.addSubject(Subject(sn, m, mx));
        }
        save();
        cout << "\nRecord updated successfully!\n";
    }
    void deleteStudent() {
        int roll = readValue<int>("Enter Roll No to delete: ");
        int idx = findIndex(roll);
        if (idx == -1) throw ResultException("Student not found!");
        students.erase(students.begin() + idx);
        save();
        cout << "\nRecord deleted successfully!\n";
    }
    void showTopper() const {
        if (students.empty()) { cout << "\nNo records found.\n"; return; }
        const Student *top = &students[0];
        for (size_t i = 1; i < students.size(); i++)
            if (students[i] > *top) top = &students[i];
        cout << "\n*** TOPPER ***";
        top->display();
    }
    void rankList() const {
        if (students.empty()) { cout << "\nNo records found.\n"; return; }
        vector<Student> sorted = students;
        sort(sorted.begin(), sorted.end(),
             [](const Student &a, const Student &b) { return a > b; });
        cout << "\n" << left << setw(6) << "Rank" << setw(8) << "Roll"
             << setw(22) << "Name" << "Percentage\n";
        cout << string(50, '-') << "\n";
        cout << fixed << setprecision(2);
        for (size_t i = 0; i < sorted.size(); i++)
            cout << left << setw(6) << (i + 1) << setw(8) << sorted[i].getRollNo()
                 << setw(22) << sorted[i].getName()
                 << sorted[i].percentage() << " %\n";
    }
    void statistics() const {
        if (students.empty()) { cout << "\nNo records found.\n"; return; }
        int pass = 0, fail = 0;
        float sum = 0, hi = 0, lo = 100;
        for (size_t i = 0; i < students.size(); i++) {
            float p = students[i].percentage();
            sum += p;
            if (p > hi) hi = p;
            if (p < lo) lo = p;
            if (students[i].isPass()) pass++; else fail++;
        }
        cout << fixed << setprecision(2);
        cout << "\n------ CLASS STATISTICS ------\n";
        cout << "Total Students : " << students.size() << "\n";
        cout << "Passed         : " << pass << "\n";
        cout << "Failed         : " << fail << "\n";
        cout << "Pass %         : " << (pass * 100.0 / students.size()) << " %\n";
        cout << "Class Average  : " << (sum / students.size()) << " %\n";
        cout << "Highest        : " << hi << " %\n";
        cout << "Lowest         : " << lo << " %\n";
    }
};
// ------------------------------------------------------------
// Menu + main()
// ------------------------------------------------------------
void showMenu() {
    cout << "\n=========================================\n";
    cout << "   STUDENT RESULT MANAGEMENT SYSTEM\n";
    cout << "=========================================\n";
    cout << " 1. Add Student Record\n";
    cout << " 2. View All Records\n";
    cout << " 3. Search Student (Result Card)\n";
    cout << " 4. Update Student Record\n";
    cout << " 5. Delete Student Record\n";
    cout << " 6. Show Topper\n";
    cout << " 7. Rank List\n";
    cout << " 8. Class Statistics\n";
    cout << " 0. Exit\n";
    cout << "=========================================\n";
}
int main() {
    ResultManager manager;
    int choice;
    do {
        showMenu();
        choice = readValue<int>("Enter your choice: ");
        try {
            switch (choice) {
                case 1: manager.addStudent();    break;
                case 2: manager.viewAll();       break;
                case 3: manager.searchStudent(); break;
                case 4: manager.updateStudent(); break;
                case 5: manager.deleteStudent(); break;
                case 6: manager.showTopper();    break;
                case 7: manager.rankList();      break;
                case 8: manager.statistics();    break;
                case 0: cout << "\nThank you! Exiting...\n"; break;
                default: cout << "\nInvalid choice!\n";
            }
        } catch (ResultException &e) {
            cout << "\nError: " << e.what() << "\n";
        }
    } while (choice != 0);
    return 0;
}
